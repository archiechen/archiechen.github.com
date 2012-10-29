---
layout: post
title: "MacOS下安装ruby-1.9.3-p286"
date: 2012-10-29 17:34
comments: true
categories: 
---

在小峰同学的帮助下，终于配好了[Octopress](http://octopress.org)，装ruby的时候遇到了问题。因为mac之前装的是ruby1.87，而Octopress需要1.9以上的版本，只好升级了。但不知为何，使用rvm安装ruby奇慢无比，不知道什么时候能装完，实在忍受不了了，就自己下载了ruby（1.9.3）的源代码，configure make & make install之后，执行gem的时候会报异常：

    It seems your ruby installation is missing psych (for YAML output).

google了一下，需要安装yaml的包，于是下载之：
    
    wget http://pyyaml.org/download/libyaml/yaml-0.1.4.tar.gz

安装之后，有重新ruby，但执行gem仍然报同样的错误，又仔细搜索了一下google，找到一篇文章《[Install Ruby 1.9.3 with libyaml on CentOS](http://collectiveidea.com/blog/archives/2011/10/31/install-ruby-193-with-libyaml-on-centos/)》,发现做法是一样的，只是执行configure时需要指定参数，而我使用的都是默认值，虽然指定的和默认值是一样的。按照文中的做法操作之后，成功了！

    $ wget http://pyyaml.org/download/libyaml/yaml-0.1.4.tar.gz
    $ tar xzvf yaml-0.1.4.tar.gz
    $ cd yaml-0.1.4
    $ ./configure --prefix=/usr/local
    $ make
    $ make install

    $ wget http://ftp.ruby-lang.org/pub/ruby/1.9/ruby-1.9.3-p286.tar.gz
    $ tar xzvf ruby-1.9.3-p286.tar.gz
    $ cd ruby-1.9.3-p286
    $ ./configure --prefix=/usr/local --enable-shared --disable-install-doc --with-opt-dir=/usr/local/lib
    $ make
    $ make install