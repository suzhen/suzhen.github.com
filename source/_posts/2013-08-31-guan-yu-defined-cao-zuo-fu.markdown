---
layout: post
title: "关于defined?操作符"
date: 2013-08-31 17:55
comments: true
categories: ruby 
---
defind?是一个和and、not一样的操作符。用来判断表达式是否存在，及是哪种类型的表达式。

{% codeblock lang:ruby %}
defined? yuchen         #=> nil
defined? true         #=> “true”
defined? $*           #=> "global-variable"
defined? Array        #=> "constant"
defined? Math::PI #=> "constant"
defined? num = 0     #=> "assignment"
defined? 100         #=> "expression"
defined? 100.times   #=> "method"
{% endcodeblock %}

除此之外还有一些用法：

1、可以来判断是否有block,作用同block_given? 
{% codeblock lang:ruby %}

    class Base
        def foo
           puts defined? yield
        end
    end
{% endcodeblock %}

 {% codeblock lang:ruby %}

a = Base.new
a.foo
a.foo {}
 {% endcodeblock %}

执行结果为：
 {% codeblock lang:ruby %}
 nil
 yield #如果有block输出"yield"
 {% endcodeblock %}

2、可以来判断方法里是否有super方法
{% codeblock lang:ruby %}

class Base
        def foo
        end
end
 
class Derived < Base
        def foo
           puts defined? super
        end
   
        def fun
           puts defined? super
        end
end
{% endcodeblock %}

 {% codeblock lang:ruby %}

obj = Derived.new
obj.foo
obj.fun
 {% endcodeblock %}

执行结果为：
 {% codeblock lang:ruby %}
 super
 nil
 {% endcodeblock %}



注意一点。如果defined?的参数是一段语句，那是不能被执行的。
{% codeblock lang:ruby %}

p defined?(def x; end)   # "expression"
x                        # error: undefined method or variable
{% endcodeblock %}

{% codeblock lang:ruby %}

p defined?(@x=1)         # "assignment"
p @x                     # nil
{% endcodeblock %}

