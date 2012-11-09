---
layout: post
title: "bootstrap+jquery+jquerui实现拖拽式任务白板"
date: 2012-11-09 17:59
comments: true
categories: javascript jquery
---

bootstrap是一套Html+CSS+Javascript的轻量框架，可以很容易的实现美观的UI界面和动态效果，最近需要实现个拖拽效果，只用bootstrap没有现成的办法，就引入了jqueryui+jquery。基础页面样式引用了[bootstrap starter](http://twitter.github.com/bootstrap/examples/starter-template.html)。

首先，把页面中引入的javascript和css都改成完整的URL，并加入jqueryui和jquery。
{% codeblock lang:html%}
<link href="http://twitter.github.com/bootstrap/assets/css/bootstrap.css" rel="stylesheet">
<link href="http://twitter.github.com/bootstrap/assets/css/doc.css" rel="stylesheet">
<link href="http://code.jquery.com/ui/1.9.1/themes/base/jquery-ui.css" rel="stylesheet"/>
<link href="http://twitter.github.com/bootstrap/assets/css/bootstrap-responsive.css" rel="stylesheet">
{% endcodeblock %}
注意js文件放在页面的最后，这样可以使页面加载更快。这里我只引入了一个了bootstrap.js。
{% codeblock lang:html%}
<script src="http://twitter.github.com/bootstrap/assets/js/jquery.js"></script>
<script src="http://code.jquery.com/ui/1.9.1/jquery-ui.js"></script>
<script src="http://twitter.github.com/bootstrap/assets/js/bootstrap.js"></script>
{% endcodeblock %}
然后删除页面中Container原来的内容，换成我们需要的白板栏，共有三个状态栏：New、Progress和Done。
{% codeblock lang:html%}
<div class="container">
    <div id="new_tasks" class="cardbox">
      <h4 class="page-header">New</h4>
    </div>
    <div id="progress_tasks"  class="cardbox">
      <h4 class="page-header">Progress</h4>
    </div>
    <div id="done_tasks"  class="cardbox">
      <h4 class="page-header">Done</h4>
    </div>
</div>
{% endcodeblock %}
cardbox是自定义的样式表，用来显示一个状态栏位。
{% codeblock lang:css%}
.cardbox { border:1px solid #AAA; float: left; width: 31%; min-height: 18em; padding: 0.5em;}
.cardbox h4 {margin: 0px;}
{% endcodeblock %}
加上样式后是这样的：

![效果图](http://archiechen.github.com/images/bootstrap.png)

增加两个个卡片，这里使用html中的ul和li元素实现，不过要调整一下样式表，li元素用了样式ui-helper-reset，这是jqueryui支持的，目的是不显示列表前的原点儿。
{% codeblock lang:html%}
<div class="cardbox">
    <h4 class="page-header">New</h4>
    <ul class="ui-helper-reset">
        <li><h5>在弹出窗口中输入任务信息，点击save按钮保存到数据库。</h5></li>
        <li><h5>任务卡片可以在状态栏之间拖拽。</h5></li>
    </ul>
</div>
{% endcodeblock %}
要增加一些css，控制任务卡片大小、内外边距以及拖拽时的鼠标图标。custom-state-active一会儿会用到，当状态栏允许放入时会变色。：
{% codeblock lang:css%}
.cardbox {border: 1px solid #AAA;float: left;margin-left: 10px;min-height: 46em;padding-left: 0.2em; width: 31%; }
.cardbox h4 { line-height: 16px; margin: 0 0 0.4em; }
.cardbox h5 { margin: 0.1em 0.1em 0 0.1em; }

.cardbox.custom-state-active { background: #eee; }
.cardbox li {background-color: white;border: 1px solid #AAA;color: #222;float: left;height: 84px;list-style: none;margin: 0.3em;padding: 0.2em;width: 164px;}
.cardbox li a { cursor: move; margin: 0 0 0.4em; }
.cardbox li p { float: right; margin: 0 10px 10px; }
{% endcodeblock %}
样式加好后，可以写js了，在页面的最后加上javascript块，先允许New中的任务卡片可以拖拽，具体参数参考[jqueryui文档](http://api.jqueryui.com/draggable/)：
{% codeblock lang:javascript%}
var $new_tasks = $( "#new_tasks" ),
    $progress_tasks = $( "#progress_tasks" ),
    $done_tasks = $( "#done_tasks" );

$( "li", $new_tasks ).draggable({
    revert: "invalid", 
    containment: "document",
    helper: "clone",
    cursor: "move"
});
{% endcodeblock %}
现在任务卡片可以拖动了，但是还不能放到其他的状态栏目里，还需要设置其他栏目允许放置。
{% codeblock lang:javascript%}
$progress_tasks.droppable({
    accept: "#new_tasks li",
    activeClass: "ui-state-highlight",
    drop: function( event, ui ) {
        move_item( ui.draggable , $progress_tasks);
    }
});

function move_item( $item, $tasks) {
    $item.fadeOut(function() {
        var $list = $( "ul", $tasks ).length ?
            $( "ul", $tasks ) :
            $( "<ul class='ui-helper-reset'/>" ).appendTo( $tasks );
        $item.appendTo( $list ).fadeIn();
    });
}
{% endcodeblock %}
accept参数指定了那些元素可以被放入该栏目，这里允许New状态的任务卡放入。move_item函数负责将拖拽的元素从原栏目放入新栏目，如果新栏目中不存在ul元素，则创建一个，move_item函数在将卡片拖拽到Progress栏目中，释放鼠标时触发调用drop。按照这个逻辑，设置Done状态栏，只允许拖入Progress状态栏中的任务卡。此时拖拽New状态时，将不能放入Done状态栏。
为了可以将人物卡片重新放回New状态，需要设置New状态栏允许放置,并设置Progress状态和Done状态都可以拖入。
{% codeblock lang:javascript%}
$new_tasks.droppable({
    accept: "#progress_tasks li,#done_tasks li",
    activeClass: "ui-state-highlight",
    drop: function( event, ui ) {
        move_item( ui.draggable , $new_tasks);
    }
});
{% endcodeblock %}
完整代码在这里[Gist](https://gist.github.com/4045555)