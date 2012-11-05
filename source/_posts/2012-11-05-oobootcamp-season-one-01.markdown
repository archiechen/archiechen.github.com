---
layout: post
title: "OOBootcamp Season One 01"
date: 2012-11-05 11:52
comments: true
categories: OOBootcamp javascript
---

有幸请到了姜志辉老师来到公司帮助我们实施《OOBootcamp》这项培训计划。这项计划将持续四期，每期四次课，课程将采用训练营的方式，以实战为主，从动手中提炼和总结知识，讲师只负责引导和指点，与传统的填鸭式培训有很大的不同。第一季我们采用的语言是javascript，用一种大家都不熟悉的语言，来理解面向对象（OO）的本质。
###创建对象
javascript中创建对象的两种方式,使用{}和new Object方式(objects.js)：
{% codeblock lang:javascript%}
var assert = require('assert')

describe('object', function() {
    it('{}', function() {
        var obj = {
            name: 'obj',
            des: 'des'
        };
        assert.equal('obj', obj.name);
        assert.equal('des', obj.des);
    });

    it('Object', function() {
        var obj2 = new Object();
        obj2.name = 'obj2';
        obj2.des = 'des';
        assert.equal('obj2', obj2.name);
        assert.equal('des', obj2.des);
    });
});
{% endcodeblock %}

代码中使用了nodejs中的单元测试框架mocha，使用之前需要先用npm安装，安装到全局环境中需要加入参数“-g”：
    $ npm install -g mocha
使用require引入mocha：
    var assert = require('assert')
在describe的function中编写测试方法，第一个参数是一组测试的名称：
    describe('object', function() {
        //这里写测试。
    });
在it的function中编写一个测试方法，第一个参数是单个测试的名称：
    it('{}', function() {
        //测试写在这里。
    });
运行测试使用mocha，而不是node：
    $ mocha  objects.js 
    ․․
    ✔ 2 tests complete (2 ms)

###函数和作用域
函数function是javascript的内建类型之一，javascript中的作用域分为全局作用域和局部作用域，局部作用域只体现在function中，即变量的作用域在声明变量的function中，未声明的变量默认使用全局作用域。
{% codeblock lang:javascript%}
var Person = function(name){
    var age=45;
    this.name = name;
    this.say_age=function(){
        return age;
    };
};
{% endcodeblock %}
在Person这个function中，age相当与一个似有变量，他只能在function内部访问到，而name和say_age都可以在外部访问，同时say_age相当于封装了对age的访问，这就是使用function模拟类的定义。使用代码如下：
{% codeblock lang:javascript%}
var jobs = new Person('jobs');
console.log(jobs.name);
console.log(jobs.say_age());
{% endcodeblock %}
执行结果：
    $ node functions.js 
    jobs
    45
如果不使用new来创建Person对象，如下代码：
{% codeblock lang:javascript%}
var iverson = Person('iverson');
console.log(typeof(iverson));
console.log(name);
console.log(say_age());
{% endcodeblock %}
不使用new的话，相当于只是执行了Person函数，由于函数没有返回值，因此iverson变量是undefined，而name和say_age则被赋值在全局作用域上。执行结果如下：
    $ node functions.js 
    undefined
    iverson
    45
这一节演示的是使用function模拟类，使用function的作用域模拟类的私有属性。

###callback和工厂模式
因为function也是对象，所以function也可以被当做参数或返回值处理。被当做参数时，就是callback方式，在函数内部可以执行传入的callback参数(callback.js)：
{% codeblock lang:javascript%}
var foo = function(func){
    var name = 'jobs';
    func(name);
};

foo(function(name){
    console.log(name);
});
{% endcodeblock %}
    $ node callback.js 
    jobs
下面定义一个普通的类Model(models.js)：
{% codeblock lang:javascript%}
var Model = function(id,name){
    var id = id;
    var name = name;

    this.say = function(){
        console.log(id+':'+name);
    };

};

var todo = new Model(1,'todo1');
todo.say();
{% endcodeblock %}
    $ node models.js 
    1:todo1
修改为工厂模式是这样的：
{% codeblock lang:javascript%}
var Model = function(id,name){
    var id = id;
    var name = name;

    return {
        say:function(){
            console.log(id+':'+name);
        }
    }
};

var todo = new Model(1,'todo1');
todo.say();
{% endcodeblock %}
    $ node models.js 
    1:todo1
使用工厂模式时有一个好处，当忘记使用new创建对象的时候，创建的对象行为和使用new时是一样的。但也有一个缺点，看以下代码：
{% codeblock lang:javascript%}
var todo = new Model(1,'todo1');
todo.say();
console.log(todo instanceof Model);
console.log(typeof(todo));
{% endcodeblock %}
    $ node models.js 
    1:todo1
    false
    object
使用工厂模式创建的对象，类型不是Model，而是object。当使用工厂模式时，要考虑应用场景是否依赖对象的类型。

###类的继承
继承是面向对象的一个重要特性，使用javascirpt模拟继承的代码(class.js)如下：
{% codeblock lang:javascript%}
var Class = function(){
    var inner = function(name){
         var name = name;
         this.show = function(){
             console.log('show ' + name);
         };
    };
    inner.inherts = function(c){
        for (i in c){
            inner.prototype[i] = c[i] ;
        };
    };
    return inner;
};

var obj = {
    info:function(){
        console.log('info');
    }
};

var obj2 = {
    save:function(){
        console.log('save');
    }
};

var Person = new Class(); // class Person()
Person.inherts(obj);
Person.inherts(obj2);
var jobs = new Person('jobs');
jobs.show();
jobs.info();
jobs.save();
{% endcodeblock %}
Class函数相当与定义类的语法，Class中的inner函数相当于类本身的定义，而inner函数中的name属性相当于类的私有属性。类的继承是通过在子类的prototype中增加父类的方法来实现。执行结果如下：
    $ node class.js 
    show jobs
    info
    save