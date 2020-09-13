# Python爬虫

## 1、xpath

### 58同城二手房-xpath

```python
import requests
from lxml import etree
if __name__=="__main__":
    url='https://jh.58.com/ershoufang/'
    headers = {
        'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/84.0.4147.89 Safari/537.36'
    }
    response = requests.get(url=url,headers=headers).text
    tree=etree.HTML(response)
    li_list=tree.xpath('//ul[@class="house-list-wrap"]/li')
    for li in li_list:
        title=li.xpath('./div[2]/h2/a/text()')[0]
        print(title)
```

### 城市质量-xpath

```python
import requests
from lxml import etree
headers={
        'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/84.0.4147.89 Safari/537.36'
    }
url='https://www.aqistudy.cn/historydata/'
page_text=requests.get(url=url,headers=headers).text
tree=etree.HTML(page_text)
hot_city=tree.xpath('//div[@class="bottom"]/ul/li/a/text()')
all_city=tree.xpath('//div[@class="bottom"]/ul/div[2]/li/a/text()')
print(hot_city)
```

### 图片存储-xpath

```python
import requests
from lxml import etree
import os
dirname='star1'
if not os.path.exists(dirname):
    os.mkdir(dirname)
url='http://pic.netbian.com/4kmingxing/index_%d.html'#爬取多页内容
for i in range(1,6):
    if i==1:
        new_url='http://pic.netbian.com/4kmingxing/'
    else:
        new_url=format(url%i)
    #url='http://pic.netbian.com/4kmingxing/'爬取一页的内容
    headers={
        'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/84.0.4147.89 Safari/537.36'
    }
    response=requests.get(url=new_url,headers=headers)
    response.encoding='gbk'
    page_text=response.text
    tree=etree.HTML(page_text)
    li_list=tree.xpath('//div[@class="slist"]/ul/li')
    for li in li_list:
        title=li.xpath('./a/img/@alt')[0]+'.jpg'
        img_src='http://pic.netbian.com'+li.xpath('./a/img/@src')[0]
        img_data=requests.get(url=img_src,headers=headers).content
        imgpath=dirname+'/'+title
        with open(imgpath,'wb') as fp:
            fp.write(img_data)
        print(title,'保存成功！')
```

## 2、协程

### await应用

```python
#示例一
'''import asyncio
async def func():
    print("请稍后...")
    response=await asyncio.sleep(2)
    print("欢迎",response)
asyncio.run(func())'''

#示例二
'''import asyncio
async def others():
    print("start")
    await asyncio.sleep(2)
    print("end")
    return "返回值"
async def func():
    print("执行协程函数内部代码")
    response=await others()
    print("IO请求结束，结果为：",response)
asyncio.run(func())'''


#示例三
import asyncio
async def others():
    print("start")
    await asyncio.sleep(2)
    print("end")
    return "返回值"
async def func():
    print("执行协程函数内部代码")
    response1=await others()
    print("IO请求结束，结果为：",response1)
    response2 = await others()
    print("IO请求结束，结果为：", response2)
asyncio.run(func())
```

### 协程基础

```python
import asyncio
import time
async def get_request(url):
    print("正在请求：",url)
    time.sleep(2)
    print("请求已完成!")
    return 'jackson'
def back(t):
    #result返回的就是特殊函数的返回值
    print('t.result返回的是：',t.result())
if __name__=="__main__":
    #这是一个协程对象
    c=get_request('www.baidu.com')
    #任务对象就是对协程的进一步封装
    task=asyncio.ensure_future(c)
    #绑定一个回调函数
    task.add_done_callback(back)
    #创建事件循环对象
    loop=asyncio.get_event_loop()
    #将任务对象注册到事件循环中且开启事件循环
    loop.run_until_complete(task)
```

### task对象

```python
'''import asyncio
async def func():
    print(1)
    await asyncio.sleep(2)
    print(2)
    return "返回值"
async def main():
    print("main开始")
    task1=asyncio.create_task(func())
    task2=asyncio.create_task(func())
    print("main结束")
    re1=await task1
    re2=await task2
    print(re1,re2)
asyncio.run(main())'''



import asyncio
async def func():
    print(1)
    await asyncio.sleep(2)
    print(2)
    return "返回值"
async def main():
    print("mian开始")
    task_list=[
        asyncio.create_task(func()),
        asyncio.create_task(func())
    ]
    print("main结束")
    result=await asyncio.wait(task_list)
    print(result)
asyncio.run(main())
```

### greenlet

```python
from greenlet import greenlet
def func1():
    print(1)
    res2.switch()
    print(2)
    res2.switch()
def func2():
    print(3)
    res1.switch()
    print(4)
res1=greenlet(func1)
res2=greenlet(func2)
res1.switch()
```

### yield

```python
def func1():
    yield 1
    yield from func2()
    yield 2
def func2():
    yield 3
    yield 4
f1=func1()
for item in f1:
    print(item)
```

### 多任务协程

```python
import asyncio
import time
'''async def get_request(url):
    print("正在请求：",url)
    time.sleep(2)#time是不支持异步模块的代码
    print("请求已完成!")
    return 'jackson'
'''
async def get_request(url):
    print("正在请求：",url)
    await asyncio.sleep(2)#支持异步模块的代码
    print("请求已完成!")
    return 'jackson'
def back(t):
    #result返回的就是特殊函数的返回值
    print('t.result返回的是：',t.result())
urls=[
    'www.baidu1.0.com',
    'www.baidu2.0.com',
    'www.baidu3.0.com'
]
if __name__=="__main__":
    start=time.time()
    tasks=[]
    #创建协程对象
    for url in urls:
        c=get_request(url)
        #创建任务对象
        task=asyncio.ensure_future(c)
        task.add_done_callback(back)
        tasks.append(task)
    #创建事件循环对象
    loop=asyncio.get_event_loop()
    #loop.run_until_complete(tasks)
    #必须使用wait对tasks进行封装才能执行成功
    loop.run_until_complete(asyncio.wait(tasks))
    print("总耗时：", time.time() - start)
```

### 多任务的异步爬虫

```python
import asyncio
import time
import aiohttp
urls=[
        'http://127.0.0.1:8000/jackson',
        'http://127.0.0.1:8000/jing',
        'http://127.0.0.1:8000/jack',
    ]
'''async def get_request(url):
    #requests是一个不支持异步的模块
    page_text=requests.get(url=url).text
    return page_text
    '''
async def get_request(url):
    #实例化好一个请求对象
   async with aiohttp.ClientSession() as se:
        #调用get发起请求，返回一个响应对象
       async with await se.get(url=url) as response:
            #获取字符串形式的响应数据
            page_text=await response.text()
            return page_text
if __name__=="__main__":
    start = time.time()
    tasks = []
    # 创建协程对象
    for url in urls:
        c = get_request(url)
        # 创建任务对象
        task = asyncio.ensure_future(c)
        tasks.append(task)
    # 创建事件循环对象
    loop = asyncio.get_event_loop()
    # loop.run_until_complete(tasks)
    # 必须使用wait对tasks进行封装才能执行成功
    loop.run_until_complete(asyncio.wait(tasks))
    print("总耗时：", time.time() - start)
```

### 线程池-梨视频

```python
import requests
from lxml import etree
import re
from multiprocessing.dummy import Pool
headers = {
    'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/84.0.4147.89 Safari/537.36'
}
url='https://www.pearvideo.com/category_5'
response = requests.get(url=url,headers=headers).text
tree=etree.HTML(response)
li_list=tree.xpath('//ul[@id="listvideoListUl"]/li')
urls=[]#存储所有视频的链接和名字
for li in li_list:
    detail_url='https://www.pearvideo.com/'+li.xpath('./div/a/@href')[0]
    name=li.xpath('./div/a/div[2]/text()')[0]+'.MP4'
    #对详情页的url发起请求
    detail_response=requests.get(url=detail_url,headers=headers).text
    #从详情页中解析出视频的地址（url）
    ex='srcUrl="(.*?)",vdoUrl'
    video_url=re.findall(ex,detail_response)[0]
    dic={
        'name':name,
        'url':video_url
    }
    urls.append(dic)
#对视频链接发起请求获取视频的二进制数据，然后将视频数据进行返回
def get_video_data(dic):
    url=dic['url']
    print(dic['name'],'正在下载...')
    data=requests.get(url=url,headers=headers).content
    #持久化存储操作
    with open(dic['name'],'wb') as fp:
        fp.write(data)
        print(dic['name'],'下载完成')
#使用线程池对视频数据进行请求（较为耗时的阻塞操作）
pool=Pool(4)
pool.map(get_video_data,urls)
pool.close()
pool.join()
```

## 3、selenium

### selenium自动化操作-移动html

```python
from selenium import webdriver
import time
#导入动作链对应的类
from selenium.webdriver import ActionChains
bro=webdriver.Chrome(executable_path='E:/firefoxdownloads/chromedriver.exe')
bro.get('https://www.runoob.com/try/try.php?filename=jqueryui-api-droppable')
#如果定位的标签是存在于iframe标签中的则必须进行标签定位
bro.switch_to.frame('iframeResult')#切换浏览器定位的作用域
div=bro.find_element_by_id('draggable')
#动作链
action=ActionChains(bro)
#点击长按指定的标签
action.click_and_hold(div)
for i in range(5):
    #perform立即执行动作链操作
    #move_by_offset（x，y）
    action.move_by_offset(20,0).perform()
    time.sleep(0.3)
#释放动作链
action.release()
bro.quit()
```

### selenium自动化-淘宝搜索

```python
from selenium import webdriver
import time
#基于浏览器的驱动程序实例化一个浏览器对象
bro=webdriver.Chrome(executable_path='E:/firefoxdownloads/chromedriver.exe')
#对目的网站发起请求
bro.get('https://www.jd.com')
#标签定位
search_text=bro.find_element_by_xpath('//*[@id="key"]')
#标签交互
search_text.send_keys('iphone11')
#点击搜索按钮
bth=bro.find_element_by_xpath('//*[@id="search"]/div/div[2]/button')
bth.click()
time.sleep(2)
#在搜索结果页面进行滚轮向下滑动的操作（执行js操作：js注入）
bro.execute_script('window.scrollTo(0,document.body.scrollHeight)')
time.sleep(2)
bro.get('https://www.baidu.com')
time.sleep(2)
#回退
bro.back()
time.sleep(2)
#前进
bro.forward()
bro.quit()
```

### selenium实现QQ空间登录

```python
from selenium import webdriver
import time
from selenium.webdriver import ActionChains
bro=webdriver.Chrome(executable_path='E:/firefoxdownloads/chromedriver.exe')
bro.get('https://qzone.qq.com/')
bro.switch_to.frame('login_frame')
a_tag=bro.find_element_by_id('switcher_plogin')
a_tag.click()
username=bro.find_element_by_id('u')
password=bro.find_element_by_id('p')
time.sleep(2)
username.send_keys('')
time.sleep(2)
password.send_keys('')
time.sleep(2)
btn=bro.find_element_by_id('login_button')
btn.click()
time.sleep(2)
bro.quit()
```

## 4、scrapy

## 1、进入到文件

### 例如：cd scrapy操作

## 2、创建项目

### 例如：scrapy startproject qiubaipro

## 3、进入qiubaipro

### 例如：cd qiubaipro

## 4、创建爬虫源文件

### 例如：scrapy genspider qiubai www.baidu

### .com（qiubai是文件名）

## 5、执行工程

### 例如：scrapy crawl qiubai

## 6、段子爬取命令（基于终端的持久化存储）

### scrapy crawl qiubai -o qiubai.csv

![image-20200824094432380](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200824094432380.png)

```python
import scrapy


class QiubaiSpider(scrapy.Spider):
    name = 'qiubai'
    #allowed_domains = ['www.xxx.com']
    start_urls = ['https://www.qiushibaike.com/text/']

    def parse(self, response):
        #解析：作者的名称+段子内容
        div_list=response.xpath('//div[@class="col1 old-style-col1"]/div')
        all_data=[]#存储所有解析到的数据
        for div in div_list:
            #xpath返回的是列表，但列表元素一定是selector类型的对象
            #extract可以将selector对象中data参数存储的字符串提取出来
            author=div.xpath('./div[1]/a[2]/h2/text()')[0].extract()
            #列表调用了extract之后，则表示将列表中每一个selector对象中data对应的字符串提取出来
            content=div.xpath('./a[1]/div/span//text()').extract()
            content=''.join(content)#转为字符串类型
            dic={
                'author':author,
                'content':content
            }
            all_data.append(dic)
        return all_data
```

## 7、setting设置

```python
USER_AGENT = 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/84.0.4147.89 Safari/537.36'
ROBOTSTXT_OBEY = False
LOG_LEVEL='ERROR'
```

## 8、基于管道的持久化存储

### 一、qiubai.py

```python
import scrapy
from qiubaipro.items import QiubaiproItem

class QiubaiSpider(scrapy.Spider):
    name = 'qiubai'
    #allowed_domains = ['www.xxx.com']
    start_urls = ['https://www.qiushibaike.com/text/']
    def parse(self, response):
        #解析：作者的名称+段子内容
        div_list=response.xpath('//div[@class="col1 old-style-col1"]/div')
        all_data=[]#存储所有解析到的数据
        for div in div_list:
            #xpath返回的是列表，但列表元素一定是selector类型的对象
            #extract可以将selector对象中data参数存储的字符串提取出来
            author=div.xpath('./div[1]/a[2]/h2/text()')[0].extract()
            #列表调用了extract之后，则表示将列表中每一个selector对象中data对应的字符串提取出来
            content=div.xpath('./a[1]/div/span//text()').extract()
            content=''.join(content)#转为字符串类型
            item=QiubaiproItem()
            item['author']=author
            item['content']=content
            yield item#将item提交给管道
```

### 二、item.py

```python
# Define here the models for your scraped items
#
# See documentation in:
# https://docs.scrapy.org/en/latest/topics/items.html

import scrapy


class QiubaiproItem(scrapy.Item):
    # define the fields for your item here like:
    # name = scrapy.Field()
    author=scrapy.Field()
    content=scrapy.Field()
    #pass
```

### 三、setting.py

```python
# Scrapy settings for qiubaipro project
#
# For simplicity, this file contains only settings considered important or
# commonly used. You can find more settings consulting the documentation:
#
#     https://docs.scrapy.org/en/latest/topics/settings.html
#     https://docs.scrapy.org/en/latest/topics/downloader-middleware.html
#     https://docs.scrapy.org/en/latest/topics/spider-middleware.html

BOT_NAME = 'qiubaipro'

SPIDER_MODULES = ['qiubaipro.spiders']
NEWSPIDER_MODULE = 'qiubaipro.spiders'
pipeline.html
ITEM_PIPELINES = {
   'qiubaipro.pipelines.QiubaiproPipeline': 300,
}
```

### 四、pipelines.py

```python
# Define your item pipelines here
#
# Don't forget to add your pipeline to the ITEM_PIPELINES setting
# See: https://docs.scrapy.org/en/latest/topics/item-pipeline.html


# useful for handling different item types with a single interface
from itemadapter import ItemAdapter


class QiubaiproPipeline:
    fp=None
    #重写父类的一个方法：该方法只在开始爬虫的时候调用一次
    def open_spider(self,spider):
        print("开始爬虫...")
        self.fp=open('./qiubai.txt','w',encoding='utf-8')
    #专门用来处理item类型对象
    #该方法可以接收爬虫文件提交过来的item对象
    #该方法每接收到一个item就会被调用一次
    def process_item(self, item, spider):
        author=item['author']
        content=item['content']
        self.fp.write(author+':'+content+'\n')
        return item
    def close_spider(self,spider):
        print("结束爬虫！")
        self.fp.close()
```

## 9、站长素材图片爬取

### 一、img.py

```python
import scrapy
from imgpro.items import ImgproItem

class ImgSpider(scrapy.Spider):
    name = 'img'
    #allowed_domains = ['www.xxx.com']
    start_urls = ['http://sc.chinaz.com/tupian/']

    def parse(self, response):
        div_list=response.xpath('//div[@id="container"]/div')
        for div in div_list:
            #使用伪属性，只有滑动才能显示src，本身为src2
            src=div.xpath('./div/a/img/@src2').extract_first()
            item=ImgproItem()
            item['src']=src
       		yield item
```

### 二、item.py

```python
# Define here the models for your scraped items
#
# See documentation in:
# https://docs.scrapy.org/en/latest/topics/items.html

import scrapy


class ImgproItem(scrapy.Item):
    # define the fields for your item here like:
    src = scrapy.Field()
    pass
```

### 三、setting.py

```python
USER_AGENT = 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/84.0.4147.89 Safari/537.36'
ROBOTSTXT_OBEY = False
LOG_LEVEL='ERROR'
#指定图片存储目录
IMAGES_STORE='./imgs'
https://docs.scrapy.org/en/latest/topics/extensions.html
#需要改变pipelines.py文件指定的imgpipeline
ITEM_PIPELINES = {
   'imgpro.pipelines.imgpipeline': 300,
}
```

### 四、pipelines.py

```python
# Define your item pipelines here
#
# Don't forget to add your pipeline to the ITEM_PIPELINES setting
# See: https://docs.scrapy.org/en/latest/topics/item-pipeline.html


# useful for handling different item types with a single interface
from itemadapter import ItemAdapter


# class ImgproPipeline:
#     def process_item(self, item, spider):
#         return item
from scrapy.pipelines.images import ImagesPipeline
import scrapy
class imgpipeline(ImagesPipeline):
    #可以根据图片地址进行图片数据的请求
    def get_media_requests(self, item, info):
        yield scrapy.Request(item['src'])
    #指定图片存储的路径
    def file_path(self, request, response=None, info=None):
        imgname=request.url.split('/')[-1]
        return imgname
    def item_completed(self, results, item, info):
        return item #返回下一个即将执行的管道类
```

## 10、网易新闻-响应对象的篡改

### 一、wangyi.py

```python
import scrapy
from selenium import webdriver
from wangyipro.items import WangyiproItem
class WangyiSpider(scrapy.Spider):
    name = 'wangyi'
    #allowed_domains = ['www.xxx.com']
    start_urls = ['https://news.163.com/']
    models_url=[]#存储五个板块对应的url
    #解析五大板块对应的详情页url
    def __init__(self):
        self.bro=webdriver.Chrome(executable_path='E:/firefoxdownloads/chromedriver.exe')
    def parse(self, response):
        li_list=response.xpath('//*[@id="index2016_wrap"]/div[1]/div[2]/div[2]/div[2]/div[2]/div/ul/li')
        alist=[3,4,6,7,8]
        for index in alist:
            model_url=li_list[index].xpath('./a/@href').extract_first()
            self.models_url.append(model_url)
        #依次对每一个板块对应的页面进行请求
        for url in self.models_url:#对每一个板块的url进行请求发送
            yield scrapy.Request(url,callback=self.parse_model)
        #每一个板块对应的新闻标题相关的内容都是动态加载的
    def parse_model(self,response):
        #解析每一个板块对应新闻的标题和新闻详情页url
        div_list=response.xpath('/html/body/div/div[3]/div[4]/div[1]/div/div/ul/li/div/div')
        for div in div_list:
            title=div.xpath('./div/div[1]/h3/a/text()').extract_first()
            new_detail_url=div.xpath('./div/div[1]/h3/a/@href').extract_first()
            item=WangyiproItem()
            item['title']=title
            #对新闻详情页的url发起请求
            yield scrapy.Request(url=new_detail_url,callback=self.parse_detail,meta={'item':item})
    def parse_detail(self,response):#解析新闻内容
        content=response.xpath('//*[@id="endText"]//text()').extract()
        content=''.join(content)
        item=response.meta['item']
        item['content']=content
        yield item
    def closed(self,spider):
        self.bro.quit()
```

### 二、pipelines.py

```python
# Define your item pipelines here
#
# Don't forget to add your pipeline to the ITEM_PIPELINES setting
# See: https://docs.scrapy.org/en/latest/topics/item-pipeline.html


# useful for handling different item types with a single interface
from itemadapter import ItemAdapter


class WangyiproPipeline:
    def process_item(self, item, spider):
        print(item)
        return item
```

### 三、items.py

```python
# Define here the models for your scraped items
#
# See documentation in:
# https://docs.scrapy.org/en/latest/topics/items.html

import scrapy


class WangyiproItem(scrapy.Item):
    # define the fields for your item here like:
    title = scrapy.Field()
    content = scrapy.Field()
```

### 四、middlewares.py

```python
# Define here the models for your spider middleware
#
# See documentation in:
# https://docs.scrapy.org/en/latest/topics/spider-middleware.html

from scrapy import signals

# useful for handling different item types with a single interface
from itemadapter import is_item, ItemAdapter

from scrapy.http import HtmlResponse
from time import sleep
class WangyiproDownloaderMiddleware:
    # Not all methods need to be defined. If a method is not defined,
    # scrapy acts as if the downloader middleware does not modify the
    # passed objects.
    def process_request(self, request, spider):
        # Called for each request that goes through the downloader
        # middleware.

        # Must either:
        # - return None: continue processing this request
        # - or return a Response object
        # - or return a Request object
        # - or raise IgnoreRequest: process_exception() methods of
        #   installed downloader middleware will be called
        return None
    #该方法拦截五大板块对应的响应对象进行篡改
    def process_response(self, request, response, spider):#spider爬虫对象
        bro=spider.bro  #获取在爬虫中定义的浏览器对象
        #挑选出指定的响应对象进行篡改，通过url制定request，再通过request制定response
        if request.url in spider.models_url:
            bro.get(request.url)#五个板块对应的url进行请求
            sleep(2)
            page_text=bro.page_source  #包含了动态加载的新闻数据
            #response #五大板块对应的响应对象
            #针对定位到的这些response进行篡改
            #实例化一个新的响应对象（符合需求：包含动态加载出的新闻数据），代替原来旧的响应对象
            #如何获取动态加载出的新闻数据？
            new_response=HtmlResponse(url=request.url,body=page_text,encoding='utf-8',request=request)
            return new_response
        else:
            #response #其他请求对应的响应对象
            return response

    def process_exception(self, request, exception, spider):
        # Called when a download handler or a process_request()
        # (from other downloader middleware) raises an exception.

        # Must either:
        # - return None: continue processing this exception
        # - return a Response object: stops process_exception() chain
        # - return a Request object: stops process_exception() chain
        pass
```

### 五、setting.py

```python
BOT_NAME = 'wangyipro'
SPIDER_MODULES = ['wangyipro.spiders']
NEWSPIDER_MODULE = 'wangyipro.spiders'
USER_AGENT = 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/84.0.4147.89 Safari/537.36'
ROBOTSTXT_OBEY = False
LOG_LEVEL='ERROR'
DOWNLOADER_MIDDLEWARES = {
   'wangyipro.middlewares.WangyiproDownloaderMiddleware': 543,
}
ITEM_PIPELINES = {
   'wangyipro.pipelines.WangyiproPipeline': 300,
}
```

