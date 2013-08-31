---
layout: post
title: "点滴知识积累"
date: 2012-12-13 02:09
comments: true
categories: ruby 
---
1) 在程序中经常看见“**\_\_FILE\_\_**”（注意是左边两个下划线，右边两下划线）,这代
表“**\_\_FILE\_\_**”所在文件的文件名。如果“**\_\_FILE\_\_**”是在require或者load的文件中，则是代
表被包含文件的绝对路径和文件名。
   
a.rb文件:
	{% codeblock lang:ruby %}
	p __FILE__
	{% endcodeblock %} 

b.rb文件
	{% codeblock lang:ruby %}
	require "./a"
	p __FILE__
	{% endcodeblock %} 
执行ruby b.rb,输出：
	/home/suzhen/workspace/a.rb
	b.rb 




