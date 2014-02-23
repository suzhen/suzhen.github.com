---
layout: post
title: "RUBY魔法师(class_eval和instance_eval)"
date: 2014-02-23 00:23
comments: true
categories: 
---
在RUBY的编程过程中对一个已经创建好的类，或者对象，进行一些运行时的改造，比如增加一个方法，添加一些变量，去除一些方法，或者引入一个module等，是经常会做的事。

  我们先来看看对一个类的动态改造。

{% codeblock lang:ruby %}
  class Dummy 
  end

  obj = Dummy.new

  Dummy.class_eval do
    #defs here go to Dummy
    def say_hi
      puts "hi"
    end
  end

  obj.say_hi #—>hi
{% endcodeblock %}

  当我们定义class的时候，什么实例方法都没有定义。之后，魔法师通过class_eval在运行时，去添加了实例方法say_hi。class_eval do ..end 里的代码，就等于是在 class Dummy ..end里定义的方法。

  再看另一个类子：

{% codeblock lang:ruby %}
  Dummy.instance_eval do
    #defs here go to Dummy’s eigenclass
    def say_hello
      puts "hello"
    end
  end

  Dummy.say_hello ＃—>hello
{% endcodeblock %}

魔法师又通过instance_eval在运行时，去添加了类方法say_hello。instance_eval do .. end里的代码，就等于在Dummy的eigenclass里了定义方法。

  魔法师如果不用instance_eval ,在class_eval也可以定义类方法。

{% codeblock lang:ruby %}
  Dummy.class_eval do
    class << self
      def say_bye
        puts "bye"
      end
    end
  end
  Dummy.say_bye #->bye
{% endcodeblock %}

  我们来总结一下class_eval和instance_eval的区别：前者是改变class本身，在do..end里的代码被放入class Dummy …end 中，换成书中的句子就是改变了self. 而后者把所有代码都被放了Dummy.eigenclass里，但没有放在Dummy类中，这样就不会改变Dummy里原来的代码，换成书中的句子就是没有改变Self.

  这两个魔法在RAILS里被无数次的使用。

  对于一个类，实例方法是放在类里的，而类方法是放在这个类的eigenClass里的。
  每个对象，都有自己的EigenClass,其中定义的方法都是这个对象（仅仅这个对象）的实例方法。

{% codeblock lang:ruby %}  
  obj = Dummy.new
  class << obj
    def cry
      puts  “wawa”
    end
  end
  obj.cry #->wawa
{% endcodeblock %}

  而instance_eval魔法正是打开eigenClass的钥匙。

{% codeblock lang:ruby %}
  obj.instace_eval do
    def cry
      puts  “wawa”
    end 
  end
  obj.cry #-> wawa
{% endcodeblock %}

  ！！！发现在一个重大秘密：

  class << obj/class_name 是打开eigenClass类，obj/class_name.instance_eval也是打开eigenClass类。两者有什么区别?

  答案就是在self上。
{% codeblock lang:ruby %}
  class << obj/class_name
    self    #是指eigenclass
  end
{% endcodeblock %}
  而
{% codeblock lang:ruby %}
  obj/class_name.instance_eval do
    self  #是指obj/class_name本身
  end
{% endcodeblock %}
  这是最最大的区别！！！切记！！ 了解这一点就能解释清楚以下的手段。

  我们去获取到一个类或者对象的eigenClass,通常采用的方式是：

  一、获取类的eigenclass：
  方式一：
{% codeblock lang:ruby %}
  e = class << Dummy
    self
  end
{% endcodeblock %}  
  或者
{% codeblock lang:ruby %}
  e =  Dummy.instance_eval do
    class << self
      self
    end
  end
{% endcodeblock %}
  方式二：
{% codeblock lang:ruby %}
  class Dummy
    def self.eigenclass
      class << self
        self
      end
    end
  end
{% endcodeblock %}   

  二、获取对象的eigenclass：

  方式1：
{% codeblock lang:ruby %}
  class Dummy 
    def eigenclass
          class << self
              self
          end
     end
  end

  obj =  Dummy.new
  obj.eigenclass 获取obj的eigenclass
{% endcodeblock %}
  方式2：
{% codeblock lang:ruby %}
    eigenclass = class << obj
      self #此处的self代表了obj的eigenclass
    end
{% endcodeblock %}
  或者：
{% codeblock lang:ruby %}
  e = obj.instance_eval do
    class << self
      self #此处的self代表了obj的eigenclass
    end
  end
{% endcodeblock %}
    
  无论哪种方式都比较麻烦。魔法师会用一个特别的方式，直接获取到eigenclass。
{% codeblock lang:ruby %}
  Dummy.singleton_class
  obj.sigleton_class
{% endcodeblock %}


  再来看一个等式，做为就之前的一个总结，看懂了就说明，理解了。
{% codeblock lang:ruby %}
  Dummy.singleton_class.class_eval {}  == Dummy.instance_class {} 
{% endcodeblock %}
  但是 
{% codeblock lang:ruby %}
  Dummy.singleton_class.class_eval { self } != Dummy.instance_eval { self }
{% endcodeblock %}

  一、等号说明在｛｝里定义的方法都是Dummy的类方法。
  二、不等号说明在｛｝中的self是有不同含义的。singleton_class.class_eval 中的self是指Dummy的eigenclass,而Dummy.instance_eval是指Dummy类本身
  
{% codeblock lang:ruby %}
  Dummy.singleton_class.class_eval do
    def say_hi
      puts "hi"
    end
  end
  Dummy.instance_eval do
    def say_hello
      puts "hello"

    end
  end
  Dummy.say_hi
  Dummy.say_hello

  p Dummy.singleton_class.instance_methods    [:say_hi,:say_hello]

  p Dummy.singleton_class.class_eval { self } == Dummy.instance_eval { self } #->false
  p Dummy.singleton_class #--> #<Class:Dummy>
  p Dummy.singleton_class.class_eval { self } #--> #<Class:Dummy>
  p Dummy.instance_eval { self } #--> Dummy
{% endcodeblock %}

singleton_class在我们对方法的操作上有很多很多的用处，仔细写在RUBY魔法师（方法高级篇）

看看以下例子，再看看我们的重大发现，顿时明白一切。

{% codeblock lang:ruby %}
 obj = Dummy.new
  class << obj
    def eigenclass_func
      self #此处的self代表了obj本身，而不是obj的eigenclass
    end
  end
{% endcodeblock %}  
  或者
{% codeblock lang:ruby %}
  def obj.eigenclass_func
    self #此处的self代表了obj本身，而不是obj的eigenclass
  end
{% endcodeblock %}  
  或者
{% codeblock lang:ruby %}
  e = obj.instance_eval do
    self #此处的self代表了obj本身，而不是obj的eigenclass
  end
{% endcodeblock %}