---
layout: post
title: "RUBY继承initialize时的顺序"
date: 2014-02-23 10:52
comments: true
categories: 
---
RUBY继承initialize时的顺序

我们分析一下有几种继承情况：

1、一个子类继承一个父类。
{% codeblock lang:ruby %}
  class Parent
    def initialize(family_name)
      puts family_name
      puts "parent"
    end
  end

  class Son < Parent
    def initialize
      puts "son"
    end
  end
{% endcodeblock %}

  子类和父类都有initialize方法，子类的会覆盖父类的同名方法，父类的initalize方法不会被执行。

  Son.new #->  son

  如果子类打算调用父类的initalize方法，就必须使用super明显的调用:

{% codeblock lang:ruby %}
  class Son < Parent
    def initialize
      super("sue")
      puts "son"
    end
  end
  
  Son.new #->  sue parent son
{% endcodeblock %}


2、一个类include一个module。

  这种方式其实就是在让class继承一个“代理类”。这个“代理类”就是module生成的。

{% codeblock lang:ruby %}
  module Skill
    def initialize
      puts "skill"
    end
  end

  class Son 
    include Skill
    def initialize
      puts "son"
    end
  end
{% endcodeblock %}

  class和module都有initialize方法，class会覆盖module的同名方法，module的initalize方法不会被执行。

  Son.new #->  son

  如果class打算调用module的initalize方法，就必须使用super明显的调用:

{% codeblock lang:ruby %}
  class Son 
    include Skill
    def initialize
      super()
      puts "son"
    end
  end

  Son.new #->  skill son
{% endcodeblock %}

3、一个类include多个module.

  一个class能include多个module,这是Matz在解决类多继承的一个方式。我们要理解这种继承方式，class能继承多类，但是一层一层的继承，继承是有顺序的，是线性的，不是菱型的。

{% codeblock lang:ruby %}
  module Skill
    def initialize
      puts "skill"
    end
  end

  module Interest
    def initialize
      puts "interest"
    end
  end

  class Son 
    include Skill
    include Interest
    
    def initialize
      super()
      puts "son"
    end
  end
{% endcodeblock %}

  Son类include两个module的顺序就非常的重要，这会影响到module的“代理类”在继承线上顺序的。这个例子中 可以认为是son类继承了interest代理类，interest代理类继承了skill代理类。   Son<Interest<Skill

  同理：同名方法会被覆盖。super的调用只会调用父类，如果要调用祖父类的同名方法，就要在父类的同名方法中使用super。super只针对父类的同名方法，不能跨级调用。

{% codeblock lang:ruby %}
  module Skill
    def initialize
      puts "skill"
    end
  end

  module Interest
    def initialize
      super()
      puts "interest"
    end
  end

  class Son 
    include Skill
    include Interest

    def initialize
      super()
      puts "son"
    end
  end

  Son.new #->  skill interest son
{% endcodeblock %}

4、一个类即继承一个父类，又包含若干个module

  分析这个情况，要明白：“代理类”要比真正要继承的类，更先被继承。在继承线上更靠前。根据以上的学习我们会很快写出结果：

{% codeblock lang:ruby %}
  module Skill
    def initialize
      super("sue")
      puts "skill"
    end
  end

  module Interest
    def initialize
      super()
      puts "interest"
    end
  end


  class Parent
    def initialize(family_name)
      puts family_name
      puts "parent"
    end
  end

  class Son < Parent
    include Skill
    include Interest

    def initialize
      super()
      puts "son"
    end
  end

  Son.new #-> sue parent skill interest son
{% endcodeblock %}

  牢记：super() 一定要(),不然会有问题！！！
