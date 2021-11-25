代理模式是为了一个对象提供一个代用品或占位符，以便控制对它的访问<br/>
代理的意义：
面向对象设计的原则---单一职责原则<br/>
单一职责原则指的是：<br/>
职责被定义为“引起变化的原因”。<br/>
将行为分布到细粒度的对象之中<br/>
如果一个对象承担的职责过多，等于把这些职责耦合到了一起，这种耦合会导致脆弱和低内聚的设计。当发生变化时，设计可能会遭到破坏。<br/>

虚拟代理：把一些开销很大的对象，延迟到真正需要它的时候才去创建
```js
var myImage = (function() {
  var imgNode = document.createElement( 'img' );
  document.body.appendChild( imgNode );

  return {
    setSrc: function( src ) {
      imgNode.src = src;
    }
  }
})();

var proxyImage = (function() {
  var img = new Image();
  img.onload = function() {
    myImage.setSrc( this.src );
  }

  return {
    setSrc: function( src ) {
      myImage.setSrc( 'file://user/loading.gif' );
      img.src = src;
    }
  }
})();

// 代理和本体接口的一致性
// 用户可以放心的请求代理，他值关心是否能得到想要的结果
// 在任何使用本体的地方都可以替换成使用代理
var myImage = (function() {
  var imgNode = document.createElement( 'img' );
  document.body.appendChild( imgNode );

  return function( src ) {
    imgNode.src = src;
  } 
})();

var proxyImage = (function() {
  var img = new Image();

  img.onload = function() {
    myImage( this.src );
  }

  return function( src ) {
    myImage( 'file://users/loading.gif' );
    img.src = src;
  }
})();

proxyImage( 'http://aasdf.jpg' );

// 虚拟代理合并请求
var synchronousFile = function( id ) {
  console.log( '开始同步文件，id为：' + id);
};

var proxySynchroonousFile = (function() {
  var cache = [],
      timer;

      return function( id ) {
        cache.push( id );
        if ( timer ) {
          return;
        }

        timer = setTimeout(function() {
          synchronousFile( cache.join());
          clearTimeout( timer );
          timer = null;
          cache.length = 0;
        }, 2000)
      }
})();

var checkbox = document.getElementByTagName( 'input' );

for ( var i = 0, c; c = checkbox[ i++ ];) {
  c.onclick = function() {
    if ( this.checked === true ) {
      proxySynchronousFile( this.id );
    }
  }
}

// 虚拟代理在惰性加载中的应用
var miniConsole = (function() {
  var cache = [];
  var handler = function( ev ) {
    if ( ev.keyCode === 123 ) {
      var script = document.createElement( 'script' );
      script.onload = function() {
        for ( var i = 0, fn; fn = cache[ i++ ]; ) {
          fn();
        }
      };
      script.src = 'miniConsole.js';
      document.getElementsByTagName( 'head' )[0].appendChild( script );
      document.body.removeEventListener( 'keydown', handler );

    }
  }

  document.body.addEventListener( 'keydown', handler, false );

  return {
    log: function() {
      var args = arguments;
      cache.push(function() {
        return miniConsole.log.apply( miniConsole, args);
      })
    }
  }
})();

miniConsole.log( 11 );   // 开始打印log

// miniConsole.js 代码

miniConsole = {
  log: function() {
    // 真正代码略
    console.log( Array.prototype.join.call( arguments ) );
  }
}
```

保护代理：用于控制不同权限的对象对目标对象的访问


缓存代理
```js
// 针对一些开销大的运算结果提供暂时的储存
var mult = function() {
  console.log( '开始计算乘积' );
  var a = 1;
  for ( var i = 0, l = arguments.length; i < l; i++ ) {
    a = a * arguments[i];
  }
  return a;
};

mult( 2, 3 );
mult( 2, 3, 4 );

// 加入缓存代理
var proxyMult = (function() {
  var cache = {};
  return fucntion() {
    var args = Array.prototype.join.call( arguments );
    if ( args in cache ) {
      return cahce[args];
    }

    return cache[ args ] = mult.apply( this, arguments );
  }
});

proxyMult( 1, 2, 3, 4 );
proxyMult( 1, 2, 3, 4 );
```

使用高阶函数动态创建代理
```js
/************ 计算乘积 *************/
var mult = function() {
  var a = 1;
  for ( var i = 0, l = arguments.length; i < 1; i++ ) {
    a = a * arguments[i];
  }
  return a;
}
/************ 计算加和 *************/
var plus = function() {
  var a = 0;
  for ( var i = 0, l = arguments.length; i < l; i++ ) {
    a = a + arguments[i];
  }

  return a;
}

/*********** 创建缓存代理的工厂 *************/
var createProxyFactory = function( fn ) {
  var cache = {};
  return function() {
    var args = Array.prototype.join.call( arguments );
    if ( args in cache ) {
      return cache[ args ];
    }
    return cache[ args ] = fn.apply( this, arguments );
  }
};

var proxyMult = createProxyFactory( mult ),
proxyPlus = createProxyFactory( plus );

console.log( proxyMult( 1, 2, 3, 4 ) );
console.log( proxyMult( 1, 2, 3, 4 ) );
console.log( proxyPlus( 1, 2, 3, 4 ) );
console.log( proxyPlus( 1, 2, 3, 4 ) );

```