* ## 闭包 
  * 能够读取其他函数内部变量的函数，在js中函数内部中的子函数才能读取局部变量
  * 突破变量的作用域限制
  * 改变了变量的生存周期
* 封装变量
  ```js
  // 以缓存机制来提高这个函数的性能
  var cache = [];
  var mult = function() {
    var args = Array.prototype.join.call(arguments, ',');
    if ( cache[ args ]) {
      return cache[ args ];
    }
    var a = 1;
    for (var i = 0, l = arguments.length; i < l; i++) {
      a = a * arguments[i];
    }
    return cache[args] = a;
  }
  console.log( mult( 1, 2, 3 ) );
  console.log( mult( 1, 2, 3 ) );
  // cache暴露在全局
  var mult = (function() {
    var cache = {};
    return function() {
      var args = Array.prototype.join.call(arguments);
      if (args in cache) {
        return cache[ args ];
      }
      var a = 0;
      for(var i = 0, l = arguments.length; i < l; i++) {
        a = a * arguments[i];
      }
      return cache[args] = a;
    }
  })() 
  // 提炼函数是代码重构常见技巧，在一个大函数中有一些代码块能够独立出来，封装在独立的小函数里面
  var mult = (function() {
    var cache = {};
    var calculate = function() {
      var a = 1;
      for ( var i = 0, l = arguments.length; i < l; i++ ) {
        a = a * arguments[i];
      }
      return a;
    }
    return function() {
      var args = Array.prototype.join.call(arguments);
      if (args in cache) {
        return cache[args];
      }
      return cache[args] = calculate.apply(null, arguments);
    }
  })()
  ```
* 延续局部变量的寿命
* 过程与数据的结合是形容面向对象中的“对象”时经常使用的表达。对象以方法的形式包含了过程，而闭包则是在过程中以环境的形式包含了数据。通常用面向对象思想能够实现的功能，用闭包也能实现。
  ```js
    // 以闭包形式
    var extent = function() {
      var value = 0;
      return {
        call: function() {
          value++;
          console.log(value);
        }
      }
    };
    var extent = extent();
    extent.call(); // => 1
    extent.call(); // => 2
    extent.call(); // => 3
    // 换成对象写法
    var extent = {
      value: 0,
      call: function(){
        this.value++;
        console.log(this.value);
      }
    };
    extent.call(); // => 1
    extent.call(); // => 2
    extent.call(); // => 3
    // 或者原型写法
    var Extent = function() {
      this.value = 0;
    }; 
    extent.prototype.call = function() {
      this.value++;
      console.log( this.value );
    };
    var extent = new Extent();
    extent.call(); // => 1
    extent.call(); // => 2
    extent.call(); // => 3
  ``` 
* 用闭包实现命令模式
  ```js
  // 命令模式的意图是把请求封装为对象，从而分离请求的发起者和请求的接收者（执行者）之间的耦合关系
  // 在命令执行之前，可以预先往命令对象中植入命令的接收者
  // 对象形式实现命令模式
  var Tv = {
    open: function() {
      console.log( '打开电视机' );
    },
    close: function() {
      console.log( '关闭电视机' );
    }
  };

  var OpenTvCommand = function( receiver ) {
    this.receiver = receiver;
  };

  OpenTvCommand.prototype.execute = function() {
    this.receiver.open();
  };

  OpenTvCommand.prototype.undo = function() {
    this.receiver.close();
  };
  
  var set Command = function( command ) {
    document.getElementById( 'execute' ).onclick = function() {
      command.execute();
    };
    document.getElementById( 'undo' ).onclick = function() {
      command.undo();
    }
  }

  setCommand( new OpenTvCommand( Tv ) );
  // 闭包版的命令模式，命令接收者会被封闭在闭包形成的环境中
  var Tv = {
    open: function() {
      console.log( '打开电视机' );
    },
    close: function() {
      console.log( '关闭电视机' );
    }
  };

  var createCommand = function( receiver ) {

    var execute = function() {
      return receiver.open();
    };

    var undo = function() {
      return receiver.close();
    };

    return {
      execute: execute,
      undo. undo
    }
  };

  var setCommand = function( command ) {
    document.getElementId( 'execute' ).onclick = function() {
      command.execute();
    };
    document.getElementId( 'undo' ).onclick = function() {
      command.undo();
    }
  };

  setCommand( createCommand( Tv ) );
  ```

* 闭包与内存管理
  * 闭包会造成内存泄漏，因为垃圾收集机制采用的是引用计数策略


* 高阶函数
  * 函数可以作为参数被传递
  * 函数可以作为返回值输出

* 函数作为参数传递，这代表我们可以抽离出一部分变化的业务逻辑，把这部分业务逻辑放在函数参数中，这样一来可以分离业务代码中变化与不变的部分。其中一个重要应用场景就是常用的回调函数
* 回调函数
  ```js
  var getUserInfo = function( userId, callback ) {
    $.ajax( 'http://xxx.com/getuserInfo?' + userId, function( data ) {
      if ( typeof callback === 'function' ) {
        callback( data );
      }
    });
  }
  getUserInfo( 13157, function( data ) {
    console.log( data.userName );
  });

  // 一般写法
  var appendDiv = function() {
    for ( var i = 0; i < 100; i++) {
      var div = document.createElement( 'div' );
      div.innerHTML = i;
      document.body.appendChild( div );
      div.style.display = 'none';
    }
  };

  appendDiv();

  // 上面写法，appendDiv无法变通，成为一个难以复用的函数，并不是每次创建了节点之后就希望它们立刻被隐藏
  // 回调函数的写法
  var appendDiv = function( callback ) {
    for ( var i = 0; i < 100; i++) {
      var div = document.createElement( 'div' );
      div.innerHTML = i;
      document.body.appendChild( div );
      if ( typeof callback === 'function' ) {
        callback( div );
      }
    }
  };

  appendDiv(function( node ) {
    node.style.display = 'none';
  });

  // Array.prototype.sort
  // 接受一个函数当作参数
  // 对数组进行排序，这是不变的部分；而使用什么规则去排序，则是可变的部分，动静分离
  [1, 4, 3].sort( function( a, b ){
    return a - b;
  });
  // => [1, 3, 4]
  [1, 4, 3].sort( function( a, b ) {
    return b - a;
  });
  ```

* 函数作为返回值输出
  ```js
  // 判断数据的类型
  // 最直接是编写isType函数
  var isString = function( obj ) {
    return Object.prototype.toString.call( obj ) === '[object String]';
  };
  var isArray = function( obj ) {
    return Object.prototype.toString.call( obj ) === '[object Array]';
  };
  var isNumber = function( obj ) {
    return Object.prototype.toString.call( obj ) === '[object Number]';
  };
  // 字符串作为参数提前植入isType函数
  var isType = function( type ){
    return function( obj ) {
      return Object.prototype.toString.call( obj ) === '[object ' + type + ']';
    }
  };
  var isString = isType( 'String' );
  var isArray = isType( 'Array' );
  var isNumber = isType( 'Number' );
  // 可以使用循环语句来批量注册这些isType函数
  var Type = {};

  for ( var i = 0, type; type = [ 'String', 'Array', 'Number' ][ i++ ];) {
    (function(type) {
      Type[ 'is' + type ] = function( obj ) {
        return Object.prototype.toString.call( obj ) === '[object ' + type + ']';
      }
    })(type)
  }
  Type.isArray( [] );
  Type.isString( 'str' );
  // 一个单例模式的例子
  var getSingle = function( fn ) {
    var ret;
    return function() {
      return ret || ( ret = fn.apply( this, arguments ));
    }
  };
  // getSingle函数
  var getScript = getSingle(function() {
    return document.createElement( 'script' );
  });

  var script1 = getScript();
  var script2 = getScript();

  console.log( script1 === script2 ); // => true
  ```
* 面向函数实现AOP
  * AOP（面向切面的编程）的主要作用是把一些跟核心各业务逻辑无关的功能抽离出来，这些跟业务逻辑无关的功能通常包括日志统计、安全限制、异常处理等。把这些功能抽离出来之后，再通过“动态织入”的方式掺入业务逻辑模块中。这样做的好处首先是可以保持业务逻辑模块的纯净和高内聚性，其次也可以很方便地复用日志统计等功能模块。
  ```js
    // JavaScript中实现AOP，都是指“动态织入”到另外一个函数之中
    // Function.prototype扩展来做到
    Function.prototype.before = function( beforefn ){
      var _self = this;
      return function() {
        beforefn.apply( this, arguments );
        return _self.apply( this, arguments );
      }
    };

    Function.prototype.after = function( afterfn ) {
      var _self = this;
      return function() {
        var ret = _self.apply( this, arguments );
        afterfn.apply( this, arguments );
        return ret;
      }
    };

    var func = function() {
      console.log(2);
    };

    func = func.before(function(){
      console.log( 1 );
    }).after(function() {
      console.log( 3 );
    });

    func();
  ``` 
* currying 函数柯里化
  * **部分求值** 一个currying的函数首先会接受一些参数，接受了这些参数之后，该函数并不会立即求值，而是继续返回另外一个函数，刚才传入的参数在函数形成的闭包中被保存起来。待到函数被真的需要求值的时候，之前传入的所有参数都会被一次性用于求值。
  ```js
    // 基本需求，每次都需要计算
    var monthlyCost = 0; 
    var cost = function( money ) {
      monthlyCost += money;
    };

    cost( 100 );
    cost( 200 );
    cost( 300 );

    console.log( monthlyCost );

    // 部分currying
    var cost = (function() {
      var args = [];
      return function() {
        if ( arguments.length === 0) {
          var money = 0;
          for (var i = 0, l = args.length; i < l; i++) {
            money += args[ i ];
          }
          return money;
        } else {
          [].push.apply( args, arguments );
        }
      }
    })();

    cost( 100 );
    cost( 200 );
    cost( 300 );
    
    console.log( cost() );

    // currying
    var currying = function( fn ) {
      var args = [];

      return function() {
        if ( arguments.length === 0 ) {
          return fn( args );
        } else {
          [].push.apply( args, arguments );
          return arguments.callee;
        }
      }
    };

    var cost = (function() {
      var money = 0; 
      
      return function() {
        for ( var i = 0, l = arguments.length; i < l; i++ ) {
          money += arguments[i];
        }
        return money;
      }
    })();

    var cost = currying( cost );
    cost( 100 );
    cost( 200 );
    cost( 300 );
    console.log( cost() );
  ```
* uncurrying 借方法
  ```js
  Function.prototype.uncurrying = function() {
    var self = this;
    return function() {
      var obj = Array.prototype.shift.call( arguments );
      return self.apply(obj, arguments);
    }
  };

  var push = Array.prototype.push.uncurrying();
  (function(){
    push( arguments, 4);
    console.log( arguments ); // => [1, 2, 3, 4]
  })(1, 2, 3);

  // 把Array.prototype上的方法“复制”到array对象上
  for ( var i = 0; fn, ary = [ 'push', 'shift', 'forEach' ]; fn = ary[ i++ ];) {
    Array[fn] = Array.prototype[ fn ].uncurrying();
  }

  var obj = {
    'length': 3,
    '0': 1,
    '1': 2,
    '2': 3
  };

  Array.push( obj, 4 );
  console.log( obj.length );

  var first = Array.shift( obj );
  console.log( first );
  console.log( obj );

  Array.forEach( obj, function( i, n ){
    console.log( n );
  });

  // uncurrying 另一种实现
  Function.prototype.uncurrying  = function() {
    var self = this;
    return function() {
			// 巧妙在call和apply之间转换
      return Function.prototype.call.apply( self, arguments );
    }
  };
  ```
* 函数节流
  ```js
  var throttle = function ( fn, interval ) {
    var _self = fn,
        timer,
        firstTime = true;
    return function() {
      var args = arguments,
        _me = this;
      if ( firstTime ) {
        _self.apply(_me, args);
        return firstTime = false;
      }

      if (timer) {
        return false;
      }

      timer = setTimeout(function() {
        clearTimeout(timer);
        timer = null;
        _self.apply(_me, args);
      }, interval || 500);
    }
  };

  window.onresize = throttle(function() {
    console.log( 1 );
  }, 500);
  ```
* 分时函数
  ```js
  var timeChunk = function( ary, fn, count ) {
    
    var start = function() {
      for ( var i = 0; i < Math.min( count || 1, ary.length ); i++ ) {
        var obj = ary.shift();
        fn( obj );
      }
    };

    return function() {
      var t = setInterval(function() {
        if ( ary.length === 0 ) {
          start = null;
          return clearInterval( t );
        }
        start();
      }, 200);
    }
  };
  ```
* 惰性加载函数
  ```js
  // 第一个方案
  var addEvent = function( elem, type, handler ) {
    if ( window.addEventListener ) {
      return elem.addEventListener( type, handler, false);
    }
    if (window.attachEvent) {
      return elem.attachEvent( 'on' + type, handler );
    }
  }

  // 方案二，解决多次调用if条件分支
  var addEvent = (function() {
    if ( window.addEventLister ) {
      return function( elem, type, handler ) {
        elem.addEventListener( type, handler, false);
      }
    }
    if ( window.attachEvent ) {
      return function( elem, type, handler ) {
        elem.attachEvent( 'on' + type, handler );
      }
    }
  })();

  // 方案三，重写函数，解决没有使用addEvent情况下
  var addEvent = function( elem, type, handler) {
    if (window.addEventListener) {
      addEvent = function( elem, type, handler) {
        elem.addEventListener( type, handler, false);
      }
    } else if ( window.attachEvent ) {
      addEvent = function( elem, type, handler ) {
        elem.attachEvent( 'on' + type, handler );
      }
    }

    addEvent( elem, type, handler );
		// TODO 测试一下模块化，导出可会产生问题
  };
  ```