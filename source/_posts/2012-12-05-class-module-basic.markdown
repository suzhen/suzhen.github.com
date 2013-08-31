---
layout: post
title: "class和module（基础篇）"
date: 2012-12-05 00:39
comments: true
categories: ruby 
---
   Class与Module是ruby程序大框架的基本构成。掌握好class与module的设计，对ruby程序的层次的清晰和扩展是非常重要的。class名与module名也都是常量。和我们定义的常量在某种意义上说没有区别。第一个字母都必须是大写。Class类是Module的子类，class是一特殊形式的模块。我们应该牢记最顶级的类是Object,最顶级的模块就是Kernel。我们从最简单的开始了解和熟悉他们的用法与设计。

   一、module的创建与mixin

   例子一：
 {% codeblock lang:ruby %}

	#创建module
	module Mymodule
	  def say
	    puts "#{self.class} say: hello"
	  end
	  #不仅能定义实例方法，还能定义常量，实例对象，类对象，宏，甚至是initialize方法。
	end

	module Myothermodule
	  def cry
	    puts "#{self} cry: wawawa~"
	  end
	end

	#在类中引用module
	class Myfirstclass
	  include Mymodule #include说明module里的方法是被当作实例方法被引用的,方法里的self代表对象
	  extend Myothermodule #extend说明module里的方法是被当作类方法被引用的，方法里的self代表类
 
	  #无论哪种引用，都要把module里的代码从头到底执行一遍，和定义类一样。
	end
 {% endcodeblock %}

 {% codeblock lang:ruby %}
	fobj=Myfirstclass.new
	fobj.say #Myfirstclass say : hello
	Myfirstclass.cry #Myothermodule say: wawawa~
 {% endcodeblock %}

可以用类的included_modules方法查看该类包含了多少个module，当然不包括extend的。
	
	Myclass.included_modules 

例子二：
我们还可以先不用在类里面混入模块，而是在类的实例里混入模块，可以达到同样的效果。
 {% codeblock lang:ruby %}
	module Mymodule
	  def say
	     puts "hello"
	  end
	end
	class Myclass
	end
  {% endcodeblock %}
*用实例对象extend模块就是实例方法
 {% codeblock lang:ruby %}
	obj = Myclass.new
	obj.extend(Mymodule)
	obj.say #hello
  {% endcodeblock %}
   
*用动态派发的方式也可以mixin一个模块
 {% codeblock lang:ruby %}
	Myclass.send :include,Mymodule
 {% endcodeblock %}

*用类extend模块就是类方法
 {% codeblock lang:ruby %}
	Myclass.extend(Mymodule)
	Myclass.say #hello
 {% endcodeblock %}
   
   以上的写法更多的是用在module的callback中
    

说明：module的作用之一是把若干类中，共有的逻辑部分提出来，起到“一处定义修改，所有引入”均可以改动的意义，目的是为了更好的组织类方法和实例方法。
我们可以把两个module放在别外一个文件内module.rb，在class文件里只要需要引用(require "路径/module")就可以使用。
重构代码的一步。mixin从技术层次理解，就是ruby自动创建了一个module名的代理类，并且让Myfirstclass类继承了这个代理类。实现一个类可以继承多个类的效果。
比如fobj或者sobj寻找方法，会先从自身类去找，如果找不到就从自身类的代理类去找，还找不到再从父类去找，如果还找不到，就去父类的代理找，一直向上。如果在某一处找到了，就停止寻找了。
还有一点要注意的是同名方法的查找,后面模块的引用方法覆盖前面模块的引用方法的. 如下例子


 {% codeblock lang:ruby %}
	module Pmodule
	  def say
	     puts "proxy say: hello"
             super
	  end
	end

	class Fclass
	  def say
	    puts "father say:hello"
	  end
	end

	class Sclass<Fclass
	  include Pmodule
	  def say
	   puts "son say:hello"
	   super
	  end
	end

	obj=Sclass.new
	obj.say
 {% endcodeblock %}

输出：
 {% codeblock lang:ruby %}
	"son say:hello"
	"proxy say:hello"    #super指向了module的方法say，而不是父类的say
	"father say:hello"   
 {% endcodeblock %}


   二、module与class的区别和理解
   
   1、module是不能实例对象的，也就是module不能拥有自己的实例变量。在module里的实例变量只能被混入的类的使用。但是常量可以直接使用。

     module Mymodule
       PI=3.14
     end
     class Myclass
       PI=3.14
     end
     
     Mymodule::PI #3.14
     Myclass::PI  #3.14

   2、module中的定义方法如果不是module的单例方法，只能混入类的被类或者类的对象执行。但如果是module的单例方法，那么可以直接执行，但不能被混入类中无论是(include,还是extend)。

     module Mymodule
        def self.say
            puts "#{self} say: hello"
        end
        def cry
            puts "wawawa~"
        end
     end   
     Mymodule.say  #Mymodule say:hello
     Mymodule.cry   #error

我们可以把这个module使用一个更显得专业的写法

     module Mymodule
       class<<self
          def say
              puts "#{self} say: hello"
          end
       end
     end   
     Mymodule.say  #Mymodule say:hello

既然单例方法可以直接使用，那么如果我们写一个宏的话，那么也可以直接引用变量了。
     
另一个例子：
     module Mymodule
      class<<self
        attr_accessor :name
        def say
            self.name="suzhen"
            puts "#{self.name} say: hello"
        end
      end
     end
     Mymodule.say  # suzhen.name say: hello
     puts Mymodule.name #suzhen

说明：name将成为Mymodule的一个变量一样可以使用了。非常的cool，感谢有ruby的单例方法。
**切记如果是module的单例方法混入是类里是没有用的。**

三，module与class的嵌套
   module里面可以嵌套若干个module，也可以嵌套若干个classs,class里面可以嵌套若干个class,也可以嵌套若干个module

	module Mymodule
	  module Myothermodule
	    PI=3.14
	  end
	end

	puts Mymodule::Myothermodule::PI

输出：#3.14



	class Myclass
	  module Mymodule
	    PI=3.14
	    def self.say(who)
	      Inclass.new.say(who)
	    end
	  end
	  class Inclass
	    def say(who)
	     puts "#{who},hello,world"
	    end
	  end
	  def sound
	    Inclass.new.say(self.class::Mymodule::PI)
	  end
	end
	puts Myclass::Mymodule::PI  #3.14
	puts Myclass::Mymodule.say("suzhen")  #suzhen,hello world
	puts Myclass.new.sound #3.14,hello world

   现在讨论一下嵌套的意义：
   module嵌套class的意义就像namespace一样，可以把一部分相关的类逻辑上分开。
   class里嵌套module也是同样的作用。我们在元编程中会，会打开一个类，把自己的逻辑写进去，这样我们就需要用一个module来把自己的逻辑隔开。好比一个独立的逻辑空间，该moduel通过class<<self...end使自身也带有方法，属性等。

   这里有两个非常有用的方法：一个是来判断一个类或者模块里是否嵌套着别一个类或者模块。二个是来获取这个名称的类或者模块。
	Mymodule.const_defined?("Myothermodule") 
	Mymodule.const_get("ClassMethods").class #Module
   通常会这种用：
	base.extend const_get("ClassMethods") if const_defined?("ClassMethods")


   四、module的callback方法
   module被include后会有一个callback的方法，我们可以重写这个方法来完成一些事。

   例子一：
	   module Mymodule
	     def self.included(cls)
		puts "minix in #{cls}"
	     end
	   end

	   class Myclass
	      include Mymodule
	   end

	   输出：
	   minix in Myclass

   如果是这个module被extend则不会callback这个方法

   说明：这个callback有一个非常有用的技巧可以用到。可以解决怎么在一个module中即有实例方法，又有类方法，而在混入类中只需要include就可以了。

   例子二：
	   module Base
	       #实例方法
	       def show  
		  puts "You came here!"  
	       end  
	 
	       #扩展类方法  
	       def self.included(base)  
		  #额外创建几个类方法
		  def base.say  
		    puts "I'm strong!"  
		  end  
		  #混入类方法
		  base.extend(ClassMethods)  
	       end  
	   
	       #类方法  
	       module ClassMethods  
		 def hello  
		   puts "Hello baby!"  
		 end  
	       end  
	   end  
	 
	  class Bus  
	    include Base  
	   end 

	  Bus.new.show
	  Bus.say  
	  Bus.hello 
	    

   五、module依赖另一个module的方式和问题
   
例子一：
	module Mymodule
	  def say
	    puts "#{self.name} say: hello" #这个self还是类或者类对象
	  end
	end

	module Myren
	  include Mymodule
	end

	class Myclass
	  include Myren
	  def name
	    name="suzhen"
	    name
	  end
	end

	a = Myclass.new
	a.say  #suzhen say:hello
	   
   会出现的问题在是callback上

	   module Mymodule
	     def self.included(base)
	       puts base  #Myren 而不是Myclass
	     end
	   end

	   module Myren
	     include Mymodule
	     def self.included(base)
	       puts base #这里才是Myclass
	     end
	   end

	   class Myclass
	     include Myren
	   end

	   a = Myclass.new

    
   问题：如果我想在Mymodule的included中做一些针对混入类的操作，就变的不可能了，因为此时的base是中间的那个module了，而不是混入类了。解决这个问题的笨方法就是
   
   Myren中去掉include Mymodule,类Myclass中引入两个module

	class Myclass
	  include Myren,Mymodule
	end  
    
   但这么做会有一个问题就是。有时候我们只想引用一个Myren，那就没办法了。那么怎么解决这个问题？Rails里有一个模块可以帮我们解决这个问题。
  
   例子二：   
	require 'rubygems'
	require 'active_support/concern'
	module Mymodule
	   extend ActiveSupport::Concern
	   included do  
	     puts self  #Myclass
	   end
	end

	module Myren
	  extend ActiveSupport::Concern
	  include Mymodule
	  included do
	    puts self  #Myclass
	  end
	end

	class Myclass
	  include Myren
	end

	a = Myclass.new

   首先引入active_support/concern文件，在两个module中分别加入extend ActiveSupport::Concern，这样两个模块就不要再用自己的callback函数了。而是改成 included do ... end 
   这个模块中的self就是混入类本身了。神奇的是不光为你做了这些，还省了你在一个module中定义实例方法和类方法的技巧。只需要把类方法放在一个名叫：ClassMethods的module里即可，非常的方便。
   
   例子三：
	require 'rubygems'
	require 'active_support/concern'
	module Mymodule
	  extend ActiveSupport::Concern
	  module ClassMethods
	    def say
	      puts "hello"
	    end
	  end

	  module InstanceMethods #也可不放把实例方法放在这个module里。
	    def cry
	     puts "wawawa~"    
	    end
	  end
	end

	class Myclass
	  include Mymodule
	end

	Myclass.say #hello
	Myclass.new.cry #wawawa

  在ruby的世界中说是针对类进行逻辑结构的搭建，不如说是针对模块的逻辑结构搭建。大量的逻辑会写在module中，而不是直接写在类的，因为那样的重用性会很差。
  良好的程序结构就是有不同的module和class在不同的层次上发挥着作用。比如：如果我们打一个别人的类，那想写点类方法，其实是很危险的，很容易就出现猴子补丁的情况。
  那么干脆新建一个module,然后让那些类方法，成为module的单例方法，就可以类似类方法的调用了。Redis::search::query()
    
  六、class也有一个callback方法，就是继承后self.inherited方法,如果class被继承就会callback这个方法，如果class的子类再被继承同样也会callback该class这个方法。


	class Myclass
	  def self.inherited(subclass)
	    p subclass
	  end
	end

	class Mysubclass<Myclass
	end

	class Mygrandsonclass<Mysubclass
	end

输出：
	Mysubclass
	Mygrandsonclass











