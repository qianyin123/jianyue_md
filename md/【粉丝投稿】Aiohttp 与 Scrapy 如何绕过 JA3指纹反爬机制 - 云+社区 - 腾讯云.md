> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [cloud.tencent.com](https://cloud.tencent.com/developer/article/1875626) ![](https://ask.qcloudimg.com/http-save/yehe-3043884/18f7e367b656d669f2f78431bd0bf563.png?imageView2/2/w/1620)

摄影：产品经理

像百合一样的蟹斗

_本文来自公众号粉丝群中的@帝国皇家近卫军_

前几天观摩k大破解JA3的文章有感，可惜里面的JA3破解的库还是老掉牙的requests， 现在我看到了肯定是想办法改成基于asyncio的库啊。这样的话，在scrapy里面启用`AsyncioReactor`也能继续复用这个算法，不至于阻塞事件循环。

今天，我们把这个算法交给最快的http库，**aiohttp[1]** 来实现。破解JA3算法在于修改`SSLContext`的ciphers，这个控制了ssl指纹的生成。

k大文章中，修改ssl指纹的用到了`create_urllib3_context`这个函数。跟进这个函数的实现，便可以发现其中的奥秘。

```
def create_urllib3_context(
    ssl_version=None, cert_reqs=None, options=None, ciphers=None
):
    
    # PROTOCOL_TLS is deprecated in Python 3.10
    if not ssl_version or ssl_version == PROTOCOL_TLS:
        ssl_version = PROTOCOL_TLS_CLIENT

    context = SSLContext(ssl_version)

    context.set_ciphers(ciphers or DEFAULT_CIPHERS)

    # Setting the default here, as we may have no ssl module on import
    cert_reqs = ssl.CERT_REQUIRED if cert_reqs is None else cert_reqs

    if options is None:
        options = 0
        # SSLv2 is easily broken and is considered harmful and dangerous
        options |= OP_NO_SSLv2
        # SSLv3 has several problems and is now dangerous
        options |= OP_NO_SSLv3
        # Disable compression to prevent CRIME attacks for OpenSSL 1.0+
        # (issue #309)
        options |= OP_NO_COMPRESSION
        # TLSv1.2 only. Unless set explicitly, do not request tickets.
        # This may save some bandwidth on wire, and although the ticket is encrypted,
        # there is a risk associated with it being on wire,
        # if the server is not rotating its ticketing keys properly.
        options |= OP_NO_TICKET

    context.options |= options

    # Enable post-handshake authentication for TLS 1.3, see GH #1634. PHA is
    # necessary for conditional client cert authentication with TLS 1.3.
    # The attribute is None for OpenSSL <= 1.1.0 or does not exist in older
    # versions of Python.  We only enable on Python 3.7.4+ or if certificate
    # verification is enabled to work around Python issue #37428
    # See: https://bugs.python.org/issue37428
    if (cert_reqs == ssl.CERT_REQUIRED or sys.version_info >= (3, 7, 4)) and getattr(
        context, "post_handshake_auth", None
    ) is not None:
        context.post_handshake_auth = True

    def disable_check_hostname():
        if (
            getattr(context, "check_hostname", None) is not None
        ):  # Platform-specific: Python 3.2
            # We do our own verification, including fingerprints and alternative
            # hostnames. So disable it here
            context.check_hostname = False

    # The order of the below lines setting verify_mode and check_hostname
    # matter due to safe-guards SSLContext has to prevent an SSLContext with
    # check_hostname=True, verify_mode=NONE/OPTIONAL. This is made even more
    # complex because we don't know whether PROTOCOL_TLS_CLIENT will be used
    # or not so we don't know the initial state of the freshly created SSLContext.
    if cert_reqs == ssl.CERT_REQUIRED:
        context.verify_mode = cert_reqs
        disable_check_hostname()
    else:
        disable_check_hostname()
        context.verify_mode = cert_reqs

    # Enable logging of TLS session keys via defacto standard environment variable
    # 'SSLKEYLOGFILE', if the feature is available (Python 3.8+). Skip empty values.
    if hasattr(context, "keylog_filename"):
        sslkeylogfile = os.environ.get("SSLKEYLOGFILE")
        if sslkeylogfile:
            context.keylog_filename = sslkeylogfile

    return context

```

观察到`context.set_ciphers`那行，思考：aiohttp的`ClientSession` 中发起请求的方法都有个ssl参数，从typing可知他们接受的参数正是`SSLContext`

emmm，有戏

那把ssl参数换成自己的就ok了吗？好像没这么简单。还得书写轮换ssl指纹的逻辑，如下

```
import random
import ssl

# ssl._create_default_https_context = ssl._create_unverified_context


ORIGIN_CIPHERS = ('ECDH+AESGCM:DH+AESGCM:ECDH+AES256:DH+AES256:ECDH+AES128:DH+AES:ECDH+HIGH:'
                  'DH+HIGH:ECDH+3DES:DH+3DES:RSA+AESGCM:RSA+AES:RSA+HIGH:RSA+3DES')


class SSLFactory:
    def __init__(self):
        self.ciphers = ORIGIN_CIPHERS.split(":")

    def __call__(self) -> ssl.SSLContext:
        random.shuffle(self.ciphers)
        ciphers = ":".join(self.ciphers)
        ciphers = ciphers + ":!aNULL:!eNULL:!MD5"

        context = ssl.create_default_context()
        context.set_ciphers(ciphers)
        return context


sslgen = SSLFactory()

```

这样每次调用`sslgen()`都会得到一个独一无二的`SSLContext`

加上逻辑实现部分后

```
async def main():
    async with aiohttp.ClientSession() as session:
        for _ in range(5):
            async with session.get("https://ja3er.com/json", headers={}, ssl=sslgen()) as resp:
                data = await resp.json()
                print(data)

asyncio.get_event_loop().run_until_complete(main()) # 专为win准备的写法

```

![](https://ask.qcloudimg.com/http-save/yehe-3043884/fc2f6ecd598149076d7a2f8c878a9246.png)

哈哈现在ok了

现在就是把这个逻辑给scrapy复用了

众所周知的是scrapy的下载逻辑就在downloadhandler里面，修改这个即可。scrapy没有对downloadhandler书写文档，不过没关系，代码就是最好的文档。

观摩代码归纳后，总结出DownloadHandler必须实现的方法

**接口文档在这[2]**

继承`HTTPDownloadHandler`会省去很多工作，比如不需要修改的走默认逻辑即可。代码就放在**github[3]** 好了，毕竟有点长

来试一下新的下载器

```
import scrapy

class Ja3Spider(scrapy.Spider):
    name = 'ja3'
    allowed_domains = ['ja3er.com']
    start_urls = ['https://ja3er.com/json']

    def start_requests(self):
        for _ in range(5):
            yield scrapy.Request(self.start_urls[0], callback=self.parse, dont_filter=True, meta={"ja3": True}) # 呐，只有这样才会调用特殊的下载逻辑

    def parse(self, response: scrapy.http.response.text.TextResponse):
        print(response.json())


```

记得修改设置

```
import sys
if sys.platform == 'win32':
    asyncio.set_event_loop_policy(asyncio.WindowsSelectorEventLoopPolicy())
ASYNCIO_EVENT_LOOP = "asyncio.SelectorEventLoop"
TWISTED_REACTOR = 'twisted.internet.asyncioreactor.AsyncioSelectorReactor'

DOWNLOAD_HANDLERS = {"https": "scrapy_unitest.handlers.Ja3DownloadHandler"}

```

爬一下再试试

![](https://ask.qcloudimg.com/http-save/yehe-3043884/fc2f6ecd598149076d7a2f8c878a9246.png)

哈哈，现在再也不怕了。

另外，值得注意的是scrapy的issue里面真的有宝藏，多注意观察会有很多收获。

**参考文献**

[1]

aiohttp: _https://docs.aiohttp.org/en/stable/_

[2]

接口文档在这: _https://github.com/scrapy/scrapy/issues/4944_

[3]

github: _https://github.com/synodriver/awsome_scrapy_utils_

END