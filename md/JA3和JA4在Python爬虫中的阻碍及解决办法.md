> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/fT1KRXZrExbcOkFRwbCyGA)

JA3和JA4在Python爬虫中的阻碍及解决办法
=========================

一、核心概念详解
--------

### 1.1 TLS/SSL握手过程详解

TLS（Transport Layer Security）和SSL（Secure Sockets Layer）是密码学协议，为网络通信提供安全性。握手过程是建立安全连接的关键步骤：

1.  **ClientHello**：客户端向服务器发送支持的TLS版本、加密套件列表、随机数、会话ID和扩展信息
    
2.  **ServerHello**：服务器响应选择的TLS版本、加密套件、会话ID和自己的随机数
    
3.  **证书交换**：服务器发送数字证书（可选）
    
4.  **密钥交换**：双方协商出用于加密通信的会话密钥
    
5.  **完成握手**：双方确认握手完成，开始使用协商的参数进行加密通信
    

### 1.2 JA3技术原理

JA3是由Salesforce在2017年开源的TLS客户端指纹识别技术，通过分析ClientHello消息中的特定字段生成MD5哈希值：

JA3字符串格式为：`TLSVersion,Ciphers,Extensions,EllipticCurves,EllipticCurvePointFormats`

例如：

```
769,47–53–5–10–49161–49162–49171–49172–50–56–19–4,0–10–11,23–24–25,0  

```

该字符串经过MD5哈希处理后生成32位的指纹：

```
e7d705a3286e19ea42f587b344ee6865  

```

### 1.3 JA4技术演进

JA4是由John Althouse在2023年提出的升级版指纹技术，主要针对TLS 1.3，改进了JA3的一些局限性：

JA4字符串格式：`[Version]_[SNI]_[ALPN]_[CipherOrder]`

其中：

*   Version：TLS版本（如12=TLS 1.2，13=TLS 1.3）
    
*   SNI：服务器名称指示（Server Name Indication）的长度
    
*   ALPN：应用层协议协商（Application-Layer Protocol Negotiation）扩展的值
    
*   CipherOrder：密码套件顺序的哈希值（取前4位）
    

示例：`13_24_h2_1a2b`（表示TLS 1.3，SNI长度为24，ALPN为h2，CipherOrder哈希前4位为1a2b）

二、识别机制的技术实现
-----------

### 2.1 指纹提取技术

服务器通过以下步骤提取客户端指纹：

1.  **数据包捕获**：使用libpcap等库捕获TLS握手过程的网络数据包
    
2.  **TLS解析**：解析ClientHello消息，提取相关字段
    
3.  **特征提取**：按照JA3/JA4定义的格式组织数据
    
4.  **哈希计算**：计算MD5哈希值作为最终指纹
    

```
# JA3指纹生成示例代码（简化版）  
def generate_ja3_fingerprint(client_hello):  
    # 提取TLS版本  
    tls_version = client_hello.version  
      
    # 提取密码套件  
    cipher_suites = "-".join([str(cs) for cs in client_hello.cipher_suites])  
      
    # 提取扩展  
    extensions = "-".join([str(ext.type) for ext in client_hello.extensions])  
      
    # 提取椭圆曲线  
    elliptic_curves = "-".join([str(ec) for ec in client_hello.get_extension(10).elliptic_curves])  
      
    # 提取椭圆曲线点格式  
    ec_point_formats = "-".join([str(pf) for pf in client_hello.get_extension(11).ec_point_formats])  
      
    # 组合JA3字符串  
    ja3_string = f"{tls_version},{cipher_suites},{extensions},{elliptic_curves},{ec_point_formats}"  
      
    # 计算MD5哈希  
    ja3_hash = hashlib.md5(ja3_string.encode()).hexdigest()  
      
    return ja3_hash  

```

### 2.2 服务器端检测实现

网站的防爬虫系统通常会：

1.  维护已知爬虫指纹数据库
    
2.  实时捕获并分析访问者的JA3/JA4指纹
    
3.  与数据库比对，判断是否为爬虫
    
4.  设置动态规则（如频率限制、CAPTCHA挑战或直接封禁）
    

```
# 服务器端检测代码示例（简化版）  
def detect_crawler(client_fingerprint):  
    # 已知爬虫指纹库  
    known_crawler_fingerprints = [  
        "e7d705a3286e19ea42f587b344ee6865",  # Python requests默认指纹  
        "a0e9f5d64349fb13191bc781f81f42e1",  # urllib指纹  
        # 更多已知爬虫指纹...  
    ]  
      
    if client_fingerprint in known_crawler_fingerprints:  
        return True  # 检测到爬虫  
      
    # 可以进一步分析访问模式、频率等  
    return False  

```

三、Python爬虫面临的挑战详解
-----------------

### 3.1 指纹识别机制

Python常用HTTP库（如requests、urllib）使用的是系统提供的SSL库，具有明显不同于常用浏览器的TLS握手特征：

1.  **默认密码套件差异**：Python默认提供的密码套件顺序与Chrome、Firefox等浏览器不同
    
2.  **TLS扩展支持差异**：爬虫工具通常缺少浏览器支持的某些TLS扩展
    
3.  **版本特征**：不同Python和OpenSSL版本组合生成的指纹各不相同
    

### 3.2 复杂的检测算法

现代网站的反爬机制不仅依赖单一指纹，还会结合多种检测手段：

1.  **行为分析**：请求频率、路径模式、时间分布
    
2.  **浏览器特征检测**：JavaScript执行能力、WebGL指纹等
    
3.  **指纹相关性分析**：JA3/JA4指纹与User-Agent、HTTP/2设置等因素的一致性
    

### 3.3 实时适应机制

防爬系统通常采用机器学习算法，能够：

1.  **自适应学习**：不断更新爬虫特征库
    
2.  **异常检测**：识别统计上异常的访问模式
    
3.  **行为聚类**：将相似行为的客户端归为同一类型
    

四、深入解决方案
--------

### 4.1 TLS指纹伪造技术

#### 4.1.1 精确控制SSL/TLS参数

使用`ssl`模块自定义SSL上下文，精确控制握手参数：

```
import ssl  
import requests  
from urllib3.poolmanager import PoolManager  
from urllib3.util import ssl_  
  
# 定义目标浏览器的密码套件顺序  
CIPHERS = (  
    'TLS_AES_128_GCM_SHA256:'  
    'TLS_AES_256_GCM_SHA384:'  
    'TLS_CHACHA20_POLY1305_SHA256:'  
    'ECDHE-ECDSA-AES128-GCM-SHA256:'  
    'ECDHE-RSA-AES128-GCM-SHA256:'  
    'ECDHE-ECDSA-AES256-GCM-SHA384:'  
    'ECDHE-RSA-AES256-GCM-SHA384:'  
    'ECDHE-ECDSA-CHACHA20-POLY1305:'  
    'ECDHE-RSA-CHACHA20-POLY1305:'  
    'ECDHE-RSA-AES128-SHA:'  
    'ECDHE-RSA-AES256-SHA:'  
    'AES128-GCM-SHA256:'  
    'AES256-GCM-SHA384:'  
    'AES128-SHA:'  
    'AES256-SHA'  
)  
  
class CustomHTTPAdapter(requests.adapters.HTTPAdapter):  
    def init_poolmanager(self, *args, **kwargs):  
        context = ssl_.create_urllib3_context(ciphers=CIPHERS)  
        # 设置其他TLS参数  
        context.options |= ssl.OP_NO_TLSv1 | ssl.OP_NO_TLSv1_1  # 禁用TLS 1.0和1.1  
        context.set_alpn_protocols(['h2', 'http/1.1'])  # 设置ALPN协议  
        kwargs['ssl_context'] = context  
        return super().init_poolmanager(*args, **kwargs)  
  
# 使用自定义适配器  
session = requests.Session()  
adapter = CustomHTTPAdapter()  
session.mount('https://', adapter)  
  
response = session.get('https://example.com', headers={  
    'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/95.0.4638.69 Safari/537.36'  
})  

```

#### 4.1.2 使用专用库

`tlsfingerprint`和`curl_cffi`等专用库可以精确模拟特定浏览器的TLS握手特征：

```
from curl_cffi import requests  
  
# 使用curl_cffi模拟Chrome浏览器  
response = requests.get("https://example.com",   
                       impersonate="chrome110",  # 模拟Chrome 110  
                       headers={  
                           "User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/110.0.0.0 Safari/537.36"  
                       })  

```

### 4.2 高级浏览器自动化

#### 4.2.1 Selenium与浏览器插件结合

```
from selenium import webdriver  
from selenium.webdriver.chrome.options import Options  
from selenium.webdriver.chrome.service import Service  
from webdriver_manager.chrome import ChromeDriverManager  
  
# 设置Chrome选项  
options = Options()  
options.add_argument("--disable-blink-features=AutomationControlled")  # 隐藏自动化特征  
options.add_experimental_option("excludeSwitches", ["enable-automation"])  
options.add_experimental_option("useAutomationExtension", False)  
  
# 添加扩展插件（如代理管理器）  
# options.add_extension('path/to/extension.crx')  
  
# 启动浏览器  
service = Service(ChromeDriverManager().install())  
driver = webdriver.Chrome(service=service, options=options)  
  
# 修改WebDriver特征  
driver.execute_cdp_cmd("Page.addScriptToEvaluateOnNewDocument", {  
    "source": """  
    Object.defineProperty(navigator, 'webdriver', {  
        get: () => undefined  
    });  
    """  
})  
  
# 访问网站  
driver.get("https://example.com")  

```

#### 4.2.2 无头浏览器优化

```
from playwright.sync_api import sync_playwright  
  
with sync_playwright() as p:  
    # 使用Playwright启动Chrome浏览器  
    browser = p.chromium.launch(  
        headless=True,  # 无头模式  
        args=[  
            "--disable-blink-features=AutomationControlled",  
            "--disable-extensions",  
            "--disable-component-extensions-with-background-pages",  
            "--disable-default-apps"  
        ]  
    )  
      
    # 创建上下文和页面  
    context = browser.new_context(  
        viewport={"width": 1920, "height": 1080},  
        user_agent="Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/96.0.4664.110 Safari/537.36"  
    )  
      
    # 修改指纹特征  
    context.add_init_script("""  
        Object.defineProperty(navigator, 'webdriver', {get: () => undefined});  
        Object.defineProperty(navigator, 'plugins', {get: () => [1, 2, 3, 4, 5]});  
    """)  
      
    page = context.new_page()  
    page.goto("https://example.com")  
    content = page.content()  
      
    browser.close()  

```

### 4.3 高级代理策略

#### 4.3.1 轮换代理与会话管理

```
import requests  
from random import choice  
import time  
  
# 代理池配置  
proxy_pool = [  
    {"http": "http://proxy1.example.com:8080", "https": "https://proxy1.example.com:8080"},  
    {"http": "http://proxy2.example.com:8080", "https": "https://proxy2.example.com:8080"},  
    # 更多代理...  
]  
  
# 浏览器UA池  
user_agents = [  
    "Mozilla/5.0 (Windows NT 10.0; Win64; x64) Chrome/96.0.4664.110 Safari/537.36",  
    "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) Safari/605.1.15",  
    # 更多UA...  
]  
  
# 会话管理  
sessions = {}  
  
def get_session(proxy):  
    """为每个代理创建和管理单独的会话"""  
    proxy_key = str(proxy)  
    if proxy_key not in sessions:  
        session = requests.Session()  
        adapter = requests.adapters.HTTPAdapter(max_retries=3)  
        session.mount('http://', adapter)  
        session.mount('https://', adapter)  
        sessions[proxy_key] = session  
    return sessions[proxy_key]  
  
# 爬取函数  
def scrape_with_rotation(urls):  
    results = []  
      
    for url in urls:  
        # 选择代理和UA  
        proxy = choice(proxy_pool)  
        ua = choice(user_agents)  
          
        # 获取会话  
        session = get_session(proxy)  
          
        try:  
            # 发送请求  
            response = session.get(  
                url,  
                proxies=proxy,  
                headers={"User-Agent": ua},  
                timeout=10  
            )  
            results.append(response.text)  
              
            # 随机延迟  
            time.sleep(2 + random.random() * 3)  
              
        except Exception as e:  
            print(f"Error scraping {url}: {e}")  
      
    return results  

```

#### 4.3.2 代理服务器TLS指纹修改

通过设置专用代理服务器，可以在代理层面修改TLS指纹：

```
# 使用mitmproxy作为中间代理修改TLS指纹的示例配置  
# mitmproxy配置文件：~/config.py  
  
from mitmproxy import ctx  
import ssl  
  
# 自定义TLS配置  
CIPHERS = "ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:..."  
  
def modify_client_hello(client_hello):  
    """修改ClientHello消息以模拟特定浏览器"""  
    # 修改扩展顺序和内容  
    # 修改密码套件顺序  
    # ...  
  
# 设置TLS上下文  
def configure(config):  
    ctx.options.update(  
        ssl_version_client="TLS1_2",  
        ciphers_client=CIPHERS,  
        # 其他TLS参数  
    )  

```

### 4.4 高级反检测技术

#### 4.4.1 浏览器指纹规避

```
from selenium import webdriver  
from selenium.webdriver.chrome.options import Options  
import random  
import time  
  
# 配置Chrome选项  
options = Options()  
  
# 随机屏幕分辨率  
resolutions = [(1366, 768), (1920, 1080), (1536, 864), (1440, 900)]  
width, height = random.choice(resolutions)  
options.add_argument(f"--window-size={width},{height}")  
  
# 随机设置语言  
languages = ["en-US", "en-GB", "fr-FR", "de-DE", "es-ES"]  
options.add_argument(f"--lang={random.choice(languages)}")  
  
# 禁用WebRTC (防止IP泄露)  
options.add_argument("--disable-webrtc")  
  
# 启动浏览器  
driver = webdriver.Chrome(options=options)  
  
# 注入JavaScript以修改Canvas和WebGL指纹  
driver.execute_cdp_cmd("Page.addScriptToEvaluateOnNewDocument", {  
    "source": """  
    // 修改Canvas指纹  
    const originalToDataURL = HTMLCanvasElement.prototype.toDataURL;  
    HTMLCanvasElement.prototype.toDataURL = function(type) {  
        const dataURL = originalToDataURL.apply(this, arguments);  
        if (this.width > 16 && this.height > 16) {  
            // 微调数据以改变指纹但保持图像基本不变  
            const noise = Math.floor(Math.random() * 10) / 100;  
            return dataURL.replace(/([0-9]+(\.[0-9]+)?)/g,   
                   (match) => (parseFloat(match) + noise).toString());  
        }  
        return dataURL;  
    };  
      
    // 修改WebGL指纹  
    const getParameterProto = WebGLRenderingContext.prototype.getParameter;  
    WebGLRenderingContext.prototype.getParameter = function(parameter) {  
        // 修改RENDERER和VENDOR信息  
        if (parameter === 37445) { // UNMASKED_RENDERER_WEBGL  
            return 'Intel Iris OpenGL Engine';  
        }  
        if (parameter === 37446) { // UNMASKED_VENDOR_WEBGL  
            return 'Intel Inc.';  
        }  
        return getParameterProto.apply(this, arguments);  
    };  
    """  
})  
  
# 访问网站并模拟人类行为  
driver.get("https://example.com")  
  
# 模拟人类行为  
def simulate_human_behavior():  
    # 随机滚动  
    scroll_amount = random.randint(100, 800)  
    driver.execute_script(f"window.scrollBy(0, {scroll_amount});")  
    time.sleep(random.uniform(0.5, 2.0))  
      
    # 随机鼠标移动（使用JavaScript模拟）  
    driver.execute_script("""  
    const event = new MouseEvent('mousemove', {  
        'view': window,  
        'bubbles': true,  
        'cancelable': true,  
        'clientX': arguments[0],  
        'clientY': arguments[1]  
    });  
    document.dispatchEvent(event);  
    """, random.randint(10, width-10), random.randint(10, height-10))  
      
    # 随机等待  
    time.sleep(random.uniform(1.0, 5.0))  
  
# 执行模拟行为  
for _ in range(5):  
    simulate_human_behavior()  
  
# 获取页面内容  
content = driver.page_source  
  
driver.quit()  

```

五、行业趋势与未来发展
-----------

### 5.1 TLS指纹技术演进

随着TLS 1.3广泛部署，指纹技术正在不断发展：

1.  **JA4S**：服务器TLS指纹，与JA4配对使用
    
2.  **JA4H**：HTTP头部指纹，补充TLS层面的识别
    
3.  **JA4L**：链路层指纹，分析TCP选项等低层特征
    
4.  **JA4X**：扩展指纹，结合多种协议特征的综合指纹
    

### 5.2 机器学习在反爬虫中的应用

网站防护正日益采用机器学习算法：

1.  **异常检测**：基于无监督学习识别非典型访问模式
    
2.  **行为分析**：深度学习模型分析鼠标移动、点击模式等
    
3.  **多因素决策**：融合多种特征（网络、行为、环境）做出封禁决策
    

### 5.3 合规与道德考量

随着隐私法规（如GDPR、CCPA）的实施，爬虫开发者需注意：

1.  **合法获取数据**：遵守网站robots.txt和使用条款
    
2.  **尊重访问限制**：遵循网站设定的访问频率限制
    
3.  **保护隐私数据**：不收集或处理个人敏感信息
    
4.  **透明度**：在适当情况下标识爬虫身份
    

六、综合实战解决方案
----------

为了最大程度避开JA3/JA4指纹检测，可采用多层次防护策略：

### 6.1 全方位TLS指纹管理方案

```
import undetected_chromedriver as uc  
from playwright.sync_api import sync_playwright  
import requests_random_user_agent  # 随机UA  
import tls_client  # 专用TLS客户端库  
import random  
import time  
  
# 方案1: 使用undetected_chromedriver  
def scrape_with_undetected_chrome(url):  
    options = uc.ChromeOptions()  
    options.add_argument("--disable-gpu")  
      
    driver = uc.Chrome(options=options)  
    driver.get(url)  
      
    # 模拟人类行为  
    time.sleep(random.uniform(1, 3))  
    driver.execute_script(f"window.scrollTo(0, {random.randint(100, 1000)});")  
    time.sleep(random.uniform(1, 3))  
      
    content = driver.page_source  
    driver.quit()  
    return content  
  
# 方案2: 使用Playwright  
def scrape_with_playwright(url):  
    with sync_playwright() as p:  
        browser = p.chromium.launch(  
            headless=False,  
            args=["--disable-blink-features=AutomationControlled"]  
        )  
        context = browser.new_context(  
            viewport={"width": 1920, "height": 1080},  
            locale="en-US"  
        )  
        page = context.new_page()  
        page.goto(url)  
          
        # 模拟人类行为  
        page.mouse.move(random.randint(100, 800), random.randint(100, 600))  
        page.mouse.wheel(delta_y=random.randint(100, 300))  
        page.wait_for_timeout(random.uniform(1000, 3000))  
          
        content = page.content()  
        browser.close()  
        return content  
  
# 方案3: 使用专用TLS客户端库  
def scrape_with_tls_client(url):  
    session = tls_client.Session(  
        client_identifier="chrome110",  
        random_tls_extension_order=True  
    )  
      
    response = session.get(url)  
    return response.text  
  
# 综合方案: 随机选择不同方法并使用代理  
def adaptive_scraping(url, proxies=None):  
    # 随机选择爬取方法  
    method = random.choice([  
        scrape_with_undetected_chrome,  
        scrape_with_playwright,  
        scrape_with_tls_client  
    ])  
      
    # 配置代理  
    if proxies and method == scrape_with_tls_client:  
        proxy = random.choice(proxies)  
        session.proxies = {  
            "http": f"http://{proxy}",  
            "https": f"https://{proxy}"  
        }  
      
    # 执行爬取  
    try:  
        return method(url)  
    except Exception as e:  
        print(f"Error with {method.__name__}: {e}")  
        # 失败时尝试备用方法  
        backup_methods = [m for m in [  
            scrape_with_undetected_chrome,  
            scrape_with_playwright,  
            scrape_with_tls_client  
        ] if m != method]  
          
        if backup_methods:  
            return random.choice(backup_methods)(url)  
        raise  

```

七、总结与最佳实践
---------

### 7.1 技术选择指南

根据不同场景选择合适的反检测技术：

1.  **简单爬取任务**：使用`requests`+`curl_cffi`等轻量库+随机代理组合
    
2.  **中等复杂度网站**：使用Selenium或Playwright+未检测的ChromeDriver+代理轮换
    
3.  **高难度网站**：多技术组合+行为模拟+专用代理服务器+指纹管理
    

### 7.2 风险评估与缓解

在开发爬虫时：

1.  **分析目标网站防护机制**：识别其使用的检测技术和策略
    
2.  **渐进式请求策略**：从低频率开始，逐步调整至最优请求频率
    
3.  **监控封禁信号**：如验证码出现、HTTP 429/403状态码等
    
4.  **降级策略**：当检测到封禁风险时，自动切换到更安全的爬取模式
    

### 7.3 长期维护策略

爬虫项目的长期成功依赖于：

1.  **持续监控**：定期检查爬虫效果，分析失败原因
    
2.  **技术更新**：跟踪并适应反爬技术的最新发展
    
3.  **分布式架构**：将爬取任务分散到多个源以减少单点风险
    
4.  **数据质量优先**：在速度和数据质量之间权衡，保证数据完整性
    

TLS指纹技术是当代反爬虫的核心技术之一，但通过深入理解其原理和采用多层防护策略，开发者可以构建更健壮、更难被检测的爬虫系统。同时，合规与道德考量同样重要，确保爬虫行为遵循相关法规和网站政策。

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/N1Fms3bbiaBDqj6QnUvCNGYdPKcFaXsJVN34VKtyptJSicszC21gxKSoU1HqicG8QsXrtWxC3TGMFu8jFjqT8rG2A/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1)