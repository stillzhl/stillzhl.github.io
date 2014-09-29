---
layout: post
title: 技术栈更新
---

来qunar.com工作有一个月了，总结一下这段时间见到的新技术，为以后更新自己的技术stack做一个指引。
## 数据库
我所在的部门数据库用的是[PostgreSQL](http://www.postgresql.org/)，老大比较推崇pg。以前用的都是mysql，至于这两个哪个好也不必太在意，因为好多比较的点可能都是在高并发的场景下才会涉及到。初学阶段应该更关心如何更快的上手，然后做个自己的项目，真正用上之后遇到困难再去补，这样的效率会比较高。
就目前我的了解：
    
* pg 未实现 `insert into on duplicate key update` 的语义，这点需要注意
* 

pg 的一些资源：

* [PostgreSQL新手入门](http://www.ruanyifeng.com/blog/2013/12/getting_started_with_postgresql.html)
* [官方文档v9.3](http://www.postgresql.org/docs/9.3/static/index.html)
* 

## 语言
我目前只是比较熟悉python，所以想再学习一些其他不同类型的语言，来丰富我的Toolbox。下面是目前进入视野的一些语言：

* Ruby: 主要看中其*面向对象* 和 rails web框架。
* Elixir：较新，目前刚刚发布1.0版本，基于erlang vm、函数式编程、支持分布式等特性。 
* C：一直想好好学的，编译型语言。做网络编程必须的语言，为以后的职业目标“高并发网络编程”做准备。
    
[Dave Thomas](http://en.wikipedia.org/wiki/Dave_Thomas_(programmer\))相当推崇[Elixir](http://elixir-lang.org/)这一语言。

最近还发现了[MacRuby](http://macruby.org/)，之前一直比较热衷于为os X开发软件，但是又不想学Object-C，现在看来学好ruby之后也可以做了。

## 工具
zsh的强大自不必多说，Oh-my-zsh的名气也非常高，但是最近发现了他的替代品[Prezto](https://github.com/sorin-ionescu/prezto)。Prezto的宣言是：
>   Instantly Awesome Zsh

用起来确实比Oh-my-zsh快很多，而且也有好多新的特性。
关于开发工具可以看看[这篇blog](http://joshsymonds.com/blog/2014/01/22/supercharge-your-rails-development-environment/)。

最近发现好多用mac的同学并没有真正使其发挥应有的作用，所以这里分享给大家一片帖子，仔细设置一下你心爱的mac吧：
	
[Mac OS X Setup Guide](http://www.sourabhbajaj.com/mac-setup/)
