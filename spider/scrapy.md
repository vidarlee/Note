# scrapy使用笔记

**scrapy是目前比较流行的爬虫框架，另外一个比较有名的爬虫框架是BeautifulSoup，虽然没有用过BeautifulSoup，但scrapy的优点是速度快，因此先着手学习scrapy。（之前都是用requests来爬网站）**

## scrapy安装
_scrapy的安装步骤这里就先省略，可以查看官网_

Link: [scrapy官网](https://doc.scrapy.org/en/latest/intro/install.html)


## 创建scrapy工程
**既然是框架，那么一般都以创建project为起点，来开始工作的。今天用爬取历史温度为任务开展学习**

Link: [目标网址](http://weather.uwyo.edu/upperair/np.html)

```
$ scrapy startproject temp
:0: UserWarning: You do not have a working installation of the service_identity module: 'No module named 'service_identity''.  Please install it from <https://pypi.python.org/pypi/service_identity> and make sure all of its dependencies are satisfied.  Without the service_identity module, Twisted can perform only rudimentary TLS client hostname verification.  Many valid certificate/hostname mappings may be rejected.
New Scrapy project 'temp', using template directory '/usr/local/lib/python3.4/dist-packages/scrapy/templates/project', created in:
    /home/vagrant/scrapy/temp

You can start your first spider with:
    cd temp
    scrapy genspider example example.com
```

## 创建爬虫类
**具体怎么执行爬虫逻辑，需要在spiders目录下定义相关的类来实现**
```
$ pwd
/home/vagrant/scrapy/temp/temp/spiders
$ ls
__init__.py  __pycache__
$ vim temp_spider.py
```

**具体代码如下：**

```
import calendar
import re
import scrapy
from datetime import datetime
from pandas import DataFrame

class CityTempItem(scrapy.Item):
    region = scrapy.Field()
    city = scrapy.Field()
    stnm = scrapy.Field()
    date = scrapy.Field()
    temp_avg = scrapy.Field()
    temp_max = scrapy.Field()
    temp_min = scrapy.Field()
    website = scrapy.Field()

class TempSpider(scrapy.Spider):
    name = "temp"

    def __init__(self):
        self.region = 'np'
        self.city = 'Ostrov Kotelnyj'
        self.stnm = 21432
        self.start_year = 1973
        self.end_year = 1974
        self.stnm = 21432
        self.website = 'uwyo.edu'
        self.proxy = 'http_proxy=http://LiYucai:2210060032@rep.proxy.nic.fujitsu.com:8080/auth.pac'
        super().__init__()

    def start_requests(self):
        base_url = 'http://weather.uwyo.edu/cgi-bin/sounding?TYPE=TEXT%3ALIST&'
        params = 'region=%s&YEAR=%s&MONTH=%s&FROM=0100&TO=%s12&STNM=%s'
        urls = []
        start_year = self.start_year
        while start_year < self.end_year:
            for i in range(1, 13):
                month = i
                last_day = calendar.monthrange(start_year, month)[1]
                url = base_url + params % (self.region, start_year, month, last_day, self.stnm)
                urls.append(url)
            start_year += 1

        for url in urls:
            yield scrapy.Request(url=url, callback=self.parse)#, meta={'proxy':self.proxy})


    def parse(self, response):
        temp = response.css('PRE::text').extract()
        date = response.css('H2::text').extract()
        temp_f = list(map(lambda t:t.TEMP.max(),list(map(lambda e:DataFrame.from_records(e,columns=temp[0].split('\n')[2].split()), list(map(lambda zz:list(map(lambda x:list(map(lambda e:float(e) if e != '' else e, re.split(r'\s{1,7}',x.strip(),11))),zz.split('\n')[5:11])), temp[0::2]))))))
        date_f = list(map(lambda x:x[-15:], date))
        temp_c = tuple(zip(date_f, temp_f))
        result = dict(temp_c)
        min_max_temp = dict()
        for key, value in result.items():
            temp_type, date = re.split(r'\s', key, 1)

            if date not in min_max_temp:
                min_max_temp[date] = [0,0]

            if temp_type == '00Z':
                min_max_temp[date][0] = value
            elif temp_type == '12Z':
                min_max_temp[date][1] = value

        for key, value in min_max_temp.items():
            item = CityTempItem()
            item['website'] = self.website
            item['region'] = self.region
            item['city'] = self.city
            item['stnm'] = self.stnm
            item['temp_min'] = value[0]
            item['temp_max'] = value[1]
            item['date'] = datetime.strptime(key, '%d %b %Y').strftime('%Y-%m-%d')
            yield item
```

### 设置代理
**如果连接互联网需要设置代理，那么在linux下面设置好http或者https代理的环境变量即可**
```
export http_proxy='xxxxxxxxx'
export https_proxy='xxxxxxxx'
```

## 开始爬数据

```
$ scrapy crawl temp

...
2018-08-20 08:47:53 [scrapy.core.scraper] DEBUG: Scraped from <200 http://weather.uwyo.edu/cgi-bin/sounding?TYPE=TEXT%3ALIST&region=np&YEAR=1973&MONTH=1&FROM=0100&TO=3112&STNM=21432>
{'city': 'Ostrov Kotelnyj',
 'date': '1973-01-01',
 'region': 'np',
 'stnm': 21432,
 'temp_max': -20.9,
 'temp_min': 21.8,
 'website': 'uwyo.edu'}
2018-08-20 08:47:53 [scrapy.core.engine] DEBUG: Crawled (200) <GET http://weather.uwyo.edu/cgi-bin/sounding?TYPE=TEXT%3ALIST&region=np&YEAR=1973&MONTH=11&FROM=0100&TO=3012&STNM=21432> (referer: None)
...
```

## 数据保存
**可以保存成为CSV，json，xml等，本文保存为CSV。**
**有两种方式可以进行保存：**
- 参数-o -t指定输出
- 通过Pipeline自定义输出
***

- 参数-o -t指定输出

  ```
  $ scrapy crawl temp -o cityname.csv -t csv
  $ ls
     cityname.csv  scrapy.cfg  temp
     $ tail -20 cityname.csv
   0.7,Ostrov Kotelnyj,1.8,,uwyo.edu,1973-08-08,np,21432
   3.8,Ostrov Kotelnyj,0,,uwyo.edu,1973-08-07,np,21432
   4.6,Ostrov Kotelnyj,2.0,,uwyo.edu,1973-08-28,np,21432
   67.8,Ostrov Kotelnyj,4.6,,uwyo.edu,1973-08-16,np,21432
   3.2,Ostrov Kotelnyj,3.0,,uwyo.edu,1973-08-11,np,21432
   6.0,Ostrov Kotelnyj,3.8,,uwyo.edu,1973-08-15,np,21432
   1.4,Ostrov Kotelnyj,1.2,,uwyo.edu,1973-08-03,np,21432
   0,Ostrov Kotelnyj,3.8,,uwyo.edu,1973-08-25,np,21432
   3.0,Ostrov Kotelnyj,0,,uwyo.edu,1973-08-05,np,21432
   4.2,Ostrov Kotelnyj,2.4,,uwyo.edu,1973-08-06,np,21432
   2.2,Ostrov Kotelnyj,0.4,,uwyo.edu,1973-08-22,np,21432
   1.0,Ostrov Kotelnyj,3.8,,uwyo.edu,1973-08-29,np,21432
   11.0,Ostrov Kotelnyj,5.2,,uwyo.edu,1973-08-26,np,21432
   4.2,Ostrov Kotelnyj,3.0,,uwyo.edu,1973-08-24,np,21432
   0.5,Ostrov Kotelnyj,1.6,,uwyo.edu,1973-08-13,np,21432
   0.0,Ostrov Kotelnyj,0.9,,uwyo.edu,1973-08-31,np,21432
   2.8,Ostrov Kotelnyj,0.9,,uwyo.edu,1973-08-10,np,21432
   6.2,Ostrov Kotelnyj,3.2,,uwyo.edu,1973-08-17,np,21432
   0.4,Ostrov Kotelnyj,0.2,,uwyo.edu,1973-08-02,np,21432
   1.2,Ostrov Kotelnyj,2.0,,uwyo.edu,1973-08-23,np,21432
  ```

**多次运行，可以看出，这样输出保存的CSV文件，里面字段的顺序都是随机的，如果需要按照自己指定的顺序保存到CSV文件，则需要采用下面Pipeline的方式**

- 通过Pipeline自定义输出

**Pipeline的方式其实就是利用Item Exporter来保存Item到指定的文件或者数据库。scrapy提供了好几种Item Exporter：BaseItemExporter，XMLItemExporter， CsvItemExporter等等。要采用Pipeline的方式保存，则需要按照以下几步来进行。参考官网**
Link: [Item Exporters](https://doc.scrapy.org/en/latest/topics/exporters.html#using-item-exporters)

### 在pipelines.py文件中定义相关的类
**注意，类中可以包含以下函数**
 - open_spider 爬虫开始时执行的操作在这个函数中定义
 - close_spider 爬虫结束时需要执行的操作在这个函数中定义
 - process_item 对于每个item需要执行的操作在这个函数中定义

### Item Exporter必须要调用以下3个API
 - start_exporting() 开始执行导出，一般是创建导出文件后立即执行
 - export_item() 导出需要导出的Item，执行导出的具体操作
 - finish_exporting() 结束导出，此外还需要执行关闭文件.file.close()等。

### 使自定义的Pipeline有效
**将定义的class追加到settings.py中的ITEM_PIPELINES设定中**
```
ITEM_PIPELINES = {
'tutorial.pipelines.TutorialPipeline': 300,
'tutorial.pipelines.TestPipeline': 800,
'tutorial.testpipelines.TestPipeline': 100
}
```

**数字可以指定1-1000，数字越小越优先执行**

***
- Pipeline例

```
from scrapy.exporters import CsvItemExporter

class TutorialPipeline(object):

    def open_spider(self, spider):
        self.list_items = {}
        self.city_files = {}

    def close_spider(self, spider):
        for city, exporter in self.city_files.items():
            items =  self.list_items[city]
            date_items = dict()

            for item in items:
                date_items[item['date']] = item

            for day in sorted(date_items):
                exporter.export_item(date_items[day])

            exporter.finish_exporting()
            exporter.file.close()

    def _exporter_for_item(self, item):
        city = item['city']
        if city not in self.city_files:
            f = open('{}.csv'.format(city), 'wb')
            exporter = CsvItemExporter(f, fields_to_export=["website", "region", "city", "stnm", "date", "temp_min", "temp_max", "temp_avg"])
            exporter.start_exporting()
            self.city_files[city] = exporter

        if city not in self.list_items:
            self.list_items[city] = []
        self.list_items[city].append(item)

    def process_item(self, item, spider):
        self._exporter_for_item(item)
        return item
```

**注意：fields_to_export可以作为参数传递给CsvItemExporter，以定义字段的输出顺序**

### 数据库的

## Debug或者调试
- scrapy parse --spider=temp -c <function name> -d 2 <url>
- scrapy shell <url>
