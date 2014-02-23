---
layout: post
title: "RUBY魔法师(inject用法)"
date: 2014-02-22 23:46
comments: true
categories: 
---
inject是魔法师常常用到的一种魔法，更具体的说应该是 Enumerable#inject 。我特别特别的钟爱它，它非常的强大，可以让代码看上去很简洁。
  当发现有可能会用到地方，就要尽可能的去施展这种魔法。

  数字求和是最常使用inject的例子，当对一个array of numbers 里的number求和时，就可以施展inject魔法，而不是很土的用array each 的方式去求各。

{% codeblock lang:ruby %}
  [1, 2, 3, 4].inject(0) { |result, element| result + element } # => 10
{% endcodeblock %}

  inject方法使用一个argument和block, 对于这个array中包含的每一个element,这个block将会执行一次. 

  具体步骤如下：

  argument将被传递给block的第一个参数，这个例子中就是block中的result.
  array中第一个element被传递给block的每二个参数，这个例子中就是把1传递给block中的element.
  这个block就变成了 ｛｜0，1｜ 0＋1 ｝
  执行block 
  把执行block的结果，即 1，再次传给block的第一个参数。
  array中的第二个element被传递给block的每二个参数，这个例子中就是把2传递给block中的element
  这个block的就变成了 ｛｜1，2｜1＋2 ｝
  执行block
  把执行block的结果，即 3，再次传给block的第一个参数。
  array中的第二个element被传递给block的每二个参数，这个例子中就是把3传递给block中的element
  以此类推。     

  这个例子中 block会被执行4次
  {% codeblock lang:ruby %}

   {|0,1| 0+1}
     |-----⬆️ 
     ⬇️ 
   {|1,2| 1+2}
     |-----⬆️  
     ⬇️ 
   {|3,3| 3+3}
     |-----⬆️ 
     ⬇️ 
   {|6,4| 6+4}

  =>10
  {% endcodeblock %}

  魔法其实就是多次执行block, 每次把当前block的返回结果，再作为下一个block的第一个参数传进去，同时传入Enumentable中的下一个element,这样就又组成了一个新的block. array需要each多少次，就会执行多次block. 那么inject最将返回的结果，就是最一次block执行的结果。

  了解inject的原理就可以创造场景的应用了
  
  建立一个hash

{% codeblock lang:ruby %}
  hash = [[:first_name, 'Shane'], [:last_name, 'Harvie']].inject({}) do |result, element|
      result[element.first] = element.last
     result
  end

  # => {:first_name=>"Shane", :last_name=>"Harvie"}
 {% endcodeblock %}

  可以更简单的写：
{% codeblock lang:ruby %}
  hash = [[:first_name, 'Shane'], [:last_name, 'Harvie']].inject({}) do |result, element|
      result.merge(element.first.to_sym => element.last)
  end
{% endcodeblock %}



{% codeblock lang:ruby %}
  TestResult = Struct.new(:status, :message)
  results = [
      TestResult.new(:failed, "1 expected but was 2"),
      TestResult.new(:sucess),
      TestResult.new(:failed, "10 expected but was 20")
  ]

  grouped_results = results.inject({}) do |grouped, test_result|
      grouped[test_result.status] = [] if grouped[test_result.status].nil?
      grouped[test_result.status] << test_result
     grouped
  end

    grouped_results
    # >> {:failed => [
    # >>    #<struct TestResult status=:failed, message="1 expected but was 2">, 
    # >>    #<struct TestResult status=:failed, message="10 expected but was 20">],
    # >>  :sucess => [ #<struct TestResult status=:sucess, message=nil> ]
    # >> }

{% endcodeblock %}



  建立一个Array

{% codeblock lang:ruby %}
  array = [1, 2, 3, 4, 5, 6].inject([]) do |result, element|
      result << element.to_s if element % 2 == 0
      result
  end

   # => ["2", "4", "6"]
{% endcodeblock %}

{% codeblock lang:ruby %}

  TestResult = Struct.new(:status, :message)
  results = [
      TestResult.new(:failed, "1 expected but was 2"),
      TestResult.new(:sucess),
      TestResult.new(:failed, "10 expected but was 20")
  ]

  messages = results.inject([]) do |messages, test_result|
     messages << test_result.message if test_result.status == :failed
       messages
  end
  # => ["1 expected but was 2", "10 expected but was 20"]

{% endcodeblock %}



  inject还可以用在动态给object发消息的

{% codeblock lang:ruby %}

class Recorder
  instance_methods.each do |meth|
    undef_method meth unless meth =~ /^(__|inspect|to_str|object_id)/
  end

  def method_missing(sym, *args)
    messages << [sym, args]
    self #返回reciver，是为了可以连续发消息。
  end

  def messages
    @messages ||= []
  end

  def play_for(obj)
    messages.inject(obj) do |result, message|
      result.send message.first, *message.last
    end
  end

end
 

 class Machine
    def will(*args); 
  puts "will";
  self;
 end

    def record(*args);puts "record";self; end

    def anything;puts "anything";self;end
    
    def you;puts "you";self;end

    def want;puts "want";self;end

 end

recorder = Recorder.new.will.record.anything.you.want


# >> #<Recorder:0x28ed8 @messages=[[:will, []], [:record, []], [:anything, []], [:you, []], [:want, []]]

recorder.play_for(Machine.new)

# will
# record
# anything
# you
# want

{% endcodeblock %}

施展这个魔法的时候，要注意，def 某个method时，一定要返回reciver本身，才可以连续的调用。

这个例子也是一个DSL的例子，也是一个方法延迟执行的例子。就好像.where().where().where()

