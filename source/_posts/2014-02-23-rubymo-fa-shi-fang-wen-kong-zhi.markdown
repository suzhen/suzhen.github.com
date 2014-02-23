---
layout: post
title: "Ruby魔法师(访问控制)"
date: 2014-02-23 11:20
comments: true
categories: 
---
一个class中的method有三种访问权限，public , protected, private
  
  
 {% codeblock lang:ruby %} 
  class Car
    def pub_method
      puts "Car running"
    end


    protected

    def pro_method
      puts "Car booting"
    end


    private

    def pri_method
      puts "engine start"
    end
  end


  car = Car.new

  car.pub_method #输出 Car running

  car.pro_method #报错 protected method不可以被调用

  car.pri_method # 报错 private method不可以调用
{% endcodeblock %}

  从以上例子可以看出 class 里的instance_method默认是public的,object的protected和private的method是不可以在外部直接调用的。只能在instance_method内部调用。

  这里特别要注意的是：

  在method调用 里有 显示调用 和 隐藏调用 两种方式，例如:

{% codeblock lang:ruby %}
  class Car
    def pub_method

      #self隐藏调用
      pro_method
      pri_method
      
      puts "Car running"

    end


    protected

    def pro_method
      puts "Car booting"
    end


    private

    def pri_method
      puts "engine start"
    end
  end

  car = Car.new

  car.pub_method 

  #输出 engine start
      Car booting
      Car running
{% endcodeblock %}

  !!! *** pro_method和pri_method都支持隐藏调，但是只有protected支持显示调用，private不支持显示调用。

{% codeblock lang:ruby %}
  class Car
    def pub_method

      #self显示调用
      self.pro_method
      self.pri_method
      
      puts "Car running"

    end


    protected

    def pro_method
      puts "Car booting"
    end


    private

    def pri_method
      puts "engine start"
    end
  end

  car = Car.new

  car.pub_method 

  self.pri_method #是会报错的，而self.pro_method是可以通过的。

{% endcodeblock %}

  显示或者隐藏调用的对象不仅仅是指self，还有同类别的对象（包括同类的对象和子类的对象）。

  例子如下：

{% codeblock lang:ruby %}
  class C
      def initialize(name)
         @name = name
      end

      def compare(c)
      #c是同类别的对象 （神奇的魔法）
          c.name == @name
      end

      protected
      def name
          @name
      end
  end

  class D < C
  end

  d1 = D.new("fankai")
  d2 = D.new("hello")

  puts d1.compare(d2)

{% endcodeblock %}

  这个例子，即保证不能让客户端读取到对象的名称，又能在类内部可以获取到实例变量。



  接着我们可以看一个ruby类的继承关系。

{% codeblock lang:ruby %}
  class A
    def pub_method
      method1
      method2
    end

    private

    def method1
      puts "hello"
    end

    protected

    def method2
      puts "world"
    end

  end

  class B < A
    def pub_method2
      method1
      method2
    end
  end

  class C < B
    def pub_method3
      method1
      method2
    end
  end

  A.new.pub_method # 输出 hello world

  B.new.pub_method2 # 输出 hello world

  C.new.pub_method3 # 输出 hello world

{% endcodeblock %}

  由些可见，ruby在继承关系中public 和 protected 和 private都是一样，全部可以被继承。


  通过我们会认为类的方法是没有private和protected之分的，我们来看以下的例子：

 {% codeblock lang:ruby %} 
  class Dummy
      private
      def self.say_hi
         puts "hi"
      end
  end

  Dummy.say_hi  #-> hi    说明private对类方法是没有的。

{% endcodeblock %}

  但是魔法师没有什么是做不到的，如果你想要，类方法有可以有这种调用限制。

{% codeblock lang:ruby %}
  class Dummy
          class << self
              protected
              def say_hello
                puts "hello"
              end
          end
      end

      Dummy.say_hello #in `<main>': protected method `say_hello' called for Dummy:Class (NoMethodError)
{% endcodeblock %}

  看了上面的例子你来会认为类的方法是没有private和protected之分的吗？？？

  我想说ruby,你他妈的太牛逼了！
