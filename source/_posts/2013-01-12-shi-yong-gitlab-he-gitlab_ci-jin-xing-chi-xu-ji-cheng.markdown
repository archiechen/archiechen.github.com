---
layout: post
title: "使用gitlab和gitlab_ci进行持续集成"
date: 2013-01-12 22:18
comments: true
categories: git supervisor 自动化部署
---
Git是越来越火了，根据聪明的程序猿都是随大流的原则，我也开始使用Git了。当然，一个人用不如大家一起用，所以还需要一个团队能共享的git仓库，有朋友推荐了gitlab，看后感觉很好很强大，而且还有gitlab_ci进行持续构建。不过只是持续构建还不够，如果能持续部署就更好了：）

持续部署其实并不难，只要在持续构建的单元测试通过后，可以自动将最新的代码部署到测试环境，并重启测试环境的服务，我们就可以在测试服务器上访问到最新的内容了,这次先只介绍一下如何持续构建。

安装gitlab很简单，只要按照官方文档一步一步做就好了，有一点需要注意的就是，官方文档的步骤只适合Ubuntu和Debian系统。

[GitLab安装指南](https://github.com/gitlabhq/gitlabhq/blob/stable/doc/install/installation.md)

安装gitlab之后，我把gitlab_ci装在了另外一台服务器上，当然也可以装在一起。安装时需要注意第二步，文档中使用RVM方式安装的Ruby，我测试时对后续步骤有影响，就干脆按照GitLab安装指南的做法，直接安装的Ruby。如果装在同一台服务器上，还要注意nginx的端口配置，两个不能都使用80端口。

[GitLab CI安装指南](https://github.com/gitlabhq/gitlab-ci/blob/master/doc/installation.md)

安装一下nose，一会儿需要执行测试。
{% codeblock lang:bash%}
sudo pip install nose
{% endcodeblock %}
系统装后好，先在GitLab上创建一个项目，叫examples，我的服务器IP是192.168.8.202，项目的SSH地址是git@192.168.8.202:yachuan.chen/examples.git。

创建本地仓库：
{% codeblock lang:bash%}
mkdir examples
cd examples
git init
touch README
git add README
git commit -m 'first commit'
git remote add origin git@192.168.8.202:yachuan.chen/examples.git
git push -u origin master
{% endcodeblock %}

接下来在GitLab CI（以后简称CI）添加一个持续构建的项目，我的地址是http://192.168.8.201，这里会出现一个错误提示“Project path is not a git repository”，持续构建的目录必须是一个Git仓库，其实就是在CI服务器上要有一份Examples的Clone，现在SSH到CI服务器上去创建Project path。
由于CI是使用gitlab_ci用户运行的，所以也是用该用户进行clone。
{% codeblock lang:bash%}
$ cd /home/gitlab_ci/workspace
$ sudo -u gitlab_ci -H git clone git@192.168.8.202:yachuan.chen/examples.git
<div class=""></div>
FATAL: R any yachuan.chen/examples deploy_ab12176da49cec6816daf762dbe6e76d DENIED by fallthru
...
{% endcodeblock %}

直接这么做会报错，因为CI服务器还没有权限使用ssh进行clone，需要本地生成一个ssh key。
{% codeblock lang:bash%}
$ sudo -u gitlab_ci -H ssh-keygen -q -N '' -t rsa -f /home/gitlab_ci/.ssh/id_rsa
$ sudo -u gitlab_ci -H cat /home/gitlab_ci/.ssh/id_rsa.pub
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQC3rFoK7n+EYolWlLmPTWQxHiAsv7xFCuVVgEeSqstMolzcBOShIdh2sYj0fsfqmllMCqt6C52V66xCV50xsu4YkUA8Zo1MIVLwwpzAI0NeYuGVWjquQaeuLl+KKqmk7fq3NRCXc7/Ce8wb8VvWPxr89uhKuakhKXd6UylpY0DjhZHqQ5XSTBjjY/CMzv9rA6he+aofS5biJF2d5DxIvOkWjXzfBzrR1cGxtnkYYvG9dYJU8A4876LjqrHLY++hS6QtCXD9gSOk0aCQ9qzvNV+lvmg1919wMfGjRKd1CnXLfkGS45EjZxq8O/gzvxDxGNTXV42Zx/72WDl4y5jaCam7 gitlab_ci@smile-1
{% endcodeblock %}
在GitLab>Project>Examples>Deploy Keys中，点击Add Delpoy Key按钮，将id_rsa.pub的内容复制到Key中，再起个名字，再执行刚才的Clone命令就之后，可以就完成Add Project的操作了。注意，Scripts是必填项，这里先只填一个/usr/local/bin/nosetests，后续在补充。

进入CI系统的Project: examples → Details页面:
![Details页面](http://localhost:4000/images/gitlab_ci_project_detail.png)
我使用的GitLab是4.0版本，按照提示，将Project URL和Project Token复制到GitLab → Project → Services中的GitLab CI中。
这时，我们进行git push后，将会触发gitlab ci执行scripts中的脚本。





