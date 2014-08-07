---
layout: post
title: Update Spider Arch Final
---

# 分布式爬虫系统文档

## 基础架构

* 系统流程图

	![](http://stillzhl.github.io/image/distribute-spider-flowchart.png)

* 网络拓扑图
    
    ![](http://stillzhl.github.io/image/distribute-spider-topo.png)
    
* 资源配置
	
	* ListSpiderBroker Redis: 
		
		redis://192.168.1.162:6978/0
		
	* GeneralSpiderBroker Redis, 缺货处理/实时抓取/新品抓取:
	
		redis://192.168.1.165:6978/0
		
	* URL Frontier/DUE等 Redis
	
		192.168.1.174:6978
		
	* CatPath Redis
	
		192.168.1.174:6979
		
	* 价格库存Mysql  

		192.168.1.178:3306 PriceStock T_PriceStock
		
	* URL Mysql
	
		192.168.1.168:3306 PriceStock T_Product_urls
		
	* 商品信息数据库
	
		192.168.1.175:3306 review T_Product_New
		
	* 价格库存 ttserver
		
		主库：192.168.1.171:11261
		
		从库：192.168.1.166:11261
		
	* 商品标题 ttserver
	
		192.168.1.178:11903
		
	
	


## DevOps

* RabbitMQ
	
	* rabbitmq需要设置启动时需要申请文件描述符的最大使用量，过少导致hit，server挂起；rabbitmq会对能够使用的虚拟内存空间作默认设置，需要根据实际系统调整，否则也会导致hit，server挂起
	
			  # login as root
			  # install rabbitmq from deb file
			$ dpkg -i rabbitmq-server_3.2.3-1_all.deb
		
			$ ulimit -n 20480
              # dump 的两个设置对应 Mnesia is overloaded 这个错误
			$ rabbitmq-server -mnesia dump_log_write_threshold 50000 -mnesia dc_dump_limit 40 -detached
			$ rabbitmqctl set_vm_memory_high_watermark 0.8
	* rabbitmq用户设置
	
			$ rabbitmqctl add_user username password
			$ rabbitmqctl set_permissions username ".*" ".*" ".*"	
	* rabbitmq的相关文件默认位置
		* log文件：

				
				/var/log/rabbitmq/rabbitmq@*.log
		* lib文件：
				
				/var/lib/rabbitmq/mnesia
		* cookie文件（用于cluster）
			
				/var/lib/rabbitmq/.erlang.cookie or ~/.erlang.cookie
	* rabbitmq High Available
		
		* 设置cluster时要求各台机器上的cookie文件相同才能通信
		* 设置hosts文件来实现机器识别，如在163上作164的备机，需要在163的hosts文件中添加以下配置项，spider@164为164的hostname：
			
				192.168.1.164	spider@164
		* 设置[rabbitmq cluster](https://www.rabbitmq.com/clustering.html)
			
				$ rabbitmqctl cluster_status
				$ rabbitmqctl stop_app
				$ rabbitmqctl join_cluster --ram rabbit@rabbit1
				$ rabbitmqctl start_app	
				
	* **目前已经弃用RabbitMQ，改成使用Redis来作为Celery的broker。RabbitMQ坑太多，文档少，比如rabbitmq要将queue的镜像刷到硬盘上，这会造成大量的磁盘写入，整个系统的吞吐量因此而大大下降。**	


* Redis 

	* @162 or @165: /d1/zhanghaili/spider_redis/

	* redis配置文件中的maxclients的默认值为10000，可根据系统需要自行增加，比如现在为100000
	
	* redis对虚拟内存的资源利用是受到linux系统限制的，需要调整内核参数来获取更大的内存空间：
		
		
			$ sudo sysctl vm.overcommit_memory=1
* Linux内核参数优化

 	如果用过的socket连接不作回收，会耗尽系统的资源，导致连接ttserver等不成功
		
		sudo sysctl -w net.ipv4.tcp_tw_recycle=1
		sudo sysctl -w net.ipv4.tcp_timestamps=1
		
* 系统内软件包版本 need fix, 若想升级软件包版本，可能需要修改代码兼容新的软件包，必须测试后再上线

* 由于各个worker随着运行时间的增长，会加大内存占用，目前采取的办法是crontab定时重启各机器上的各worker（通过fabric脚本）。
	
	crontab @192.168.1.172
	
		10 */2 * * * cd /home/zhanghaili && fab sv_worker:action=restart,name_prefix=general-default
		10 */2 * * * cd /home/zhanghaili && fab sv_worker:action=restart,name_prefix=list-default
		20 */1 * * * cd /home/zhanghaili && fab sv_worker:action=restart,name_prefix=list-dl
		30 */1 * * * cd /home/zhanghaili && fab sv_worker:action=restart,name_prefix=list-prs
		40 */1 * * * cd /home/zhanghaili && fab sv_worker:action=restart,name_prefix=list-pi
		50 */1 * * * cd /home/zhanghaili && fab sv_worker:action=restart,name_prefix=os-pi
		0 */1 * * * cd /home/zhanghaili && fab sv_worker:action=restart,name_prefix=os-dl
		5 */1 * * * cd /home/zhanghaili && fab sv_worker:action=restart,name_prefix=os-prs
		8 */1 * * * cd /home/zhanghaili && fab sv_worker:action=restart,name_prefix=dp-dl
		14 */1 * * * cd /home/zhanghaili && fab sv_worker:action=restart,name_prefix=dp-pi
		35 */1 * * * cd /home/zhanghaili && fab sv_worker:action=restart,name_prefix=dp-prs

* 下载worker（downloader）需采用eventlet的并发模式启动，其他worker则采用多进程的并发模式启动。若worker存在阻塞（如写文件或写数据库），则使用eventlet模式会挂起所有worker，需注意! 目前在升级了软件包之后，使用eventlet会报错，基本情况是下载的时候读取chunk package 的时候字节长度和预期的不一样，所以**改成使用多进程模型**。
