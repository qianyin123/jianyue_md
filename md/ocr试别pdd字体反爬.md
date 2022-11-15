> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/FCfpo9C-In_sxk8Rx7gYAg)

> ocr-字体反爬

*   引用了大佬的文章 `https://mp.weixin.qq.com/s/LkEGg7Jp-YSIGZ7PXfh7bw`
    

```
# -*- coding: UTF-8 -*-  
# @author: ylw  
# @file: font_decrypt2.py  
# @time: 2022/11/15  
# @desc: 利用ocr 试别字体  
import os  
  
import ddddocr  
import requests  
  
from fontTools.ttLib import TTFont  
from fontTools.pens.basePen import BasePen  
from reportlab.graphics.shapes import Path  
from reportlab.lib import colors  
from reportlab.graphics import renderPM  
from reportlab.graphics.shapes import Group, Drawing  
  
  
class ReportLabPen(BasePen):  
    """  
    画出字体的类  
    """  
    def __init__(self, glyphset, path=None):  
        BasePen.__init__(self, glyphset)  
        if path is None:  
            path = Path()  
        self.path = path  
  
    def _moveTo(self, p):  
        (x, y) = p  
        self.path.moveTo(x, y)  
  
    def _lineTo(self, p):  
        (x, y) = p  
        self.path.lineTo(x, y)  
  
    def _curveToOne(self, p1, p2, p3):  
        (x1, y1) = p1  
        (x2, y2) = p2  
        (x3, y3) = p3  
        self.path.curveTo(x1, y1, x2, y2, x3, y3)  
  
    def _closePath(self):  
        self.path.closePath()  
  
  
class TtfToImage:  
    """  
    将ttf文件中的文字转成image图片的类  
    """  
  
    def __init__(self, ttf_file, fmt='png'):  
        """  
        初始化对象  
        :param ttf_file: ttf文件的绝对路径  
        :param fmt: 输出的图片格式，默认png  
        """  
        self.__ttf_file = ttf_file  
        path, file_name = os.path.split(self.__ttf_file)  
  
        # 将ttf文件的文件名作为图片输出的文件夹名称，并放置在与ttf文件相同的目录下  
        out_path = os.path.join(path, file_name.split('.')[0])  
        self.__check_out_path(out_path)  
        self.out_path = out_path  
  
        self.fmt = fmt  
  
    @staticmethod  
    def __check_out_path(path):  
        if not os.path.isdir(path):  
            os.mkdir(path)  
  
    def draw_to_image(self):  
        """  
        将字体画图输出  
        返回字体文件储存img得 absPath  
        """  
        font = TTFont(self.__ttf_file)  
        gs = font.getGlyphSet()  
        glyphnames = font.getGlyphNames()  
        n = 0  
        for i in glyphnames:  
            if i[0] == '.':  
                continue  
  
            g = gs[i]  
            pen = ReportLabPen(gs, Path(fillColor=colors.black, strokeWidth=5))  
            g.draw(pen)  
            width, height = g.width, g.width * 1.5  # 在画布上 字体的宽高  
            g = Group(pen.path)  
            g.translate(dx=0, dy=0)  # 在画布上字体的x, y轴的偏移  
  
            d = Drawing(width, height)  
            d.add(g)  
            image_file = os.path.join(self.out_path, f'{i.replace("uni", "") + "." + self.fmt}')  
            renderPM.drawToFile(d, image_file, self.fmt)  
            n += 1  
            print(f'第{n}个字体制作完毕，图片为{image_file}！')  
  
        return self.out_path  
  
  
class Font_decrypt:  
    def __init__(self, crawl_name, foneFile_savePath: str):  
        """  
        :param crawl_name: 引用此文件得爬虫名字  
        :param foneFile_savePath: font文件保存到什么路径下  
        """  
        self.__ocr = ddddocr.DdddOcr()  
  
        self.__crawl_name = crawl_name  
        self.__foneFile_savePath = foneFile_savePath  
        self.__fontFile_name = f"font_{crawl_name}.ttf"  
        self.__font_file_path = f"{self.__foneFile_savePath}/{self.__fontFile_name}"  
        self.__this_session = requests.session()  
        self.__headers = {  
            "accept-encoding": "gzip, deflate, br",  
            "accept-language": "zh-CN,zh;q=0.9,en;q=0.8,en-GB;q=0.7,en-US;q=0.6",  
            "cache-control": "no-cache",  
            "content-length": "28",  
            "content-type": "application/json; charset=utf-8",  
            "origin": "https://mms.pinduoduo.com",  
            "pragma": "no-cache",  
            "referer": "https://mms.pinduoduo.com/",  
            "sec-ch-ua-mobile": "?0",  
            "sec-fetch-dest": "empty",  
            "sec-fetch-mode": "cors",  
            "sec-fetch-site": "cross-site",  
            "user-agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/100.0.4896.127 Safari/537.36 Edg/100.0.1185.44"  
        }  
        self.__num_code_map, self.web_spider_rule = self.__get_num_code_map()  
  
    def __get_num_code_map(self):  
        num_code_map = {}  
        web_spider_rule = self.__crawl_font_file()  
  
        ttf_to_img = TtfToImage(self.__font_file_path)  
        imgFiles_path = ttf_to_img.out_path  
  
        for file_name in os.listdir(imgFiles_path):  
            file_path = os.path.join(imgFiles_path, file_name)  
            os.remove(file_path)  
  
        ttf_to_img.draw_to_image()  
  
        for file_name in os.listdir(imgFiles_path):  
            if not file_name.endswith('png'):  
                continue  
  
            file_name_t = file_name.split(r'.', 1)[0]  
  
            if file_name_t == 'period':  
                num_code_map[file_name_t] = '.'  
                continue  
  
            file_path = os.path.join(imgFiles_path, file_name)  
            with open(file_path, 'rb') as f:  
                img_bytes = f.read()  
  
            text = self.__ocr.classification(img_bytes)  
            num_code_map[f"&#x{file_name_t.lower()}"] = text  
  
        return num_code_map, web_spider_rule  
  
    def __crawl_font_file(self):  
        url = 'https://api.yangkeduo.com/api/phantom/web/en/ft'  
        json_ = {"scene": "cp_b_data_center"}  
        response = self.__this_session.request('post', url, headers=self.__headers, json=json_).json()  
        web_spider_rule = response['web_spider_rule']  
        ttf_url = response['ttf_url']  
  
        response = requests.get(ttf_url, headers={  
            "accept-encoding": "gzip, deflate, br",  
            "accept-language": "zh-CN,zh;q=0.9,en;q=0.8,en-GB;q=0.7,en-US;q=0.6",  
            "cache-control": "no-cache",  
            "origin": "https://mms.pinduoduo.com",  
            "pragma": "no-cache",  
            "referer": "https://mms.pinduoduo.com/",  
            "sec-ch-ua-mobile": "?0",  
            "sec-ch-ua-platform": "\"Windows\"",  
            "sec-fetch-dest": "font",  
            "sec-fetch-mode": "cors",  
            "sec-fetch-site": "cross-site",  
            "user-agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/100.0.4896.127 Safari/537.36 Edg/100.0.1185.44"  
        })  
        with open(self.__font_file_path, 'wb') as f:  
            f.write(response.content)  
  
        return web_spider_rule  
  
    def font_decrypt(self, data_s):  
        """  
        data 列表嵌套字典  
        :param data_s:  
        :return:  
        """  
        print(self.__num_code_map)  
  
        new_data = []  
          
        for data in data_s:  
            temp_data = {}  
            for key, value in data.items():  
                value = repr(value).replace('\\u', '&#x').replace("'", "").replace('"', "")  
  
                for k, v in self.__num_code_map.items():  
                    value = value.replace(k, v)  
                temp_data[key] = value  
  
            new_data.append(temp_data)  
  
        print(new_data)  
        return new_data  
  
  
if __name__ == '__main__':  
    # ttf = TtfToImage(r'C:\Program Files (x86)\code_ylw\pdd\font_file\font_pdd_dataCentre_visitors_number.ttf')  
    # ttf.draw_to_image()  
  
    decrypt = Font_decrypt('pdd_dataCentre_visitors_number', r'C:\Program Files (x86)\code_ylw\pdd\font_file')  
    decrypt.font_decrypt()  
  

```