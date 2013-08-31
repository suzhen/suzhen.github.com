---
layout: post
title: "GridFS学习和使用"
date: 2013-08-31 22:59
comments: true
categories: mongodb 
---
GridFS是mongdb的一种存放任意大小文件的功能。把文件存在数据库中的好处是便于检索，备份，以前以后分布式的扩展。而普通document只能保存16MB的文件。

一 基本概念：

  GridFS将文件存入两个collections中。

  **fs.files**
  存放文件相关的数据，比如大小，上传日期，文件类型等
	{
	  "_id" : <ObjectID>,
	  "length" : <num>,
	  "chunkSize" : <num>
	  "uploadDate" : <timestamp>
	  "md5" : <hash>

	  "filename" : <string>,
	  "contentType" : <string>,
	  "aliases" : <string array>,
	  "metadata" : <dataObject>,
	}
  **fs.chunks**
  存放文件的字节数据，每个chunks的document的大小是256k。一个大文件会变分割成多个document存放的。
	{
	  "_id" : <string>, 
	  "files_id" : <string>,有索引
	  "n" : <num>,有索引
	  "data" : <binary>
	}

  一个files的document对应着一或者多个chunks的document。

二 shell操作：

	./mongofiles <options> <commands> <filename>

	option: -d 指定数据库
		-h GridFS所在主机名。默认是本机，商品27017
		-u 用户名
		-p 密码

	command:
	      list 文件列表
	      search 根据文件名称查找
	      put 把文件存入数据库
	      get 从数据库中提出文件。注意：如果文件名是带路径的，那必须在当前文件夹下创建该目录结构才可以提出文件。
	      delete 删除数据库中文件

三 rails上传文件到GridFS:

  在rails4.0环境下用到相关gem
	gem "mongoid", github: 'mongoid/mongoid', ref: '11e45e5a30a45458b83db99ab6c9d9ccc337e66f'
	gem 'mongoid-grid_fs', github: 'ahoward/mongoid-grid_fs'

	gem "carrierwave"
	gem "carrierwave-mongoid", git: "git://github.com/jnicklas/carrierwave-mongoid.git"

  首先创建carrierwave的配置文件
	touch config/initializers/carrierwave.rb
  内容如下：
	 {% codeblock lang:ruby %}
	require 'carrierwave/mongoid'
	CarrierWave.configure do |config|
	  config.storage = :grid_fs
	  config.root = Rails.root.join('tmp')
	  config.cache_dir = "uploads" 
	  config.grid_fs_access_url   = "/images"  #@model.image_url.就是在数据库中的文件名称前，加上的路径(/images+数据库中的文件名称)。
	end
	 {% endcodeblock %}

  在upload目录下
	 {% codeblock lang:ruby %}
	 class PictureUploader < CarrierWave::Uploader::Base
	  #保存在数据库中的文件名
	  def store_dir                                                                                                                                                              
	     "uploads/#{model.class.to_s.underscore}/#{mounted_as}/#{model.id}"
	  end              
	 end
	 {% endcodeblock %}

  这样就可以所文件保存到GridFS中了。

  读取文件：
  1、用脚本读取数据库的数据，在写在页面上。这种方式是比较效率低，只适合在开发环境中使用。

  创建一个GridfsController负责读取GridFS中的文件，并写到页面上去。
	 {% codeblock lang:ruby %}
	class GridfsController < ActionController::Metal
	  def serve
	    gridfs_path = env["PATH_INFO"].gsub("/images/", "")
	    begin
	      gridfs_file = Mongoid::GridFS[gridfs_path]
	      self.response_body = gridfs_file.data
	      self.content_type = gridfs_file.content_type
	    rescue
	      self.status = :file_not_found
	      self.content_type = 'text/plain'
	      self.response_body = ''
	    end
	  end
	end
	 {% endcodeblock %}


 所有对文件的访问路径都路由到这个controller中。
	 {% codeblock lang:ruby %}
	  get "/images/uploads/*path" => "gridfs#serve"
	 {% endcodeblock %}

  2、用nginx-gridfs的方法









