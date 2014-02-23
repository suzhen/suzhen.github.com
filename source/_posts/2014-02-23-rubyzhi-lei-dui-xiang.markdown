---
layout: post
title: "Ruby之类对象"
date: 2014-02-23 11:26
comments: true
categories: 
---
先熟悉一下类对象。

  
  它有点像是作用在整个类继承层次的 全局变量, 它的行为像是没有命名空间的全局常量, 在 一个类及其所有子类中 总是唯一的. 既然他属于 类和它的子孙 公用的那一部分, 那也必然应该属于类了, @@类变量是私有的, 只能在类定义内部隐式的访问它.

  @@对象不仅被类族中类方法访问，也可以被类族中的实例对象访问到。

{% codeblock lang:ruby %} 
  class Dummy
    @@name = "suzhen"

    def self.name
      @@name
    end
    def name
      @@name
    end
  end

  p Dummy.name    # suzhen
  p Dummy.new.name  #suzhen
{% endcodeblock %}

  active_support中对这种类对象有很好的封装。看下面代码：

{% codeblock lang:ruby %} 
  require "active_support/all"

  class Dummy
    mattr_accessor :name
  end

  Dummy.name = "suzhen"

  p Dummy.name #suzhen
  p Dummy.new.name  #suzhen

{% endcodeblock %}

  mattr_accessor 就是帮我们创建了一组类方法和一组实例方法

  有的时候，我们只想给出类方法的访问，而不想给出实例方法的访问。

  mattr_accessor :name,:instance_accessor=>false

  至于mattr_accessor 是如何实现的，可以看rails的代码。

  https://github.com/rails/rails/blob/master/activesupport/lib/active_support/core_ext/module/attribute_accessors.rb

  mattr_accessor 与 cattr_accessor 是一样的。


  我们看下面的例子：

{% codeblock lang:ruby %}   
  module SomeModule
    @@logger = "suzhen"

  end

  class Dummy

    include SomeModule

    def self.say
      @@logger
    end
    def say
      @@logger
    end
  end

  p Dummy.say   ＃suzhen
  p Dummy.new.say #suzhen
{% endcodeblock %}

  当一个class引入一个module就是这个class就继承了这个module,那么module中定义的类变量，就可以在这上类家庭中使用了。

{% codeblock lang:ruby %} 
  module SomeModule
    mattr_accessor :logger
  end

  class Dummy

    include SomeModule

  end

  p Dummy.new.logger  ＃suzhen

  p Dummy.logger  #error
{% endcodeblock %}

  但是上面这个例子中，mattr_accessor 把类方法 self.logger 和self.logger=定义在了module中，include自然不会引用这种类方法。但实例方法是可以用的。

{% codeblock lang:ruby %} 
  module SomeMoudle
    def self.included(base)
      base.class_eval do
        mattr_accessor :logger
        self.logger = "suzhen"   #初始化的时候一定把self加上。显示的调用。
      end
    end
  end
{% endcodeblock %}

  用以上这种方法自然可以解决这个问题。

