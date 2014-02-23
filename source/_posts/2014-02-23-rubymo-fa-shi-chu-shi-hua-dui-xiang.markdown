---
layout: post
title: "RUBY魔法师(初始化对象)"
date: 2014-02-23 11:00
comments: true
categories: 
---
  我们会创建一个class。通过这个class去new出一个又一个对象。这个类里有许多的对象实例变量，有的是需要new的时候去设置的。有的对象实例变量有一些默认值。我们该怎么去设计一个好的类呢。

  最普通的设计方法：

{% codeblock lang:ruby %}
  class Volume
      attr_accessor :name, :size, :type, :owner, :date_created, :date_modified, :iscsi_target, :iscsi_portal

      SYSTEM = 0
      DATA = 1

      def initialize(args={})
        @name = args[:name]
        @size = args[:size]
        @type = args[:type].nil? ? SYSTEM :  args[:type]
        @owner = args[:owner]
        @iscsi_target = args[:iscsi_target]
        @iscsi_portal = args[:iscsi_portal]
        @date_created = Time.new(2013,2,2)
        @date_modified = Time.new(2013,2,2)
      end

      def inspect
        return {:name => @name,
                :size => @size,
                :type => @type,
                :owner => @owner,
                :date_created => @date_created,
                :date_modified => @date_modified,
                :iscsi_target => @iscsi_target,
                :iscsi_portal => @iscsi_portal }
      end

      def to_json
        self.inspect.to_json
      end
  end
{% endcodeblock %}


  要求：init的时候，可以传入配置项，也可以什么都不传。但无论传不传argument都要保证有@type的实例变量是有值的。

  我们要对这个类进行重构。

  用rspec 去保证重构的正确性。

{% codeblock lang:ruby %}
  require "./init"

  describe "Object Init" do


    context "all arguments are existing" do
      before :each do
        @v = Volume.new({:name=>'suzhen',:size=>"300",:owner=>"sue",:iscsi_target=>"",:iscsi_portal=>"",:type=>0})
      end

      it "Volume object name should has value" do @v.name.should == "suzhen" end
      it "Volume object size should has value" do @v.size.should == "300" end
          it "Volume object owner should has value" do @v.owner == "sue" end
      it "Volume object iscsi_portal should has value" do @v.iscsi_portal.should == "" end
      it "Volume object iscsi_target should has value" do @v.iscsi_target.should == "" end
      it "Volume object date_created should has value" do @v.date_created.should == Time.new(2013,2,2) end
      it "Volume object date_modified should has value" do @v.date_modified.should == Time.new(2013,2,2) end
      it "Volume object type should has value" do @v.type.should == 0 end
      it "to json should " do
       JSON.parse(@v.to_json).should == {"name"=>"suzhen", "size"=>"300", "type"=>0, "owner"=>"sue", "date_created" => Time.new(2013,2,2).to_s, "date_modified"=> Time.new(2013,2,2).to_s, "iscsi_target"=>"", "iscsi_portal"=>""}
      end
    end


    context "none arguments" do
      before :each do
        @v = Volume.new
      end
      it "Volume object name should nil" do @v.name.should == nil end
      it "Volume object size should nil" do @v.size.should == nil end
          it "Volume object owner should nil" do @v.owner == nil end
      it "Volume object iscsi_portal should nil" do @v.iscsi_portal.should == nil end
      it "Volume object iscsi_target should nil" do @v.iscsi_target.should == nil end
      it "Volume object date_created should nil" do @v.date_created.should == Time.new(2013,2,2) end
      it "Volume object date_modified should nil" do @v.date_modified.should == Time.new(2013,2,2) end
      it "Volume object type should has value" do @v.type.should == 0 end
      it "to_json should " do 
        JSON.parse(@v.to_json).should =={"name"=>nil, "size"=>nil, "type"=>0, "owner"=>nil, "date_created"=>"2013-02-02 00:00:00 +0800", "date_modified"=>"2013-02-02 00:00:00 +0800", "iscsi_target"=>nil, "iscsi_portal"=>nil}
      end
    end
    

  end

{% endcodeblock %}

  1，看到一长串的实例变量名称，会把这些名称放进一个array里，并且把这个array冻结起来，不能随意增加或者减少了。然后再遍历这个array,生成读写方法。

  注释掉这句 # attr_accessor :name, :size, :type, :owner, :date_created, :date_modified, :iscsi_target, :iscsi_portal

{% codeblock lang:ruby %}
   ATTRIBUTES = [
      :name, :size, :type, :owner, :date_created, :date_modified,:iscsi_target,:iscsi_portal
    ].freeze

    ATTRIBUTES.each do |attr|
      attr_accessor attr
    end
{% endcodeblock %}

    test通过。

   还可以再简单一点

  注释掉
  ＃ ATTRIBUTES.each do |attr|
    ＃  attr_accessor attr
   ＃ end

  改为：

   attr_accessor *ATTRIBUTES

   test通过。

  2 修改initialize方法，因为有了之前的ATTRIBUTES数组，这里就可以不用一个个写方法赋值了。
  
  注释掉原来的initialize方法

  ＃def initialize(args={})
    ＃    @name = args[:name]
   ＃     @size = args[:size]
    ＃    @type = args[:type].nil? ? SYSTEM :  args[:type]
    ＃    @owner = args[:owner]
    ＃    @iscsi_target = args[:iscsi_target]
     ＃   @iscsi_portal = args[:iscsi_portal]
    ＃    @date_created = Time.new(2013,2,2)
     ＃   @date_modified = Time.new(2013,2,2)
   ＃ end

  改为：

{% codeblock lang:ruby %}
    DEFAULTS = {
        :type => SYSTEM
      }.freeze

    def initialize(args=nil)
            args = args ? DEFAULTS.merge(args) : DEFAULTS
        ATTRIBUTES.each do |attr|
          if (args.key?(attr))
            instance_variable_set("@#{attr}", args[attr])
          end
        end
    @date_created = Time.new(2013,2,2)
           @date_modified = Time.new(2013,2,2)
      end

   test通过。

        3 修改inspect方法

   ＃def inspect
     ＃   return {:name => @name,
    ＃            :size => @size,
      ＃          :type => @type,
      ＃          :owner => @owner,
      ＃          :date_created => @date_created,
      ＃          :date_modified => @date_modified,
       ＃         :iscsi_target => @iscsi_target,
       ＃         :iscsi_portal => @iscsi_portal }
     ＃ end

  def inspect
        ATTRIBUTES.inject({ }) do |h, attr|
          h[attr] = instance_variable_get("@#{attr}")
          h
        end
    end
{% endcodeblock %}

   test通过。

  可以把inspect的效果写的更绚一点
  
  Hash[ *ATTRIBUTES.map { |attr| [ attr, instance_variable_get("@#{attr}") ] }.flatten ]

   test通过。


  4 如果我们觉得用initialize传入option这种方式太土，可以用一种更COOL的方式。

{% codeblock lang:ruby %}
  def initialize(&block)
       ＃第一步先初始化默认值
        ATTRIBUTES.each do |attr|
          self.__send__ "#{attr}=", DEFAULTS[attr]
        end
          ＃通过attr_access 定义的读写方法来赋值
        yield(self) if block_given?
    end
{% endcodeblock %}

  那么初始化方式为：

{% codeblock lang:ruby %}
  @v = Volume.new {|v| v.name = "suzhen"; v.size = "300";v.owner="sue";v.iscsi_target="";v.iscsi_portal="";v.date_created=Time.new(2013,2,2);v.date_modified=Time.new(2013,2,2)}
{% endcodeblock %}
















