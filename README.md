# Scrapy_Selenium_PhantomJS
基于scrapy静态网页爬取，结合Selenium和PhantomJS实现简单的自动加载js的动态页面
基于scrapy静态网页爬取，结合Selenium和PhantomJS实现简单的自动加载js的动态页面

1、	利用PhantomJS来获取页面初始化进行js自动加载的页面
利用PhantomJS(PhantomJS就是一个没有界面的浏览器，提供了JavaScript
接口,利用执行js来达到浏览器的效果),编写js代码用来输出访问某个具体网页返回的内容。
  ![Alt Text](
     https://github.com/appliance/Scrapy_Selenium_PhantomJS/blob/master/1.png
    )
	 （注意：必须安装PhantomJS并配置好环境变量）
打开cmd进入目标js目录中，输入命令：phantomjs ./test.js http://baidu.com
就会打印得到的html代码。
在此处我试过直接需要进行js加载的动态页面（除了那些需要进行某个操作出发的js除外，自动加载的js是可以实现的，如果需要进行某个动作触发js，就需要研究Selenium这个自动化测试工具了）

2、	Selenium自动化测试工具
Selenium需要进行pip安装，这里主要用到了它的Webdriver操作浏览器。
Selenium操作无界面浏览器PhantomJS
	  ![Alt Text](
     https://github.com/appliance/Scrapy_Selenium_PhantomJS/blob/master/2.png
    )
此处的driver.get()方法会等待页面加载完成后才返回，也就是说js会加载完毕，有很多ajax的情况除外。

3、	研究scrapy的下载器中间件
下载器中间件在下载器和Scrapy引擎之间，每一个request和response都会通过中间件进行处理。在中间件中，对request进行处理的函数是process_request(request, spider)
（概念：process_request() 必须返回其中之一: 返回 None 、返回一个 Response 对象、返回一个 Request 对象或raise IgnoreRequest 。如果其返回 None ，Scrapy将继续处理该request，执行其他的中间件的相应方法，直到合适的下载器处理函数(download handler)被调用， 该request被执行(其response被下载)。如果其返回 Response 对象，Scrapy将不会调用 任何 其他的 process_request() 或 process_exception() 方法，或相应地下载函数； 其将返回该response。 已安装的中间件的 process_response() 方法则会在每个response返回时被调用。如果其返回 Request 对象，Scrapy则停止调用 process_request方法并重新调度返回的request。当新返回的request被执行后， 相应地中间件链将会根据下载的response被调用。）
动态爬虫需要中间件处理request，将加载过特定js的页面作为response返回给spider。
简单说这三者的关系就是：Scrapy通过Selenium使用PhantomJS，爬取加载过JS的页面。

4、	理念理论熟悉后的具体操作
我们要爬取的页面是http://china.nba.com/teamindex/ 中国nba篮球网。（若将浏览
器禁止js运行，此网站是显示不出内容的，说明此网站是需要js自动加载的动态页面，直接调用scrapy去爬取，返回的都是空）
基于scrapy静态网页爬取的知识，结合Selenium和PhantomJS实现简单的自动加载js
的动态页面
创建完scrapy项目后依旧需要设置items,piplines以及创建一个爬虫文件，关键点是
对爬虫文件、middlewares.py，以及settings.py文件的编写

爬虫文件rosterLinkSpd.py的内容
 	  ![Alt Text](
     https://github.com/appliance/Scrapy_Selenium_PhantomJS/blob/master/3.png
    )
区别于静态网页的爬取，在此爬虫文件的parse函数，（添加头from scrapy.http import Request）先创建一个新的request请求，结果设置返回给parse_post函数进行处理。并在新request中添加一个meta用于中简件做处理的标识，返回新的request请求
（middlewares.py）中间件的process_request函数接受到新的request请求进行处理。代码 
  ![Alt Text](
     https://github.com/appliance/Scrapy_Selenium_PhantomJS/blob/master/4.png
    )
此文件需要添加头from selenium import webdriver
from scrapy.http import HtmlResponse
中间件中利用Selenium调用PhantomJS来进行动态页面的请求。获取加载完的动态页面，再利用HtmlResponse将结果返回给爬虫文件，爬虫文件中的parse_post函数按照静态网页的套路直接利用xpath进行获取内容就可以了。

还有一个重点是settings.py的设置
User-agent  获取浏览器中的值进行替换
如：
USER_AGENT = 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/51.0.2704.79 Safari/537.36 Edge/14.14393'

Cookies_enabled 将cookie关闭以防止被禁止
COOKIES_ENABLED = False

将中间件打开
DOWNLOADER_MIDDLEWARES = {
   'nbapjt.middlewares.NbapjtDownloaderMiddleware': 543,
}

将piplines打开
ITEM_PIPELINES = {
   'nbapjt.pipelines.NbapjtPipeline': 300,
}

参考文件链接：https://jiayi.space/post/scrapy-phantomjs-seleniumdong-tai-pa-chong#fb_new_comment
