## 设计模式分类

### 1.创建型

- 构造器模式
- 工厂模式
- 抽象工厂模式
- 原型模式
- 单例模式
- 建造者模式

### 2.结构设计型

- 装饰模式
- 外观模式
- 享元模式
- 适配器模式
- 代理模式

### 3.行为设计型

- 迭代模式
- 中介者模式
- 观察者模式
- 访问者模式

---

### 1.构造器模式

#### 1).对象创建

``` javascript
var newObject = {};

var newObject = Object.create(null);

var newObject = new Object();

```

``` javascript
.

[]

defineProperty

defineProperties
```

#### 2).构造函数

- 希望属性独立，方法共享
- 最优使用基于原型的构造器

``` javascript
function Foo(value) {
  this.value = value;
}

Foo.prototype.method = function() {
  return this.value;
}

var Bar = new Foo("hello");
Foo.method();
```

### 2.模块化模式

#### 1).基础模块化

利用闭包立即执行封装私有属性方法，保存变量

``` javascript
var myModule = (function(){

  // private
  var data = [];

 return {

  add: function(item) {
    data.push(item);
  },

  getCount: function() {
    return data.length;
  }

 };

})();

myModule.add({
  name: 'rccoder',
  sex: 'F'
});
myModule.getCount();
```

#### 2).模块化变体

- 实现“真正”的封装
- 支持所谓的变量方法私有化
- 无法为私有成员创建自动化的单元测试，不能及时的为私有成员添加补丁

##### a).Import mixins(导入混合)

jQuery，understore 这种导入全局的问题。

``` javascript
var myModule = (function($, U) {

  function methodPrivate1() {
    $('#container').html('Hello  World!');
  }

  function methodPrivate2() {
    U.filter([1, 2, 'a'], isNumber);
  }

  return {
    methodPublic1: function() {
      methodPrivate1();
    },

    methodPublic2: function() {
      methodPublic();
    }

  };
}(jQuery, Underscore));

myModule.methodPrivate1();
```

##### b).Exports(导出)

可以申明全局变量但是不使用它们

``` javascript
var myModule = (function(){
  var module = {};

  var variablePrivate = 1;

  function methodPrivate() {

  };

  module.variablePrivate = 2;
  module.methodPublic = function() {

  };

  return module;
}());
```

##### c). 工具箱模式

由相关框架或者工具实现的模块化方法。

YUI

``` javascript
Y.namespace('store.basket') = (function() {
  var varPrivate = 1;

  var methodPrivate = function() {
    Y.log('Yes! I can');
  }

  return {
    varPublic: 2,
    methodPublic: function() {
      Y.log('Hi! I am a public method');
      Y.log(varPrivate);
      Y.log(methodPrivate());
    }
  };

}());
```

 jQuery

实现目标的初始化，出自 Ben Cherry 提供的方案

``` javascript
function lib(module) {
  $(function() {
    if(module.init) {
      module.init();
    }
  });

  return module;
}

var myLib = lib(function() {
  return {
    init: function() {

    }
  };
}());
```

#### 3).暴露式模块模式

在之前的模式中，当我们想调用一个公共方法中的另外一个公共方法的时候，我们不得不重复主对象的名称。

现在我们在 私有域中定义我们所有的函数和变量，返回一个匿名的对象，对象中包含指向想要暴露的私有成员，使得这些私有变量方法公有化。

更加重要的是，对于返回的这个匿名对象我们可以优雅的写key的名词来让语法更加一致，能在很高的程度上增加可读性。

``` javascript
var revealModule = (function() {
  var varPrivate, varPublic;

  function methodPrivate() {

  }

  function methodPublic() {
    methodPrivate();
  }

  return {
    varPublic: varPublic,
    methodPulic: methodPublic
  }
}());

revealModule.methodPublic();
```

但是这种方法的缺点也是特别明显的，比如一个私有函数需要使用公有函数的时候，这个公有函数无法被重载。即私有函数是私有的实现，不能用于公共成员。

### 3.单例模式

* 限制一个类只能有**一个实例化对象**
* 一般来说是创建一个类 ，这个类包含一个方法，这个方法会在没有对象存在的情况下创建一个新的实例对象。如果对象存在，只返回这个对象的引用
* `javascript` 中借助“闭包"的思想完成

``` javascript
var mySingle = (function() {
  // 存储这个单例实例的引用
  var instance;
  
  function init() {
    var privateRandom = Math.random();
    
    return {
      getRandom: function() {
        return privateRandom
      }
    }
  }
  
  return {
    getInstance: function() {
      if(!instance) {
        instance = init();
      }
      return instance;
    }
  }
}());

// 创建实例
var mySingle1 = mySingle.getInstance();
var mySingle2 = mySingle.getInstance();

mySingle1.getRandom() === mySingle2.getRandom();   // true
```

* 单例模式不像静态模式，在真正用到之前是不需要分配资源的
* 一般用在一个对象要和另外的对象进行跨系统协作的时候。
* 单例模式更加难以测试

### 4.观察者模式

* 存在一个被称作是被观察者的对象，维护一组被称为观察者的对象，这些对象都依赖这被观察者，被观察者自动将自身的状态的任何变化都通知给观察者。
  
* 当被观察者需要将一些变化通知给观察者的时候，会采用广播的方式，广播包含特定的通知与数据
  
* 当观察者不需要接收这些通知的时候，被观察者就可以将他从所维护的关系中删掉。
  
* > 一个或者更多的观察者对一个被观察者的状态感兴趣，将自身的这种兴趣通过附着自身的方式注册在被观察者身上。当被观察者发生变化，而这种便 可也是观察者所关心的，就会产生一个通知，这个通知将会被送出去，后将会调用每个观察者的更新方法。当观察者不在对被观察者的状态感兴趣的 时候，它们只需要简单的将自身剥离即可。

#### 发布/订阅模式

* 观察者模式想要接收被观察者的通知的时候要在发起这个通知的被观察者上面进行相关的注册事件
* 发布/订阅模式依赖的是一个主题/事件频道，这个频道位于订阅者和发布者之间。这个事件系统中可以传递订阅者需要的值，来极其方便的避免订阅者与发布者之间的依赖性
* 不是对象直接调用另外一个对象的方法，而是加了中间的一层。即通过订阅另外一个对象的一个特定的任务事件，等待这个任务事件出现的时候得到通知
* pubsub.js

缺点是：

* 发布者不能确定订阅者是否正常进行
* 订阅者之间没有感知

##### 例子：解耦搜索内容 ——Ajax

``` 
1.输入内容，点击搜索
2.发布publish(search/content，data)
3.订阅subscribe(search/content, data)
4.
  a)一个订阅触发事件append“正在搜索xxx” 
  b)一个订阅事件触发Ajax拉取数据，去的数据后发布 publish(search/result, data)
5.订阅subscribe(search/result, data),append搜索到的内容
```

这样的话`ajax`只关注数据，不用理会其他DOM操作

### 5.中介者模式

* 观察模式中共享被观察对象
* 取消了对象之间直接的发布订阅，用维护一个中间节点来代替
* Mediator.js
* 中间层可能会造成性能上的损失

#### 与观察者模式的区别

* 观察者模式中观察者和主题互相来维护关系，观察者有可能是另外一个观察者的主题。也就是没有封装单一的对象
* 中间人这种模式让观察者创建了观察者对象，这些观察者对象会发布事件。

### 6.命令模式

* 将方法的调用，请求，操作全部封装到一个单独的对象中
* 从执行的命令中分离出命令的责任，并把他委托给一个对象
* 会将行为与需求绑定在一起

``` javascript
(function(){
  var NumberCal = {
    add: function(a, b) {
      return a+b;
    },
    sub: function(a, b) {
      return a-b;
    }
  }
})()
```

这样可能会因为内部API的变化产生一些问题, 可能需要拓展一下让之更加方便

``` javascript
NumberCal.execute = function(name) {
  return NumberCal[name] && NumberCal[name].apply(NumberCal, [].slice.call(argument, 1));
}

NumberCal.execute("add", "1", "2");
NumberCal.execute("sub", "1", "2");
```

### 7.门面模式

