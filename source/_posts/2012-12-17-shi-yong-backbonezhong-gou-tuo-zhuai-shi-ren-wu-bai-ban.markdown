---
layout: post
title: "使用Backbone重构拖拽式任务白板"
date: 2012-12-17 17:16
comments: true
categories: javascript backbone
---
上次用jqueryui实现了一个可以拖拽的任务白板，实际项目中随着功能逐渐增多，传统的js组织形式就很凌乱了，这个时候要引入前端js框架，来保证代码的质量，尤其是清晰度和扩展性。前端js的框架可选的很多，比如backbone.js、ember.js等，本文采用的是backbone。使用backbone先要引入两个js文件，一个是underscore.js，这个文件是backbone框架依赖的基础库，另外一个是backbone.js本身。
{% codeblock lang:html%}
  <script src="http://backbonejs.org/test/vendor/underscore.js"></script>
  <script src="http://backbonejs.org/backbone.js"></script>
{% endcodeblock %}

重构之前先分析一下页面内容，需要有以下几个元素：

* 模型：
    - 任务
    - 任务集合
* 视图：
    - 任务卡
    - 任务栏

使用继承Backbone.Model创建Task类作为模型,继承Bankbone.View创建TaskCard作为视图。
{% codeblock lang:javascript%}
var Task = Backbone.Model.extend({});

var TaskCard = Backbone.View.extend({});
{% endcodeblock %}
利用javascript是动态语言的特性，模型暂时不需要添加属性和方法，现在需要定义TaskCard视图的属性，一个Card是一个li元素，视图渲染时，会使用Model填充内容,填充的内容定义为一个模板（template）。
{% codeblock lang:javascript%}
var TaskCard = Backbone.View.extend({
    tagName: "li",
    //卡片模板，这里可以更复杂。
    template: _.template('<h5><%-title%></h5>'),
    //使用模板和模型渲染卡片成html。
    render: function() {
      this.$el.html(this.template(this.model.toJSON()));
      return this;
    }
});
{% endcodeblock %}
现在可以使用Task和TaskCard在页面上显示任务卡片了，我们可以动态的创建任务卡片。
{% codeblock lang:javascript%}
//创建任务对象
var task1 = new Task({'title':'在弹出窗口中输入任务信息，点击save按钮保存到数据库。'});
var task2 = new Task({'title':'任务卡片可以在状态栏之间拖拽。'});

//创建卡片对象
var taskCard1 = new TaskCard({model:task1});
var taskCard2 = new TaskCard({model:task2});

//将卡片放到NEW状态的任务栏中
$('#taskcards_ul').append(taskCard1.render().$el);
$('#taskcards_ul').append(taskCard2.render().$el);
{% endcodeblock %}
下一步将任务栏设计为一个视图类，方便进行动态扩展。
{% codeblock lang:javascript%}
var TaskBoard = Backbone.View.extend({
  tagName: "div",
  className:"cardbox",
  //使用预先定义的模板
  template: _.template($('#taskboard-template').html()),
  initialize:function(){
    this.render();
  },
  render: function() {
    //使用一个对象渲染模板
    this.$el.html(this.template({name:this.options.name}));
    return this;
  }
});
//创建一个名字为“New”的任务栏。
var newTaskBoard = new TaskBoard({
  name:'New'
});
//将任务栏显示到页面中
$('#taskboards').append(newTaskBoard.$el);
{% endcodeblock %}
现在页面上只有一个New状态的任务栏了。这里用到了一个模板taskboard-template，是预先在html中定义好的一段html代码，underscore.js使用JSON格式的Object对模板进行渲染，<%%>之间用Object的属性填充。Taskboard的模板如下：
{% codeblock lang:html%}
<script type="text/template" id="taskboard-template">
    <h4 class="page-header"><%-name%></h4>
    <ul id="taskcards_ul" class="ui-helper-reset"></ul>
</script>
{% endcodeblock %}
再添加两个任务栏，分别是Progress和Done。
{% codeblock lang:javascript%}

var progresssTaskBoard = new TaskBoard({
  name:'Progress'
});

var doneTaskBoard = new TaskBoard({
  name:'Done'
});

//将任务栏显示到页面中
$('#taskboards').append(progresssTaskBoard.$el);
$('#taskboards').append(doneTaskBoard.$el);
{% endcodeblock %}
现在三个状态栏都是动态创建的，加入任务卡的代码显得很不协调，因为是直接通过jquery将TaskCard放入到状态栏中，现在需要引入Collection类了。
{% codeblock lang:javascript%}
var TaskList = Backbone.Collection.extend({
  model: Task
});
{% endcodeblock %}
Collection是一组Model的集合，并提供了add、remove等集合操作的方法，接下来创建一个New状态的TaskList，并且和New状态的任务栏关联起来。修改之前的代码，让Taskboard可以和TaskList绑定，并监听TaskList的add事件，当add事件被触发时，调用Taskboard的addOne方法，创建一个任务卡并放到状态栏中。同时删除创建任务卡的代码，修改之前添加任务卡的逻辑，不需要在最外层创建TaskCard了。
{% codeblock lang:javascript%}
    var TaskBoard = Backbone.View.extend({
      tagName: "div",
      className:"cardbox",
      template: _.template($('#taskboard-template').html()),
      initialize:function(){
        this.render();
        //绑定一个任务集合
        this.tasks = this.options.tasks||new TaskList;
        //监听任务集合的add事件
        this.tasks.on('add',this.addOne,this);
      },
      render: function() {
        this.$el.html(this.template({name:this.options.name}));
        this.taskcards = this.$('#taskcards_ul');
        return this;
      },
      addOne: function(task){
        var taskcard = new TaskCard({model:task});
        taskcard.render().$el.appendTo(this.taskcards).fadeIn();
      }
    });
    //创建任务集合
    var newTasks = new TaskList;

    var newTaskBoard = new TaskBoard({
      name:'New',
      //设置任务栏关联的任务集合
      tasks:newTasks
    });

    //将卡片放到NEW状态的任务栏中
    newTasks.add(task1);
    newTasks.add(task2);
{% endcodeblock %}
页面布局调整完后加入拖拽效果，先套用之前的代码逻辑，只是将其转移到TaskCard和TaskBoard中。
{% codeblock lang:javascript%}
    var TaskCard = Backbone.View.extend({
      ... ...
      //类构造函数
      initialize:function(){
        //设置任务卡可以被拖拽，不需要每次render时重复设置，在这里只会设置一次。
        this.$el.draggable({
            revert: "invalid", 
            containment: "document",
            helper: "clone",
            cursor: "move"
        });
      },
      ... ...
    });

    var TaskBoard = Backbone.View.extend({
      ... ...
      initialize:function(){
        ... ...
        //使用闭包保持现在的上下文
        var that = this;
        //设置当前状态栏被拖入时的行为。
        this.$el.droppable({
            activeClass: "ui-state-highlight",
            drop: function( event, ui ) {
                var $card = ui.draggable;
                $card.fadeOut(function() {
                    $card.appendTo( that.taskcards ).fadeIn();
                });
                //输出拖拽之后，任务集合的变化。
                console.log(that.tasks.length);
                console.log(newTasks.length);
            }
        });
      },
      ... ...
    });
{% endcodeblock %}
现在页面上可以实现拖拽效果了，但我们观察浏览器的console发现，拖拽之后，任务集合的状态并没有变，只是视图变了，模型并没有变。我们希望拖拽结束后，被拖拽的Task从原来的集合中删除，并加入到拖入栏目关联的集合中。在drop函数中无法直接获取Task对象，可以利用模型的cid是唯一特点，将cid保存在li元素的id属性中，然后在drop时取出id属性，根据这个id遍历所有任务集合，获取Task对象。修改后代码如下：
{% codeblock lang:javascript%}
    //创建任务集合，三个都要在外部创建，并初始化到对应的TaskBoard中。
    var newTasks = new TaskList;
    var progressTasks = new TaskList;
    var doneTasks = new TaskList;

    var TaskCard = Backbone.View.extend({
      ... ...
      render: function() {
        this.$el.html(this.template(this.model.toJSON()));
        //把li元素的id属性设置为模型的cid
        this.$el.attr('id', this.model.cid);
        return this;
      }
    });

    var TaskBoard = Backbone.View.extend({
      ... ...
      initialize:function(){
        ... ...

        var that = this;
        this.$el.droppable({
            activeClass: "ui-state-highlight",
            drop: function( event, ui ) {
                var cid = $(ui.draggable).attr('id');
                ui.draggable.fadeOut(function() {
                    //遍历所有任务集合
                    _.each([newTasks,progressTasks,doneTasks],function(from){
                      //跳过当前状态栏的任务集合
                      if(that.tasks!=from){
                        var draggableTask = from.get(cid);
                        if(typeof draggableTask != 'undefined') {
                          from.remove(draggableTask);
                          that.tasks.push(draggableTask);
                        }
                      }
                    });
                    console.log(that.tasks.length);
                    console.log(newTasks.length);
                });
            }
        });
      },
      ... ...
    });

{% endcodeblock %}
现在观察控制台输出，已经达到我们预期的效果了。其实代码还有重构空间，fadeOut行为应该绑定到任务卡上，为TaskCard自定义一个事件叫dropped，from.remove会触发TaskList的remove事件，在响应remove事件时，再触发一次task的dropped事件，就可以将fadout行为和TaskBorad解耦。重构后的最终代码参考：[Gist](https://gist.github.com/4335764)
