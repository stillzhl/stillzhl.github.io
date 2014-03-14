---
layout: post
title: How To Use Proxy in Python
---

## 在Python中使用代理的几种方式总结：

###### 代理服务：
    
    proxy = http://192.168.1.172:11190
    proxy_host = '192.168.1.172'
    proxy_port = 11190
    url = "http://www.python.org/index.html"


* With urllib:

		$ export http_proxy='http://192.168.1.172:11190'    # in shell
		import urllib
		f = urllib.urlopen(url)
    

* With httplib:
    
    
    	import httplib

    	conn = httplib.HTTPConnection(proxy_host, proxy_port)
    	conn.request("GET", url)
    	r = conn.getresponse()
    	print r.status, r.reason
    	print r.msg

    	html = ''
    	while True:
        	html = r.read(1024)
        	print html
        	if len(html) < 1024:
            	break
    	
* With socket:

    	import socket

    	soc = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    	soc.connect(proxy_host, proxy_port)
    	msg = 'GET {0} HTTP/1.1\r\nAccept: text/plain\r\n\r\n'.format(url)
    	html = ''
    	while True:
        	html = soc.recv(1024)
        	print html
        	if len(html) < 1024:
            	break
    	soc.close()

* With urllib2.ProxyHandler:
    
    	from urllib2 import ProxyHandler
    	from urllib2 import Request
    	from urllib2 import build_opener
    	from urllib2 import HTTPHandler

    	req = Request(url)
    	
    	# proxy handler
    	proxy = '{0}{1}'.format(proxy_host, proxy_port)
    	ph = Proxyhandler({'http': proxy})
    	opener = build_opener(ph, HTTPHandler).open

    	req = opener(url)
    	html = req.read()
    
* With Scrapy

可以使用scrapy自己的httpproxy middalware，并通过环境变量http_proxy设置代理；
若为单独请求设置代理则采用如下方式：
    
    req_meta['proxy'] = proxy
   	yield Request(url, meta=req_meta, callback='parse')
