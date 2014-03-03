---
layout: post
title: Headless Browser (Selenium+PhantomJS)
---

# 搭建  Headless Browser (Selenium+PhantomJS)
搭建Headless Brwoser，为Scrapy抓取页面服务，适用于分布式架构，在一台机器上部署之后，其他机器可通过remote方式使用。
Phantomjs和Selenium的结合由[Ghost Driver](https://github.com/detro/ghostdriver)提供，现在这个项目已经被include到Phantomjs的官方最新版本：
> THAT'S IT!! Because of latest stable GhostDriver being embedded in PhantomJS, you shouldn't need anything else to get started.

#### 安装

###### Selenium RC
    $ cd ~/tmp
    $ wget http://selenium.googlecode.com/files/selenium-server-standalone-2.34.0.jar

###### Selenium Python Client
selenium的[python doc](http://selenium.googlecode.com/git/docs/api/py/index.html)
    $ pip install selenium

###### PhantomJS
Phantomjs的最新版本托管在Google Code上，被墙，需用通过代理下载，[ProxyChains设置](http://jianshu.io/p/PWWfsg)

    $ proxychians wget 'https://phantomjs.googlecode.com/files/phantomjs-1.9.1-linux-x86_64.tar.bz2'
    $ tar jxvf phantomjs-1.9.1-linux-x86_64.tar.bz2
###### Java
selenium运行需要java环境

    $ sudo apt-get install openjdk-7-jre

#### 部署
selenium和phantomjs在测试阶段可在前台运行观察日志
###### Selenium RC
selenium在默认的4444端口启动，很矬的一个端口...

    $ cd /path/to/selenium_jar/
    $ nohup java -jar selenium-server-standalone-2.34.0.jar -role hub > /dev/null &

###### PhantomJS
    $ cd /path/to/phantomjs
    $ nohup ./bin/phantomjs --webdriver=8080 --webdriver-selenium-grid-hub=http://127.0.0.1:4444 > /dev/null &

#### Python Example
用来测试的url为当当网手机版的掌上绝杀页面，绝杀倒计时为JS生成

    $ cat python_example.py
    $ #!/usr/bin/env python
      #coding: utf-8

      import sys

      from selenium import webdriver


      def main(url):
          caps = {
              'takeScreenshot': False,
              'javascriptEnabled': True,
          }
          phantom_link = 'http://127.0.0.1:8080/wd/hub'

          driver = webdriver.Remote(
              command_executor=phantom_link, 
              desired_capabilities=caps
              )

          driver.get(url)

          print driver.title
          print driver.find_element_by_xpath('//ul[@id="clock_0"]').text


      if __name__ == "__main__":
          url = sys.argv[1]
          main(url)

    $ python python_example.py 'http://m.dangdang.com/touch/topics.php?page_id=23387'
