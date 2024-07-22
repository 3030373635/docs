---
title: scrapy
---




# 一、介绍

> Scrapy一个开源和协作的框架，其最初是为了页面抓取 (更确切来说, 网络抓取 )所设计的，使用它可以以快速、简单、可扩展的方式从网站中提取所需的数据。但目前Scrapy的用途十分广泛，可用于如数据挖掘、监测和自动化测试等领域，也可以应用在获取API所返回的数据(例如 Amazon Associates Web Services ) 或者通用的网络爬虫。
>
> Scrapy 是基于twisted框架开发而来，twisted是一个流行的事件驱动的python网络框架。因此Scrapy使用了一种非阻塞（又名异步）的代码来实现并发。整体架构大致如下

<img src="https://cos.liuqm.cc/1-1-20221011223104110.png">

```python
1. 引擎(EGINE)

   引擎负责控制系统所有组件之间的数据流，并在某些动作发生时触发事件。有关详细信息，请参见上面的数据流部分。

2. 调度器(SCHEDULER)
   用来接受引擎发过来的请求, 压入队列中, 并在引擎再次请求的时候返回. 可以想像成一个URL的优先级队列, 由它来决定下一个要抓取的网址是什么, 同时去除重复的网址

3. 下载器(DOWLOADER)
   用于下载网页内容, 并将网页内容返回给EGINE，下载器是建立在twisted这个高效的异步模型上的

4. 爬虫(SPIDERS)
   SPIDERS是开发人员自定义的类，用来解析responses，并且提取items，或者发送新的请求

5. 项目管道(ITEM PIPLINES)
   在items被提取后负责处理它们，主要包括清理、验证、持久化（比如存到数据库）等操作

6. 下载器中间件(Downloader Middlewares)

  	爬虫中间件：位于爬虫与引擎之间
    下载器中间件：位于下载器与
    你可用该中间件做以下几件事

   1. process a request just before it is sent to the Downloader (i.e. right before Scrapy sends the request to the website);
   2. change received response before passing it to a spider;
   3. send a new Request instead of passing received response to a spider;
   4. pass response to a spider without fetching a web page;
   5. silently drop some requests.

7. 爬虫中间件(Spider Middlewares)
   位于EGINE和SPIDERS之间，主要工作是处理SPIDERS的输入（即responses）和输出（即requests）
```

**scrapy运行大致流程**

```
1. 引擎从调度器中取出一个链接(URL)用于接下来的抓取
2. 引擎把URL封装成一个请求(Request)传给下载器
3. 下载器把资源下载下来，并封装成应答包(Response)
4. 爬虫解析Response
5. 解析出实体（Item）,则交给实体管道进行进一步的处理
6. 解析出的是链接（URL）,则把URL交给调度器等待抓取
```

# 二、安装

```python
#Windows平台
    1、pip3 install wheel #安装后，便支持通过wheel文件安装软件，wheel文件官网：https://www.lfd.uci.edu/~gohlke/pythonlibs
    3、pip3 install lxml
    4、pip3 install pyopenssl
    5、下载并安装pywin32：https://sourceforge.net/projects/pywin32/files/pywin32/
    6、下载twisted的wheel文件：http://www.lfd.uci.edu/~gohlke/pythonlibs/#twisted
    7、执行pip3 install 下载目录\Twisted-17.9.0-cp36-cp36m-win_amd64.whl
    8、pip3 install scrapy
  
#Linux平台
    1、pip3 install scrapy
```

# 三、命令行工具

```python
#1 查看帮助
    scrapy -h
    scrapy <command> -h

#2 有两种命令：其中Project-only必须切到项目文件夹下才能执行，而Global的命令则不需要
    Global commands:
        startproject #创建项目
        genspider    #创建爬虫程序 
        settings     #如果是在项目目录下，则得到的是该项目的配置
        runspider    #运行一个独立的python文件，不必创建项目
        shell        #scrapy shell url地址  在交互式调试，如选择器规则正确与否
        fetch        #独立于程单纯地爬取一个页面，可以拿到请求头
        view         #下载完毕后直接弹出浏览器，以此可以分辨出哪些数据是ajax请求
        version      #scrapy version 查看scrapy的版本，scrapy version -v查看scrapy依赖库的版本
    Project-only commands:
        crawl        #运行爬虫，必须创建项目才行，确保配置文件中ROBOTSTXT_OBEY = False(不遵循爬虫协议)
        check        #检测项目中有无语法错误
        list         #列出项目中所包含的爬虫名
        edit         #编辑器，一般不用
        parse        #scrapy parse url地址 --callback 回调函数  #以此可以验证我们的回调函数是否正确
        bench        #scrapy bentch压力测试

#3 官网链接
    https://docs.scrapy.org/en/latest/topics/commands.html
```

**简单使用**

```python
scrapy startproject MyProject # 创建项目
cd MyProject
scrapy genspider baidu www.baidu.com #创建爬虫程序 
scrapy crawl baidu # 开始运行爬虫程序
```

**示范使用**

```python
#1、执行全局命令：请确保不在某个项目的目录下，排除受该项目配置的影响
scrapy startproject MyProject # 创建项目

cd MyProject
scrapy genspider baidu www.baidu.com #创建爬虫程序 

scrapy settings --get XXX #如果切换到项目目录下，看到的则是该项目的配置

scrapy runspider baidu.py

scrapy shell https://www.baidu.com
    response
    response.status
    response.body
    view(response)
  
scrapy view https://www.taobao.com #如果页面显示内容不全，不全的内容则是ajax请求实现的，以此快速定位问题

scrapy fetch --nolog --headers https://www.taobao.com

scrapy version #scrapy的版本

scrapy version -v #依赖库的版本


#2、执行项目命令：切到项目目录下
scrapy crawl baidu
scrapy check
scrapy list
scrapy parse http://quotes.toscrape.com/ --callback parse
scrapy bench
```

# 四、项目结构

**注意：一般创建爬虫文件时，以网站域名命名**

```python
firstscrapy  # 项目名字
        firstscrapy # 包，跟项目同名，是不是发现跟django的目录很相似
            -spiders # 所有的爬虫文件放在里面
                -baidu.py # 一个个的爬虫（以后基本上都在这写东西）
                -chouti.py# 一个个的爬虫（以后基本上都在这写东西）
            -middlewares.py # 中间件（爬虫，下载中间件都写在这）
            -pipelines.py   # 持久化相关写在这（items.py中类的对象）
            -main.py        # 自己加的，执行爬虫
            -items.py       # 设置数据存储模板，用于结构化数据，如：Django的Model
            -settings.py    # 配置文件，如：递归的层数、并发数，延迟下载等。强调:配置文件的选项必须大写否则视为无效，正确写法USER_AGENT='xxxx'
        scrapy.cfg          # 上线相关
```

# 五、spiders

## 5.1 介绍

```python
#1、Spiders是由一系列类（定义了一个网址或一组网址将被爬取）组成，具体包括如何执行爬取任务并且如何从页面中提取结构化的数据。

#2、换句话说，Spiders是你为了一个特定的网址或一组网址自定义爬取和解析页面行为的地方
```

## 5.2 Spiders会循环做如下事情

```python
#1、生成初始的Requests来爬取第一个URLS，并且标识一个回调函数
第一个请求定义在start_requests()方法内默认从start_urls列表中获得url地址来生成Request请求，默认的回调函数是parse方法。回调函数在下载完成返回response时自动触发

#2、在回调函数中，解析response并且返回值
返回值可以4种：
        包含解析数据的字典
        Item对象
        新的Request对象（新的Requests也需要指定一个回调函数）
        或者是可迭代对象（包含Items或Request）

#3、在回调函数中解析页面内容
通常使用Scrapy自带的Selectors，但很明显你也可以使用Beutifulsoup，lxml或其他你爱用啥用啥。

#4、最后，针对返回的Items对象将会被持久化到数据库
通过Item Pipeline组件存到数据库：https://docs.scrapy.org/en/latest/topics/item-pipeline.html#topics-item-pipeline）
或者导出到不同的文件（通过Feed exports：https://docs.scrapy.org/en/latest/topics/feed-exports.html#topics-feed-exports）
```

## 5.3 Spiders总共提供了五种类

> 主要使用第一种

```python
#1、scrapy.spiders.Spider #scrapy.Spider等同于scrapy.spiders.Spider
#2、scrapy.spiders.CrawlSpider
#3、scrapy.spiders.XMLFeedSpider
#4、scrapy.spiders.CSVFeedSpider
#5、scrapy.spiders.SitemapSpider
```

## 5.4 导入使用

```python
# -*- coding: utf-8 -*-
import scrapy
from scrapy.spiders import Spider,CrawlSpider,XMLFeedSpider,CSVFeedSpider,SitemapSpider

class AmazonSpider(scrapy.Spider): #自定义类，继承Spiders提供的基类
    name = 'amazon'
    allowed_domains = ['www.amazon.cn']
    start_urls = ['http://www.amazon.cn/']
  
    def parse(self, response):
        pass
```

## 5.5 scrapy.spiders.Spider

> 这是最简单的spider类，任何其他的spider类都需要继承它（包含你自己定义的）。
>
> 该类不提供任何特殊的功能，它仅提供了一个默认的start_requests方法默认从start_urls中读取url地址发送requests请求，并且默认parse作为回调函数

```python
class AmazonSpider(scrapy.Spider):
    name = 'amazon' 
  
    allowed_domains = ['www.amazon.cn'] 
  
    start_urls = ['http://www.amazon.cn/']
  
    custom_settings = {
        'BOT_NAME' : 'Egon_Spider_Amazon',
        'REQUEST_HEADERS' : {
          'Accept': 'text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8',
          'Accept-Language': 'en',
        }
    }
  
    def parse(self, response):
        pass
```

### 5.5.1 scrapy.spider属性与方法详解

```python
#1、name = 'amazon' 
定义爬虫名，scrapy会根据该值定位爬虫程序
所以它必须要有且必须唯一（In Python 2 this must be ASCII only.）

#2、allowed_domains = ['www.amazon.cn'] 
定义允许爬取的域名，如果OffsiteMiddleware启动（默认就启动），
那么不属于该列表的域名及其子域名都不允许爬取
如果爬取的网址为：https://www.example.com/1.html，那就添加'example.com'到列表.

#3、start_urls = ['http://www.amazon.cn/']
如果没有指定url，就从该列表中读取url来生成第一个请求

#4、custom_settings
值为一个字典，定义一些配置信息，在运行爬虫程序时，这些配置会覆盖项目级别的配置
所以custom_settings必须被定义成一个类属性，settings会在类实例化前被加载

#5、settings
通过self.settings['配置项的名字']可以访问settings.py中的配置，如果自己定义了custom_settings还是以自己的为准

#6、logger
日志名默认为spider的名字
self.logger.debug('=============>%s' %self.settings['BOT_NAME'])

#5、crawler：了解
该属性必须被定义到类方法from_crawler中

#6、from_crawler(crawler, *args, **kwargs)：了解
You probably won’t need to override this directly  because the default implementation acts as a proxy to the __init__() method, calling it with the given arguments args and named arguments kwargs.

#7、start_requests()
该方法用来发起第一个Requests请求，且必须返回一个可迭代的对象。它在爬虫程序打开时就被Scrapy调用，Scrapy只调用它一次。
当没有写该函数时，默认从start_urls里取出每个url来生成Request(url, dont_filter=True)
如果你想要改变起始爬取的Requests，你就需要覆盖这个方法，例如你想要起始发送一个GET请求，如下
class MySpider(scrapy.Spider):
    name = 'amazon'

    def parse(self, response):
        print("%s %s" %(response.url,len(response.body)))

    def start_requests(self):
        yield scrapy.Request("https://www.amazon.cn/s?k=iphone12",callback=self.parse)
        yield scrapy.Request("https://www.amazon.cn/s?k=iphone12",callback=self.parse)
        yield scrapy.Request("https://www.amazon.cn/s?k=iphone12",callback=self.parse)





#8针对参数dont_filter,请看5.5.2节自定义去重规则


    
#9、parse(response)
这是默认的回调函数，所有的回调函数必须返回an iterable of Request and/or dicts or Item objects.

#10、log(message[, level, component])：了解
Wrapper that sends a log message through the Spider’s logger, kept for backwards compatibility. For more information see Logging from Spiders.

#11、closed(reason)
爬虫程序结束时自动触发
```

### 5.5.2 去除重复的url

```python
去重规则应该多个爬虫共享的，但凡一个爬虫爬取了，其他都不要爬了，实现方式如下

#方法一：
1、新增类属性
visited=set() #类属性

2、回调函数parse方法内：
def parse(self, response):
    if response.url in self.visited:
        return None
    .......

    self.visited.add(response.url) 

#方法一改进：针对url可能过长，所以我们存放url的hash值
def parse(self, response):
        url=md5(response.request.url)
    if url in self.visited:
        return None
    .......

    self.visited.add(url) 

#方法二：Scrapy自带去重功能
配置文件：
DUPEFILTER_CLASS = 'scrapy.dupefilters.RFPDupeFilter' #默认的去重规则帮我们去重，去重规则在内存中
DUPEFILTER_DEBUG = False
JOBDIR = "保存范文记录的日志路径，如：/root/"  # 最终路径为 /root/requests.seen，去重规则放文件中

scrapy自带去重规则默认为RFPDupeFilter，只需要我们指定
Request(...,dont_filter=False) ，如果dont_filter=True则告诉Scrapy这个URL不参与去重。

#方法三：
我们也可以仿照RFPDupeFilter自定义去重规则，

from scrapy.dupefilter import RFPDupeFilter，看源码，仿照BaseDupeFilter

#步骤一：在项目目录下自定义去重文件dup.py
class UrlFilter(object):
    def __init__(self):
        self.visited = set() #或者放到数据库

    @classmethod
    def from_settings(cls, settings):
        return cls()

    def request_seen(self, request):
        if request.url in self.visited:
            return True
        self.visited.add(request.url)

    def open(self):  # can return deferred
        pass

    def close(self, reason):  # can return a deferred
        pass

    def log(self, request, spider):  # log that a request has been filtered
        pass

#步骤二：配置文件settings.py：
DUPEFILTER_CLASS = '项目名.dup.UrlFilter'


# 源码分析：
from scrapy.core.scheduler import Scheduler
见Scheduler下的enqueue_request方法：self.df.request_seen(request)
```

### 5.5.3 栗子

```python
例一：
import scrapy

class MySpider(scrapy.Spider):
    name = 'example.com'
    allowed_domains = ['example.com']
    start_urls = [
        'http://www.example.com/1.html',
        'http://www.example.com/2.html',
        'http://www.example.com/3.html',
    ]

    def parse(self, response):
        self.logger.info('A response from %s just arrived!', response.url)
    
  
#例二：一个回调函数返回多个Requests和Items
import scrapy

class MySpider(scrapy.Spider):
    name = 'example.com'
    allowed_domains = ['example.com']
    start_urls = [
        'http://www.example.com/1.html',
        'http://www.example.com/2.html',
        'http://www.example.com/3.html',
    ]

    def parse(self, response):
        for h3 in response.xpath('//h3').extract():
            yield {"title": h3}

        for url in response.xpath('//a/@href').extract():
            yield scrapy.Request(url, callback=self.parse)
        
        
#例三：在start_requests()内直接指定起始爬取的urls，start_urls就没有用了，

import scrapy
from myproject.items import MyItem

class MySpider(scrapy.Spider):
    name = 'example.com'
    allowed_domains = ['example.com']

    def start_requests(self):
        yield scrapy.Request('http://www.example.com/1.html', self.parse)
        yield scrapy.Request('http://www.example.com/2.html', self.parse)
        yield scrapy.Request('http://www.example.com/3.html', self.parse)

    def parse(self, response):
        for h3 in response.xpath('//h3').extract():
            yield MyItem(title=h3)

        for url in response.xpath('//a/@href').extract():
            yield scrapy.Request(url, callback=self.parse)
```

### 5.5.4 参数传递

```python
我们可能需要在命令行为爬虫程序传递参数，比如传递初始的url，像这样
#命令行执行
scrapy crawl myspider -a category=electronics

#在__init__方法中可以接收外部传进来的参数
import scrapy

class MySpider(scrapy.Spider):
    name = 'myspider'

    def __init__(self, category=None, *args, **kwargs):
        super(MySpider, self).__init__(*args, **kwargs)
        self.start_urls = ['http://www.example.com/categories/%s' % category]
        #...

    
#注意接收的参数全都是字符串，如果想要结构化的数据，你需要用类似json.loads的方法
```

### 5.5.5 其他通用spiders

> https://docs.scrapy.org/en/latest/topics/spiders.html#generic-spiders

# 六、Selectors

> https://docs.scrapy.org/en/latest/topics/selectors.html
>
> 解析器，解析文档，提取信息
>
> 这里主要有两种解析器：
>
> - css解析器
> - xpath解析器
>
> 注意：
> response.selector.css()
> response.selector.xpath()
>
> 可简写为
> response.css()
> response.xpath()

```python
"""
  #1、//与/
  #2、text：获取标签里面的文本
  #3、extract与extract_first:从selector对象中解出内容
  #4、属性：xpath的属性加前缀@
  #4、嵌套查找
  #5、设置默认值
  #4、按照属性查找
  #5、按照属性模糊查找
  #6、正则表达式
  #7、xpath相对路径
  #8、带变量的xpath
"""


#1、//与/：//代表查找所有的，子子孙孙的，/代表只查找儿子
response.xpath('//body/a/')
response.css('div a::text')

>>> response.xpath('//body/a') #如果//在开头，代表从整篇文档中寻找。body之后的/代表body的儿子
[]
>>> response.xpath('//body//a') #如果//在开头，代表从整篇文档中寻找。body之后的//代表body的子子孙孙
[<Selector xpath='//body//a' data='<a href="image1.html">Name: My image 1 <'>, <Selector xpath='//body//a' data='<a href="image2.html">Name: My image 2 <'>, <Selector xpath='//body//a' data='<a href="
image3.html">Name: My image 3 <'>, <Selector xpath='//body//a' data='<a href="image4.html">Name: My image 4 <'>, <Selector xpath='//body//a' data='<a href="image5.html">Name: My image 5 <'>]

#2 text
>>> response.xpath('//body//a/text()')
>>> response.css('body a::text')

#3、extract与extract_first:从selector对象中解出内容
>>> response.xpath('//div/a/text()').extract()
['Name: My image 1 ', 'Name: My image 2 ', 'Name: My image 3 ', 'Name: My image 4 ', 'Name: My image 5 ']
>>> response.css('div a::text').extract()
['Name: My image 1 ', 'Name: My image 2 ', 'Name: My image 3 ', 'Name: My image 4 ', 'Name: My image 5 ']

>>> response.xpath('//div/a/text()').extract_first()
'Name: My image 1 '
>>> response.css('div a::text').extract_first()
'Name: My image 1 '

#4、属性：xpath的属性加前缀@
>>> response.xpath('//div/a/@href').extract_first()
'image1.html'
>>> response.css('div a::attr(href)').extract_first()
'image1.html'

#4、嵌套查找
>>> response.xpath('//div').css('a').xpath('@href').extract_first()
'image1.html'

#5、设置默认值
>>> response.xpath('//div[@id="xxx"]').extract_first(default="not found")
'not found'

#4、按照属性查找
response.xpath('//div[@id="images"]/a[@href="image3.html"]/text()').extract()
response.css('#images a[@href="image3.html"]/text()').extract()

#5、按照属性模糊查找
response.xpath('//a[contains(@href,"image")]/@href').extract()
response.css('a[href*="image"]::attr(href)').extract()

response.xpath('//a[contains(@href,"image")]/img/@src').extract()
response.css('a[href*="imag"] img::attr(src)').extract()

response.xpath('//*[@href="image1.html"]')
response.css('*[href="image1.html"]')

#6、正则表达式
response.xpath('//a/text()').re(r'Name: (.*)')
response.xpath('//a/text()').re_first(r'Name: (.*)')

#7、xpath相对路径
>>> res=response.xpath('//a[contains(@href,"3")]')[0]
>>> res.xpath('img')
[<Selector xpath='img' data='<img src="image3_thumb.jpg">'>]
>>> res.xpath('./img')
[<Selector xpath='./img' data='<img src="image3_thumb.jpg">'>]
>>> res.xpath('.//img')
[<Selector xpath='.//img' data='<img src="image3_thumb.jpg">'>]
>>> res.xpath('//img') #这就是从头开始扫描
[<Selector xpath='//img' data='<img src="image1_thumb.jpg">'>, <Selector xpath='//img' data='<img src="image2_thumb.jpg">'>, <Selector xpath='//img' data='<img src="image3_thumb.jpg">'>, <Selector xpa
th='//img' data='<img src="image4_thumb.jpg">'>, <Selector xpath='//img' data='<img src="image5_thumb.jpg">'>]

#8、带变量的xpath
>>> response.xpath('//div[@id=$xxx]/a/text()',xxx='images').extract_first()
'Name: My image 1 '
>>> response.xpath('//div[count(a)=$yyy]/@id',yyy=5).extract_first() #求有5个a标签的div的id
'images'
```

# 七、items

> https://docs.scrapy.org/en/latest/topics/items.html

# 八、 Item Pipeline

**自定义pipeline**

```python
#一：可以写多个Pipeline类
#1、如果优先级高的Pipeline的process_item返回一个值或者None，会自动传给下一个pipline的process_item,
#2、如果只想让第一个Pipeline执行，那得让第一个pipline的process_item抛出异常raise DropItem()

#3、可以用spider.name == '爬虫名' 来控制哪些爬虫用哪些pipeline

二：示范
from scrapy.exceptions import DropItem

class CustomPipeline(object):
    def __init__(self,v):
        self.value = v

    @classmethod
    def from_crawler(cls, crawler):
        """
        Scrapy会先通过getattr判断我们是否自定义了from_crawler,有则调它来完
        成实例化
        """
        val = crawler.settings.getint('MMMM')
        return cls(val)

    def open_spider(self,spider):
        """
        爬虫刚启动时执行一次
        """
        print('000000')

    def close_spider(self,spider):
        """
        爬虫关闭时执行一次
        """
        print('111111')


    def process_item(self, item, spider):
        # 操作并进行持久化

        # return表示会被后续的pipeline继续处理
        return item

        # 表示将item丢弃，不会被后续pipeline处理
        # raise DropItem()
```

**示范**

```python
#1、settings.py
HOST="127.0.0.1"
PORT=27017
USER="root"
PWD="123"
DB="amazon"
TABLE="goods"



ITEM_PIPELINES = {
   'Amazon.pipelines.CustomPipeline': 200,
}

#2、pipelines.py
class CustomPipeline(object):
    def __init__(self,host,port,user,pwd,db,table):
        self.host=host
        self.port=port
        self.user=user
        self.pwd=pwd
        self.db=db
        self.table=table

    @classmethod
    def from_crawler(cls, crawler):
        """
        Scrapy会先通过getattr判断我们是否自定义了from_crawler,有则调它来完
        成实例化
        """
        HOST = crawler.settings.get('HOST')
        PORT = crawler.settings.get('PORT')
        USER = crawler.settings.get('USER')
        PWD = crawler.settings.get('PWD')
        DB = crawler.settings.get('DB')
        TABLE = crawler.settings.get('TABLE')
        return cls(HOST,PORT,USER,PWD,DB,TABLE)

    def open_spider(self,spider):
        """
        爬虫刚启动时执行一次
        """
        self.client = MongoClient('mongodb://%s:%s@%s:%s' %(self.user,self.pwd,self.host,self.port))

    def close_spider(self,spider):
        """
        爬虫关闭时执行一次
        """
        self.client.close()


    def process_item(self, item, spider):
        # 操作并进行持久化

        self.client[self.db][self.table].save(dict(item))
```

> https://docs.scrapy.org/en/latest/topics/item-pipeline.html

# 九、Dowloader Middeware

> 无论在哪个阶段返回request，都会直接交给引擎
>
> request阶段，如果返回response，那么会从最后一个中间件向前一直执行response方法
>
> request阶段，如果抛异常，那么会从最后一个中间件向前一直执行response方法

```python
下载中间件的用途
    1、在process——request内，自定义下载，不用scrapy的下载
    2、对请求进行二次加工，比如
        设置请求头
        设置cookie
        添加代理
            scrapy自带的代理组件：
                from scrapy.downloadermiddlewares.httpproxy import HttpProxyMiddleware
                from urllib.request import getproxies
```

**下载器中间件**

```python
class DownMiddleware1(object):
    def process_request(self, request, spider):
        """
        请求需要被下载时，经过所有下载器中间件的process_request调用
        :param request: 
        :param spider: 
        :return:  
            None,继续后续中间件去下载；
            Response对象，停止process_request的执行，开始执行process_response
            Request对象，停止中间件的执行，将Request重新调度器
            raise IgnoreRequest异常，停止process_request的执行，开始执行process_exception
        """
        pass



    def process_response(self, request, response, spider):
        """
        spider处理完成，返回时调用
        :param response:
        :param result:
        :param spider:
        :return: 
            Response 对象：转交给其他中间件process_response
            Request 对象：停止中间件，request会被重新调度下载
            raise IgnoreRequest 异常：调用Request.errback
        """
        print('response1')
        return response

    def process_exception(self, request, exception, spider):
        """
        当下载处理器(download handler)或 process_request() (下载中间件)抛出异常
        :param response:
        :param exception:
        :param spider:
        :return: 
            None：继续交给后续中间件处理异常；
            Response对象：停止后续process_exception方法
            Request对象：停止中间件，request将会被重新调用下载
        """
        return None
```

**配置代理**

```python
#1、与middlewares.py同级目录下新建proxy_handle.py
import requests

def get_proxy():
    return requests.get("http://127.0.0.1:5010/get/").text

def delete_proxy(proxy):
    requests.get("http://127.0.0.1:5010/delete/?proxy={}".format(proxy))
  
  

#2、middlewares.py
from Amazon.proxy_handle import get_proxy,delete_proxy

class DownMiddleware1(object):
    def process_request(self, request, spider):
        """
        请求需要被下载时，经过所有下载器中间件的process_request调用
        :param request:
        :param spider:
        :return:
            None,继续后续中间件去下载；
            Response对象，停止process_request的执行，开始执行process_response
            Request对象，停止中间件的执行，将Request重新调度器
            raise IgnoreRequest异常，停止process_request的执行，开始执行process_exception
        """
        proxy="http://" + get_proxy()
        request.meta['download_timeout']=20
        request.meta["proxy"] = proxy
        print('为%s 添加代理%s ' % (request.url, proxy),end='')
        print('元数据为',request.meta)

    def process_response(self, request, response, spider):
        """
        spider处理完成，返回时调用
        :param response:
        :param result:
        :param spider:
        :return:
            Response 对象：转交给其他中间件process_response
            Request 对象：停止中间件，request会被重新调度下载
            raise IgnoreRequest 异常：调用Request.errback
        """
        print('返回状态吗',response.status)
        return response


    def process_exception(self, request, exception, spider):
        """
        当下载处理器(download handler)或 process_request() (下载中间件)抛出异常
        :param response:
        :param exception:
        :param spider:
        :return:
            None：继续交给后续中间件处理异常；
            Response对象：停止后续process_exception方法
            Request对象：停止中间件，request将会被重新调用下载
        """
        print('代理%s，访问%s出现异常:%s' %(request.meta['proxy'],request.url,exception))
        import time
        time.sleep(5)
        delete_proxy(request.meta['proxy'].split("//")[-1])
        request.meta['proxy']='http://'+get_proxy()

        return request
```

# 十、Spider Middleware

**1、爬虫中间件方法介绍**

```python
from scrapy import signals

class SpiderMiddleware(object):
    # Not all methods need to be defined. If a method is not defined,
    # scrapy acts as if the spider middleware does not modify the
    # passed objects.

    @classmethod
    def from_crawler(cls, crawler):
        # This method is used by Scrapy to create your spiders.
        s = cls()
        crawler.signals.connect(s.spider_opened, signal=signals.spider_opened) #当前爬虫执行时触发spider_opened
        return s

    def spider_opened(self, spider):
        # spider.logger.info('我是egon派来的爬虫1: %s' % spider.name)
        print('我是egon派来的爬虫1: %s' % spider.name)

    def process_start_requests(self, start_requests, spider):
        # Called with the start requests of the spider, and works
        # similarly to the process_spider_output() method, except
        # that it doesn’t have a response associated.

        # Must return only requests (not items).
        print('start_requests1')
        for r in start_requests:
            yield r

    def process_spider_input(self, response, spider):
        # Called for each response that goes through the spider
        # middleware and into the spider.
        # 每个response经过爬虫中间件进入spider时调用

        # 返回值：Should return None or raise an exception.
        #1、None: 继续执行其他中间件的process_spider_input
        #2、抛出异常：
        # 一旦抛出异常则不再执行其他中间件的process_spider_input
        # 并且触发request绑定的errback
        # errback的返回值倒着传给中间件的process_spider_output
        # 如果未找到errback，则倒着执行中间件的process_spider_exception

        print("input1")
        return None

    def process_spider_output(self, response, result, spider):
        # Called with the results returned from the Spider, after
        # it has processed the response.

        # Must return an iterable of Request, dict or Item objects.
        print('output1')

        # 用yield返回多次，与return返回一次是一个道理
        # 如果生成器掌握不好（函数内有yield执行函数得到的是生成器而并不会立刻执行），生成器的形式会容易误导你对中间件执行顺序的理解
        # for i in result:
        #     yield i
        return result

    def process_spider_exception(self, response, exception, spider):
        # Called when a spider or process_spider_input() method
        # (from other spider middleware) raises an exception.

        # Should return either None or an iterable of Response, dict
        # or Item objects.
        print('exception1')
```

**2、当前爬虫启动时以及初始请求产生时**

```python
#步骤一：
'''
打开注释：
SPIDER_MIDDLEWARES = {
   'Baidu.middlewares.SpiderMiddleware1': 200,
   'Baidu.middlewares.SpiderMiddleware2': 300,
   'Baidu.middlewares.SpiderMiddleware3': 400,
}

'''


#步骤二：middlewares.py
from scrapy import signals

class SpiderMiddleware1(object):
    @classmethod
    def from_crawler(cls, crawler):
        s = cls()
        crawler.signals.connect(s.spider_opened, signal=signals.spider_opened) #当前爬虫执行时触发spider_opened
        return s

    def spider_opened(self, spider):
        print('我是egon派来的爬虫1: %s' % spider.name)

    def process_start_requests(self, start_requests, spider):
        # Must return only requests (not items).
        print('start_requests1')
        for r in start_requests:
            yield r


    
    
class SpiderMiddleware2(object):
    @classmethod
    def from_crawler(cls, crawler):
        s = cls()
        crawler.signals.connect(s.spider_opened, signal=signals.spider_opened)  # 当前爬虫执行时触发spider_opened
        return s

    def spider_opened(self, spider):
        print('我是egon派来的爬虫2: %s' % spider.name)

    def process_start_requests(self, start_requests, spider):
        print('start_requests2')
        for r in start_requests:
            yield r


class SpiderMiddleware3(object):
    @classmethod
    def from_crawler(cls, crawler):
        s = cls()
        crawler.signals.connect(s.spider_opened, signal=signals.spider_opened)  # 当前爬虫执行时触发spider_opened
        return s

    def spider_opened(self, spider):
        print('我是egon派来的爬虫3: %s' % spider.name)

    def process_start_requests(self, start_requests, spider):
        print('start_requests3')
        for r in start_requests:
            yield r


#步骤三：分析运行结果
#1、启动爬虫时则立刻执行：

我是egon派来的爬虫1: baidu
我是egon派来的爬虫2: baidu
我是egon派来的爬虫3: baidu


#2、然后产生一个初始的request请求，依次经过爬虫中间件1,2,3：
start_requests1
start_requests2
start_requests3
```

**3、process_spider_input返回None时**

```python
#步骤一：打开注释：
SPIDER_MIDDLEWARES = {
   'Baidu.middlewares.SpiderMiddleware1': 200,
   'Baidu.middlewares.SpiderMiddleware2': 300,
   'Baidu.middlewares.SpiderMiddleware3': 400,
}

'''

#步骤二：middlewares.py
from scrapy import signals

class SpiderMiddleware1(object):

    def process_spider_input(self, response, spider):
        print("input1")

    def process_spider_output(self, response, result, spider):
        print('output1')
        return result

    def process_spider_exception(self, response, exception, spider):
        print('exception1')


class SpiderMiddleware2(object):

    def process_spider_input(self, response, spider):
        print("input2")
        return None

    def process_spider_output(self, response, result, spider):
        print('output2')
        return result

    def process_spider_exception(self, response, exception, spider):
        print('exception2')


class SpiderMiddleware3(object):

    def process_spider_input(self, response, spider):
        print("input3")
        return None

    def process_spider_output(self, response, result, spider):
        print('output3')
        return result

    def process_spider_exception(self, response, exception, spider):
        print('exception3')


#步骤三：运行结果分析

#1、返回response时，依次经过爬虫中间件1,2,3
input1
input2
input3

#2、spider处理完毕后，依次经过爬虫中间件3,2,1
output3
output2
output1
```

**4、process_spider_input抛出异常时**

```python
#步骤一：
'''
打开注释：
SPIDER_MIDDLEWARES = {
   'Baidu.middlewares.SpiderMiddleware1': 200,
   'Baidu.middlewares.SpiderMiddleware2': 300,
   'Baidu.middlewares.SpiderMiddleware3': 400,
}

'''

#步骤二：middlewares.py

from scrapy import signals

class SpiderMiddleware1(object):

    def process_spider_input(self, response, spider):
        print("input1")

    def process_spider_output(self, response, result, spider):
        print('output1')
        return result

    def process_spider_exception(self, response, exception, spider):
        print('exception1')


class SpiderMiddleware2(object):

    def process_spider_input(self, response, spider):
        print("input2")
        raise Type

    def process_spider_output(self, response, result, spider):
        print('output2')
        return result

    def process_spider_exception(self, response, exception, spider):
        print('exception2')


class SpiderMiddleware3(object):

    def process_spider_input(self, response, spider):
        print("input3")
        return None

    def process_spider_output(self, response, result, spider):
        print('output3')
        return result

    def process_spider_exception(self, response, exception, spider):
        print('exception3')

    

#运行结果    
input1
input2
exception3
exception2
exception1

#分析：
#1、当response经过中间件1的 process_spider_input返回None，继续交给中间件2的process_spider_input
#2、中间件2的process_spider_input抛出异常，则直接跳过后续的process_spider_input，将异常信息传递给Spiders里该请求的errback
#3、没有找到errback，则该response既没有被Spiders正常的callback执行，也没有被errback执行，即Spiders啥事也没有干，那么开始倒着执行process_spider_exception
#4、如果process_spider_exception返回None，代表该方法推卸掉责任，并没处理异常，而是直接交给下一个process_spider_exception，全都返回None，则异常最终交给Engine抛出
```

**5、指定errback**

```python
#步骤一：spider.py
import scrapy


class BaiduSpider(scrapy.Spider):
    name = 'baidu'
    allowed_domains = ['www.baidu.com']
    start_urls = ['http://www.baidu.com/']


    def start_requests(self):
        yield scrapy.Request(url='http://www.baidu.com/',
                             callback=self.parse,
                             errback=self.parse_err,
                             )

    def parse(self, response):
        pass

    def parse_err(self,res):
        #res 为异常信息，异常已经被该函数处理了，因此不会再抛给因此，于是开始走process_spider_output
        return [1,2,3,4,5] #提取异常信息中有用的数据以可迭代对象的形式存放于管道中，等待被process_spider_output取走



#步骤二：
'''
打开注释：
SPIDER_MIDDLEWARES = {
   'Baidu.middlewares.SpiderMiddleware1': 200,
   'Baidu.middlewares.SpiderMiddleware2': 300,
   'Baidu.middlewares.SpiderMiddleware3': 400,
}

'''

#步骤三：middlewares.py

from scrapy import signals

class SpiderMiddleware1(object):

    def process_spider_input(self, response, spider):
        print("input1")

    def process_spider_output(self, response, result, spider):
        print('output1',list(result))
        return result

    def process_spider_exception(self, response, exception, spider):
        print('exception1')


class SpiderMiddleware2(object):

    def process_spider_input(self, response, spider):
        print("input2")
        raise TypeError('input2 抛出异常')

    def process_spider_output(self, response, result, spider):
        print('output2',list(result))
        return result

    def process_spider_exception(self, response, exception, spider):
        print('exception2')


class SpiderMiddleware3(object):

    def process_spider_input(self, response, spider):
        print("input3")
        return None

    def process_spider_output(self, response, result, spider):
        print('output3',list(result))
        return result

    def process_spider_exception(self, response, exception, spider):
        print('exception3')



#步骤四：运行结果分析
input1
input2
output3 [1, 2, 3, 4, 5] #parse_err的返回值放入管道中，只能被取走一次，在output3的方法内可以根据异常信息封装一个新的request请求
output2 []
output1 []
```

# 十一、自定义扩展

```python
自定义扩展（与django的信号类似）
    1、django的信号是django是预留的扩展，信号一旦被触发，相应的功能就会执行
    2、scrapy自定义扩展的好处是可以在任意我们想要的位置添加功能，而其他组件中提供的功能只能在规定的位置执行
```

```python
#1、在与settings同级目录下新建一个文件，文件名可以为extentions.py,内容如下
from scrapy import signals


class MyExtension(object):
    def __init__(self, value):
        self.value = value

    @classmethod
    def from_crawler(cls, crawler):
        val = crawler.settings.getint('MMMM')
        obj = cls(val)

        crawler.signals.connect(obj.spider_opened, signal=signals.spider_opened)
        crawler.signals.connect(obj.spider_closed, signal=signals.spider_closed)

        return obj

    def spider_opened(self, spider):
        print('=============>open')

    def spider_closed(self, spider):
        print('=============>close')

#2、配置生效
EXTENSIONS = {
    "Amazon.extentions.MyExtension":200
}
```

# 十二、settings.py

```python
#==>第一部分：基本配置<===
#1、项目名称，默认的USER_AGENT由它来构成，也作为日志记录的日志名
BOT_NAME = 'Amazon'

#2、爬虫应用路径
SPIDER_MODULES = ['Amazon.spiders']
NEWSPIDER_MODULE = 'Amazon.spiders'

#3、客户端User-Agent请求头
#USER_AGENT = 'Amazon (+http://www.yourdomain.com)'

#4、是否遵循爬虫协议
# Obey robots.txt rules
ROBOTSTXT_OBEY = False

#5、是否支持cookie，cookiejar进行操作cookie，默认开启
#COOKIES_ENABLED = False

#6、Telnet用于查看当前爬虫的信息，操作爬虫等...使用telnet ip port ，然后通过命令操作
#TELNETCONSOLE_ENABLED = False
#TELNETCONSOLE_HOST = '127.0.0.1'
#TELNETCONSOLE_PORT = [6023,]

#7、Scrapy发送HTTP请求默认使用的请求头
#DEFAULT_REQUEST_HEADERS = {
#   'Accept': 'text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8',
#   'Accept-Language': 'en',
#}



#===>第二部分：并发与延迟<===
#1、下载器总共最大处理的并发请求数,默认值16
#CONCURRENT_REQUESTS = 32

#2、每个域名能够被执行的最大并发请求数目，默认值8
#CONCURRENT_REQUESTS_PER_DOMAIN = 16

#3、能够被单个IP处理的并发请求数，默认值0，代表无限制，需要注意两点
#I、如果不为零，那CONCURRENT_REQUESTS_PER_DOMAIN将被忽略，即并发数的限制是按照每个IP来计算，而不是每个域名
#II、该设置也影响DOWNLOAD_DELAY，如果该值不为零，那么DOWNLOAD_DELAY下载延迟是限制每个IP而不是每个域
#CONCURRENT_REQUESTS_PER_IP = 16

#4、如果没有开启智能限速，这个值就代表一个规定死的值，代表对同一网址延迟请求的秒数
#DOWNLOAD_DELAY = 3


#===>第三部分：智能限速/自动节流：AutoThrottle extension<===
#一：介绍
from scrapy.contrib.throttle import AutoThrottle #http://scrapy.readthedocs.io/en/latest/topics/autothrottle.html#topics-autothrottle
设置目标：
1、比使用默认的下载延迟对站点更好
2、自动调整scrapy到最佳的爬取速度，所以用户无需自己调整下载延迟到最佳状态。用户只需要定义允许最大并发的请求，剩下的事情由该扩展组件自动完成


#二：如何实现？
在Scrapy中，下载延迟是通过计算建立TCP连接到接收到HTTP包头(header)之间的时间来测量的。
注意，由于Scrapy可能在忙着处理spider的回调函数或者无法下载，因此在合作的多任务环境下准确测量这些延迟是十分苦难的。 不过，这些延迟仍然是对Scrapy(甚至是服务器)繁忙程度的合理测量，而这扩展就是以此为前提进行编写的。


#三：限速算法
自动限速算法基于以下规则调整下载延迟
#1、spiders开始时的下载延迟是基于AUTOTHROTTLE_START_DELAY的值
#2、当收到一个response，对目标站点的下载延迟=收到响应的延迟时间/AUTOTHROTTLE_TARGET_CONCURRENCY
#3、下一次请求的下载延迟就被设置成：对目标站点下载延迟时间和过去的下载延迟时间的平均值
#4、没有达到200个response则不允许降低延迟
#5、下载延迟不能变的比DOWNLOAD_DELAY更低或者比AUTOTHROTTLE_MAX_DELAY更高

#四：配置使用
#开启True，默认False
AUTOTHROTTLE_ENABLED = True
#起始的延迟
AUTOTHROTTLE_START_DELAY = 5
#最小延迟
DOWNLOAD_DELAY = 3
#最大延迟
AUTOTHROTTLE_MAX_DELAY = 10
#每秒并发请求数的平均值，不能高于 CONCURRENT_REQUESTS_PER_DOMAIN或CONCURRENT_REQUESTS_PER_IP，调高了则吞吐量增大强奸目标站点，调低了则对目标站点更加”礼貌“
#每个特定的时间点，scrapy并发请求的数目都可能高于或低于该值，这是爬虫视图达到的建议值而不是硬限制
AUTOTHROTTLE_TARGET_CONCURRENCY = 16.0
#调试
AUTOTHROTTLE_DEBUG = True
CONCURRENT_REQUESTS_PER_DOMAIN = 16
CONCURRENT_REQUESTS_PER_IP = 16



#===>第四部分：爬取深度与爬取方式<===
#1、爬虫允许的最大深度，可以通过meta查看当前深度；0表示无深度
# DEPTH_LIMIT = 3

#2、爬取时，0表示深度优先Lifo(默认)；1表示广度优先FiFo

# 后进先出，深度优先
# DEPTH_PRIORITY = 0
# SCHEDULER_DISK_QUEUE = 'scrapy.squeue.PickleLifoDiskQueue'
# SCHEDULER_MEMORY_QUEUE = 'scrapy.squeue.LifoMemoryQueue'
# 先进先出，广度优先

# DEPTH_PRIORITY = 1
# SCHEDULER_DISK_QUEUE = 'scrapy.squeue.PickleFifoDiskQueue'
# SCHEDULER_MEMORY_QUEUE = 'scrapy.squeue.FifoMemoryQueue'


#3、调度器队列
# SCHEDULER = 'scrapy.core.scheduler.Scheduler'
# from scrapy.core.scheduler import Scheduler

#4、访问URL去重
# DUPEFILTER_CLASS = 'step8_king.duplication.RepeatUrl'



#===>第五部分：中间件、Pipelines、扩展<===
#1、Enable or disable spider middlewares
# See http://scrapy.readthedocs.org/en/latest/topics/spider-middleware.html
#SPIDER_MIDDLEWARES = {
#    'Amazon.middlewares.AmazonSpiderMiddleware': 543,
#}

#2、Enable or disable downloader middlewares
# See http://scrapy.readthedocs.org/en/latest/topics/downloader-middleware.html
DOWNLOADER_MIDDLEWARES = {
   # 'Amazon.middlewares.DownMiddleware1': 543,
}

#3、Enable or disable extensions
# See http://scrapy.readthedocs.org/en/latest/topics/extensions.html
#EXTENSIONS = {
#    'scrapy.extensions.telnet.TelnetConsole': None,
#}

#4、Configure item pipelines
# See http://scrapy.readthedocs.org/en/latest/topics/item-pipeline.html
ITEM_PIPELINES = {
   # 'Amazon.pipelines.CustomPipeline': 200,
}



#===>第六部分：缓存<===
"""
1. 启用缓存
    目的用于将已经发送的请求或相应缓存下来，以便以后使用
  
    from scrapy.downloadermiddlewares.httpcache import HttpCacheMiddleware
    from scrapy.extensions.httpcache import DummyPolicy
    from scrapy.extensions.httpcache import FilesystemCacheStorage
"""
# 是否启用缓存策略
# HTTPCACHE_ENABLED = True

# 缓存策略：所有请求均缓存，下次在请求直接访问原来的缓存即可
# HTTPCACHE_POLICY = "scrapy.extensions.httpcache.DummyPolicy"
# 缓存策略：根据Http响应头：Cache-Control、Last-Modified 等进行缓存的策略
# HTTPCACHE_POLICY = "scrapy.extensions.httpcache.RFC2616Policy"

# 缓存超时时间
# HTTPCACHE_EXPIRATION_SECS = 0

# 缓存保存路径
# HTTPCACHE_DIR = 'httpcache'

# 缓存忽略的Http状态码
# HTTPCACHE_IGNORE_HTTP_CODES = []

# 缓存存储的插件
# HTTPCACHE_STORAGE = 'scrapy.extensions.httpcache.FilesystemCacheStorage'


#===>第七部分：线程池<===
REACTOR_THREADPOOL_MAXSIZE = 10

#Default: 10
#scrapy基于twisted异步IO框架，downloader是多线程的，线程数是Twisted线程池的默认大小(The maximum limit for Twisted Reactor thread pool size.)

#关于twisted线程池：
http://twistedmatrix.com/documents/10.1.0/core/howto/threading.html

#线程池实现：twisted.python.threadpool.ThreadPool
twisted调整线程池大小：
from twisted.internet import reactor
reactor.suggestThreadPoolSize(30)

#scrapy相关源码：
D:\python3.6\Lib\site-packages\scrapy\crawler.py

#补充：
windows下查看进程内线程数的工具：
    https://docs.microsoft.com/zh-cn/sysinternals/downloads/pslist
    或
    https://pan.baidu.com/s/1jJ0pMaM
  
    命令为：
    pslist |findstr python

linux下：top -p 进程id


#===>第八部分：其他默认配置参考<===
D:\python3.6\Lib\site-packages\scrapy\settings\default_settings.py
```

# 十三、常见问题

**默认只能在cmd中执行爬虫，如果想在pycharm中执行需要做**

```python
#在项目目录下新建：entrypoint.py
from scrapy.cmdline import execute
execute(['scrapy', 'crawl', 'xiaohua',"--nolog"])
```

**windows会乱码,加第二行代码**

```python
import sys,io
sys.stdout=io.TextIOWrapper(sys.stdout.buffer,encoding='gb18030')
import scrapy


class ChoutiSpider(scrapy.Spider):
    name = 'chouti'
    allowed_domains = ['chouti.com']
    start_urls = ['http://dig.chouti.com/']

    def parse(self,response):
        print(response.text)