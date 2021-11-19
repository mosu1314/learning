* 动态类型语言中实现一个原则：面向接口编程，而不是面向实现编程，也是鸭子类型（一个对象有效的语义，不是由继承自特定的类或实现特定的接口，而是由"当前方法和属性的集合"决定）
* 多态：统一操作作用于不同的对象上面，可以产生不同的解释和不同的执行结果。换句话说，给不同的对象发送同一个消息的时候，这些对象会根据这个消息分别给出不同的反馈。
* 多态思想背后是将”做什么“和”谁去做以及怎样去做“分离开来，也就是将“不变的事物”与“可能改变的事物”分离开来。
* 多态根本作用就是通过过程化的条件分支语句转化为对象的多态性
* 将行为分布在各个对象中，并让这些对象各自负责自己的行为，这正是面向对象设计的优点
* 封装的目的是信息隐藏，封装数据、实现、类型、变化
* 封装数据
  ```js
    var myObject = (function( {
      var __name = 'sven';
      return {
        getName: function() {
          return __name;
        }
      }
    }))();
    console.log(myObject.getName()); // => sven
    console.log(myObject.__name); //=> undefined
  ```
* 封装实现：对象对它自己的行为负责，对于其他对象或者用户都不关心它的内部实现
* 封装类型：js不区分类型时一种失色，也可以说是一种解脱
* 封装变化：把系统中稳定不变的部分和容易变化的部分隔离开来，保证程序的稳定性和可扩展性
* 原型模式，克隆是创建对象的手段
* 所有数据都是对象除了`undefined`
* 要得到一个对象，不是通过实例化类，而是找到一个对象作为原型并克隆它
* 理解`new`运算的工程
    ```js
    function Person(name) {
      this.name = name;
    }
    Person.prototype.getName = function() {
      return this.name;
    }

    var objectFactory = function() {
      var obj = new Object();,
          Constructor = [].shift.call(arguments);
      obj.__proto__ = Constructor.prototype;
      var ret = Constructor.apply(obj, arguments);
      return typeof === 'object' ? ret : obj;
    }
    var a = objectFactory(Person, 'sven');
    console.log(a.name);
    console.log(a.getName());
    console.log(Object.getPrototypeOf(a) === Person.prototype);

    var a = objectFactory(A, 'sven');
    var a = new A('sven');
    ```  
* 对象会记住它的原型
* `this`总是指向一个对象，具体是指向某个对象，在运行时基于函数的执行环境动态绑定的，而非函数被声明时的环境。
* 作为对象的方法的调用
  ```js
  var obj = {
    a: 1,
    getA: function() {
      alert( this == obj);
      alert( this.a );
    }
  };
  obj.getA();
  ```
* 作为普通函数调用
  ```js
    window.name = 'globalName';

    var getName = function() {
      return this.name;
    }
    console.log(getName());
    // 或者：
    window.name = 'globalName';

    var myObject = {
      name: 'sven',
      getName: function() {
        return this.name;
      }
    };

    var getName = myObject.getName;
    console.log(getName());
    // 或者：
    "use strict"

    window.name = 'globalName';
    var myObject = {
      name: 'sven',
      getName: function() {
        var callback = function() {
          console.log(this);
        }
        callback();
      }
    };
    myObject.getName();
  ```
* 构造器调用
  ```js
    var MyClass = function() {
      this.name = 'sven';
    }
    var obj = new MyClass();
    alert(obj.name);
    // 或者：
    var MyClass = function() {
      this.name = 'sven';
      return {
        name: name
      }
    }
    var obj = new MyClass();
    alert(obj.name);
    // 构造器不显式地返回任何数据，或者返回一个非对象类型的数据，就不会造成上述问题
    var MyClass = function () {
      this.name = 'sven';
      return 'anne';
    }
    var obj = new MyClass();
    alert(obj.name);
  ```
* Function.prototype.call 或 Function.prototype.apply
  ```js
    var obj1 = {
      name: 'sven',
      getName: function() {
        return this.name;
      }
    };

    var obj2 = {
      name: 'anne'
    };
    console.log(obj1.getName()); // => sven 
    console.log(obj1.getName.call( obj2)); // => anne
  ```

* `call`和`apply`是`Function`的原型定义的
* 改变this的指向
* 模拟`bind`的实现
  ```js
  Function.prototype.bind = function(context) {
    var self = this;
    return function() {
      return self.apply( context, arguments );
    }
  }
  var obj = {
    name: 'sven'
  };

  var func = function() {
    alert( this.name );
  }.bind( obj );

  func();

  // 加强版的
  Function.prototype.bind = function() {
    var self = this,
        context = [].shift.call(arguments),
        args = [].slice.call(arguments);
    return function() {
      return self.apply( context, [].concat.call(args, [].slice.call(arguments)));
    }
  }

  var obj = {
    name: 'sven'
  };

  var func = function( a, b, c, d) {
    alert( this.name );
    alert( [a, b, c, d ]);
  }.bind(obj, 1, 2);

  func(3, 4);
  ```
* ## 借用其他对象的方法
  ```js
  // 借用构造函数，实现类似继承的效果
  var A = function() {
    this.name = name;
  }
  var B = function() {
    A.apply(this, arguments);
  }
  B.prototype.getName = function() {
    return this.name;
  }
  var b = new B('sven');
  console.log(b.getName());   // => 'sven'
  //------------------------------------------
  (function() {
    Array.prototype.push.call(arguments, 3);
    console.log(arguments);   // => [1, 2, 3]
  })(1, 2);
  // 借用Array.prototype对象上的方法
  // 需要满足对象上条件和array类似
  // 对象可存取属性
  // length属性可读写
  ```