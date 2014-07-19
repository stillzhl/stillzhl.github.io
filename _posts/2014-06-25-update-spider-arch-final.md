---
layout: post
title: Update Spider Arch Final
---

# 爬虫升级记[最终总结]

## 基础架构

* 网络拓扑图
    
    ![](http://stillzhl.github.io/image/distribute-spider-topo.jpg)


## DevOps

* RabbitMQ
	
	* rabbitmq需要设置启动时需要申请文件描述符的最大使用量，过少导致hit，server挂起；rabbitmq会对能够使用的虚拟内存空间作默认设置，需要根据实际系统调整，否则也会导致hit，server挂起
	
			# login as root
			# install rabbitmq from deb file
			dpkg -i rabbitmq-server_3.2.3-1_all.deb
		
			ulimit -n 20480
			rabbitmq-server -detached
			rabbitmqctl set_vm_memory_high_watermark 0.8
	* rabbitmq用户设置
	
			rabbitmqctl add_user username password
			rabbitmqctl set_permissions username ".*" ".*" ".*"	
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
			
				rabbitmqctl cluster_status
				rabbitmqctl stop_app
				rabbitmqctl join_cluster --ram rabbit@rabbit1
				rabbitmqctl start_app			
				

* redis client max 
	
	redis配置文件中的默认值为10000，可根据系统需要自行增加
	
* sysctl tcp 连接回收，内核参数优化
	如果用过的socket连接不作回收，会耗尽系统的资源，导致连接ttserver等不成功
		
		sudo sysctl -w net.ipv4.tcp_tw_recycle=1
		sudo sysctl -w net.ipv4.tcp_timestamps=1
		
* 系统内软件包版本 need fix, 若想升级软件包版本，可能需要修改代码兼容新的软件包，必须测试后再上线

* 由于各个worker随着运行时间的增长，会加大内存占用，目前采取的办法是crontab定时重启各机器上的各worker（通过fabric脚本）。目前在172上执行crontab

* 下载worker（downloader）需采用eventlet的并发模式启动，其他worker则采用多进程的并发模式启动。若worker存在阻塞（如写文件或写数据库），则使用eventlet模式会挂起所有worker，需注意!
