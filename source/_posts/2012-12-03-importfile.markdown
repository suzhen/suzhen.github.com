---
layout: post
title: "require/load/autoload的区别和用法"
date: 2012-12-03 01:34
comments: true
categories: ruby 
---
首先我们建两个目录，一个命名rubylib，目录下有一个lib.rb文件，另一个目录命名rubyconfig,目录下有一个cfg.rb文件。

lib.rb
	{% codeblock lang:ruby %}
	puts "I'm lib"
	{% endcodeblock %}
cfg.rb
	{% codeblock lang:ruby %}
	puts "I'm config"
	{% endcodeblock %}
再建立一个testrequire.rb文件
	{% codeblock lang:ruby %}
	require "./rubylib/lib"
	puts "hello,world"
	require "./rubylib/lib" #遇到相同的文件，不再加载
	{% endcodeblock %}
	

输出结果
	I'm lib
	hello,world

再建立一个testload.rb文件
	{% codeblock lang:ruby %}
	load "./rubyconfig/cfg.rb"
	puts "hello,world"
	load "./rubyconfig/cfg.rb" #无论是否加载过，都会加载
	{% endcodeblock %}

输出结果
	I'm config
	hello,world
	I'm config

说明：require和load同样都是用来加载文件。require的加载路径可以不写.rb,但是load就一定要写全文件名。require是只加载一次，如果有相同的文件，已经加载，则不再重复加载，用于加载库文件;而load是每次都会加载，无论是否已经加载过，常用于加载配置文件。
**require和load都表现为立即加载文件。而autoload则是使用到某个类时，才会去加载必要的文件。**


建立一个mylib.rb的文件
	{% codeblock lang:ruby %}
	puts "I was loaded!"

	class MyLibrary
	end
	{% endcodeblock %}

使用require加载会立即加载
	irb(main):001:0> require 'mylibrary'
	I was loaded!
	=> true

使用autoload只会在这个类被需要的时候，才去加载
	irb(main):001:0> autoload :MyLibrary, 'mylibrary'
	=> nil
	irb(main):002:0> MyLibrary.new
	I was loaded!
	=> #<MyLibrary:0x0b1jef>

autoload不会被重复加载!

如果有一些类或者模块一定会被用上，则使用require，如果不一定会被使用，才使用autoload。	


