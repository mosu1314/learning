# 装饰者模式
将一个对象嵌入另一个对象之中，实际上相当于这个对象被另一个对象包装起来，形成一条包装链。请求随着这条链依次传递到所有的对象，每个对象都有处理这条请求的机会。


# 模拟传统面向对象语言的装饰者模式
```js
var Plane = function() {};

Plane.prototype.fire = function() {
  console.log( '发射普通子弹' );
};


// 增加两个装饰类，分别是导弹和原子弹
var MissilbeDecorator = function( plane ) {
  this.plane = plane;
};

MissibleDecorator.prototype.fire = function() {
  this.plane.fire();
  console.log( '发射导弹' );
};

var AtomDecorator = function( plane ) {
  this.plane = plane;
};

AtomDecorator.prototype.fire = function() {
  this.plane.fire();
  console.log( '发射原子弹' );
};
```

# 回到JavaScript的装饰者
```js
var plane = {
  fire: function() {
    console.log( '发射普通子弹' );
  }
};

var missibleDecorator = function() {
  console.log( '发射导弹' );
};

var atomDecorator = function() {
  console.log( '发射原子弹' );
};

var fire1 = plane.fire;

plane.fire = function() {
  fire1();
  missibleDecorator();
};

var fire2 = plane.fire;

plane.fire = function() {
  fire2();
  atomDecorator();
}

plane.fire();

```

# 装饰者函数
```js
// 1
var a = function() {
  alert( 1 );
};

var _a = a;

a = function() {
  _a();
  alert( 2 );
};

a();

// 2
window.onload = function() {
  alert( 1 );
};


var _onload = window.onload || function() {};

window.onload = function() {
  _onload();
  alert( 2 );
};
// 必须维护_load这个中间变量，虽然看起来并不起眼，但如果函数的装饰链较长，或者需要装饰的函数变多，这些中间变量的数量也会越来越多。
// 会遇到this被劫持的问题
```

```html
<!-- 被劫持后修改 -->
<html>
  <button id="button"></button>
  <script>
    var _getElementById = document.getElementById;

    document.getElementById = function() {
      alert( 1 );
      return _getElementById.apply( document, arguments );
    }

    var button = document.getElementById( 'button' );
  </script>
</html>
```


# 用AOP装饰函数
```js
Function.prototype.before = function( beforeFn ) {
  var _self = this;
  return function() {
    beforeFn.apply( this, arguments );
    return _self.apply( this, arguments );
  }
};

Function.prototype.after = function( afterFn ) {
  var _self = this;
  return function() {
    var ret = _self.apply( this, arguments );
    afterFn.apply( this, arguments );
    return ret;
  }
};

// 不污染原型方式
var before = fucntion( fn, beforeFn ) {
  return function() {
    beforeFn.apply( this, arguments );
    return fn.apply( this, arguments );
  }
}

var a = before(
  function() { alert( 3 ) },
  function() { alert( 2 ) }
);

a = before( a, function() { alert( 1 ) } );
a();
```

# AOP的应用案例
把行为依照职责分成颗粒度更细的函数，随后通过装饰把它们合并到一起，这有助于我们编写一个松耦合和高复用性系统。

## 数据统计上报
```html
<html>
  <button tag="login" id="button">地啊你打开登录浮层</button>
  <script>
    Function.prototype.after = function( afterFn ) {
      var _self = this;
      return function() {
        var ret = _self.apply( this, arguments );
        afterFn.apply( this, arguments );
        return ret;
      }
    };

    var showLogin = function() {
      console.log( '打开登录浮层' );
    };

    var log = function() {
      console.log( '上报标签为：' + this.getAttribute( 'tag' ) );
    }

    showLogin = showLogin.after( log );

    document.getElementById( 'button' ).onclick = showLogin;
  </script>
</html>
```


## 用AOP动态改变函数的参数

```js
Function.prototype.before = function( beforeFn ) {
  var _self = this;
  return function() {
    beforeFn.apply( this, arguments );
    return _self.apply( this, arguments );
  }
};

var getToken = function() {
  return 'Token';
};

var ajax = function( type, url, param ) {
  console.log( param );
};

ajax = ajax.before( function( type, url, param ) {
  param.Token = getToken();
});
ajax( 'get', 'http://xxx.com/userinfo', { name: 'sven' } );

```

# 插入式的表单验证
```js
Function.prototype.before = function( beforeFn ) {
  var _self = this;
  return function() {
    if ( beforeFn.apply( this, arguments ) === false ) {
      return;
    }
    return _self.apply( this, arguments );
  }
};

var validata = function() {
  if ( username.value === '' ) {
    alert( '用户名不能为空' );
    return false;
  }
  if ( passsword.value === '' ) {
    alert( '密码不能为空' );
    return false;
  }
};

var formSubmit = function() {
  var param = {
    username: username.value,
    password: password.value
  };
  ajax( 'http://xxx.com/login', param );
};

formSubmit = formSubmit.before( validata );

submitBtn.onclick = function() {
  formSumbit();
};
```
被装饰者返回的是新的函数，原函数上保存了一些属性会丢失。


# 装饰者模式和代理模式

