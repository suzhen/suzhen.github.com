---
layout: post
title: "RUBY魔法师（方法高级篇）"
date: 2014-02-23 11:37
comments: true
categories: 
---

  一、动态创建method和动态调用方法

  问题：什么是动态创建和动态调用方法？
  答：就是代码在运行过程时，才定义的方法。同样，只有在代码运行时，才能开始调用的方法，是动态调用方法，这是一组很有用的魔法。
  

  魔法师用define_method来动态创建实例方法：

{% codeblock lang:ruby %} 
  class Dummy
    #动态创建一个实例方法say_hi
    define_method :say_hi do | name |
      puts "HI,#{name}"
    end
    #动态创建一个类方法say_hello
    class << self
      define_method :say_hello do | name |
        puts "Hello,#{name}"
      end
    end
  end

  Dummy.new.say_hi("suzhen") #->  HI,suzhen

  Dummy.say_hello("suzhen") #-> Hello,suzhen

{% endcodeblock %}

  创建单例方法：

{% codeblock lang:ruby %} 
  obj = Dummy.new
  obj2 = Dummy.new
  class << obj
        define_method :say_hi do
          puts "HI"
        end
   end

   obj.say_hi #->HI
   obj.say_hi #-> mt_ad.rb:30:in `<main>': undefined method `say_hi' for #<Dummy:0x007fce6c109068> (NoMethodError)
{% endcodeblock %}

  说明：

  1、define_method是一个private方法

  2、由于define_method是一个private方法，它不能被显式调用。
  
  3、define_method是Module类的一个instance_method,只能由一个具体的module和一个具体的class来调用。
    
    因为:

{% codeblock lang:ruby %} 
  module DummyModule; end

  class Dummy; end

  p DummyModule.class #—>Module

  p Dummy.class #—>Class
  
  p Dummy.class.superclass #—>Module

{% endcodeblock %}

  一个具体的module或者一个具体的class都是Module类的实例。


  4、为当前self(必须是类，不管是普通类还是eigenclass)创建一个instance method,换句话说就是define_method消息的接收者只能是类对象。

  比如：

{% codeblock lang:ruby %} 
  Dummy.instance_eval do
    #当前self是Dummy类本身，所以此处定义的是Dummy的实例方法。
    define_method :say_hello do
      puts "hello,suzhen"
    end
  end

  class Dummy
    def self.create_method
      #类方法中的self，代表的也是Dummy类本身，所以此处定义的是Dummy的实例方法。
      define_method :say_hi do 
        puts "suzhen"
      end
    end
  end

  Dummy.new.say_hello

  Dummy.create_method
  
  Dummy.new.say_hi

  ————————————————————
  
  class << Dummy
    #此处的self是Dummy的eigenclass，所以就是创建eigenclass类的实例方法，也就是Dummy的类方法。
    define_method :say_bye do
      puts "bye"
    end
  end
  
  Dummy.say_bye
{% endcodeblock %}

  或者：

{% codeblock lang:ruby %} 
  class Dummy
    def self.create_class_method 
      class << self
         define_method :say_love do
          puts "love"
         end 

      end
    end
  end

  Dummy.create_class_method
  Dummy.say_love #—> love

  ————————————————————

  class Dummy
      def define_new_method_by_other_method

        #注意这里的self代表的是Dummy的实例对象。并不是类对象，所以是没有define_method方法的。

        define_method :say_hi do | name |
          puts "HI,#{name}"
        end
      end
  end

  Dummy.new.define_new_method_by_other_method #报错

{% endcodeblock %}

  综上所述可以看出，我们在使用define_method方法都是在当前类的内部。而作为private方法是不可以在外面调用的，比如：

  Dummy.define_method :say_hi do end # 访问保护

  或者

  obj = Dummy.new

  obj.class.define_method :say_hi do end＃访问保护


  但魔法师当然有他的魔术：send

  send 不仅可以动态调用方法，还可以调用私有方法。

{% codeblock lang:ruby %} 
  proc = lambda{ puts "suzhen" }
  Dummy.send :define_method,:say_hi,&proc
  Dummy.new.say_hi
{% endcodeblock %}
  或者

{% codeblock lang:ruby %} 
  obj.class.send :define_method,:say_hi,&proc
{% endcodeblock %}
  给obj的eigenclass动态定义方法

{% codeblock lang:ruby %} 
  obj.singleton_class.send  :define_method,:say_hi do
    puts  "suzhen"
  end

  obj.say_hi #suzhen

  obj2.say_hi #报错
{% endcodeblock %}

  在类的外部动态创建类的类方法

{% codeblock lang:ruby %} 
  Dummy.singleton_class.send  :define_method,:say_hi do
    puts  "suzhen"
  end
  
  Dummy.say_hi #—> suzhen
  
{% endcodeblock %}

  我们发现从外面给一个实例对象动态创建方法，或者给一个类动态创建方法，都需要先获取他们的eigenclass，再通过send 动态调用define_method方法，把要动态创建的方法名和block做为参数再传入。


  这样太麻烦了。有没有简单点的方法呢？魔法师给了我们一个魔法：define_singleton_method 它是Object类的一个实例方法。

{% codeblock lang:ruby %} 
  obj = Dummy.new
  obj2 = Dummy.new

  obj.define_singleton_method :say_hi do
    puts "suzhen"
  end

  obj.say_hi #-> suzhen
  
  obj2.say_hi #->报错
{% endcodeblock %}

  或者

{% codeblock lang:ruby %} 
  Dummy.define_singleton_method :say_hello do
      puts "hello"
  end

  Dummy.say_hello

  define_singleton_method 为我们的工作减少不小的工作量：
{% endcodeblock %}

  1，减少获取eigenclass的过程： (class << obj ;self; end) ｜ (class << Dummy ;self; end) 或者obj.singeton_class | Dummy.singenton_class

  2，减少了eigenclass使用send动态调用define_method的过程:   (eigenClass).send :define_method,agru,&block

  

  下一个要考虑的问题就是：

  设置动态创建的method的权限。

{% codeblock lang:ruby %} 
  obj.singleton_class.class_eval do
      private  :say_hi 
  end
  或者：
  obj.singleton_class.send :private,:say_hi

  obj.say_hi ＃报错

  Dummy.singleton_class.class_eval do
      private :say_hello
  end

  Dummy.say_hello #报错
{% endcodeblock %}

  
  
  我们来复习一下上面的内容发现：defind_methd 可以在一个普通类里动态定义方法，也可以在eigenClass里动态定义方法。可以在一个类内部动态定义方法，也可以在一个类外部通过send进行动态方法的定义。




  以上说的是method的动态定义，动态执行。  那么根据增删查改的模式，我还要了解 “删”，“查”，“改”（重载或者）

  那么如何删除一个方法？

 {% codeblock lang:ruby %}  
  undef_method  

  remove_method  相应的callback method_removed


  class Parent
      def hello
        puts "In parent"
      end
  end
  class Child < Parent
      def hello
        puts "In child"
      end
  end

  c = Child.new
  c.hello

  class Child
     remove_method :hello  # remove from child, still in parent
  end
  c.hello

  class Child
      undef_method :hello   # prevent any calls to 'hello'
  end
  c.hello

  In child
  In parent
  prog.rb:23: undefined method `hello' for #<Child:0x401b3bb4> (NoMethodError)
{% endcodeblock %}
  undef_method与remove_method的用法与define_method是一样的。区别在于，remove_method 只是移除了本类中的方法，但父类的方法还在。对象还是可以调用该方法的。但undef_method就彻底把方法从家族中移除了。

  如何查看有哪些方法呢？

  methods;
  instance_methods;
  private_instance_methods;protected_instance_methods;public_instance_methods;

