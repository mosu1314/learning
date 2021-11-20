定义：保证一个类仅有一个实例，并提供一个访问它的全局访问点。

实现单例模式
```js
var Singleton = function( name ) {
  this.name = name;
};

Singleton.instance = null;
Singleton.prototype.getName = function() {
  console.log( this.name );
};

Singleton.getInstance = function( name ) {
  if ( !this.instance ) {
    this.instance = new Singleton( name );
  }
  return this.instance;
}

var a = Singleton.getInstance( 'sven1' );
var b = Singleton.getInstance( 'sven2' );

// 或者
var Singleton = function( name ) {
  this.name = name;
};

Singleton.prototype.getName = function() {
  console.log( this.name );
};

Singleton.getInstance = (function() {
  var instance = null;
  return function( name ) {
    if ( !instance ) {
      instance = new Singleton( name );
    }
    return instance;
  }
})();

// 证明是单例模式
var a = Singleton.getInstance( 'sven1' );
var b = Singleton.getInstance( 'sven2' );

console.log( a === b ); // => true
```

透明的单例模式：用户从这个类中创建对象的时候，可以像使用其他任何普通类一样
```js
var CreateDiv = (function() {
  var instance;
  
  var CreateDiv = function( html ) {
    if ( instance ) {
      return instance;
    }

    this.html = html;
    this.init();
    return instance = this;
  };

  CreateDiv.prototype.init = function() {
    var div = document.createElement( 'div' );
    div.innerHTML = this.html;
    document.body.appendChild( div );
  };

  return CreateDiv;
  
})();

var a = new CreateDiv( 'sven1' );
var b = new CreateDiv( 'sven2' );

console.log( a === b ); // => true
// 增加复杂度
// 不符合“单一职责原则"
```

使用代理实现单例模式
```js
var CreateDiv = function( html ) {
  this.html = html;
  this.init();
};

CreateDiv.prototype.init = function() {
  var div = document.createElement( 'div' );
  div.innerHTML = this.html;
  document.body.appendChild( div );
};

// 引入代理类 proxySingletonCreateDiv:
var ProxySingletonCreateDiv = (function() {

  var instance;
  return function( html ) {
    if ( !instance ){
      instance = new CreateDiv( html );
    }
    return instance;
  }
})();

var a = new ProxySingletonCreateDiv( 'sven1' );
var b = new ProxySingletonCreateDiv( 'sven2' );

console.log( a === b );

```

惰性单例：只有需要时才创建
```js
// 这段代码仍然违反单一职责原则，创建对象和管理单例的逻辑都放在createLoginLayer对象内部。
var CreateLoginLayer = (function() {
  var div;

  return function() {
    if ( !div ){
      div = document.createElement( 'div' );
      div.innerHTML = '我是登录浮窗';
      div.style.display = 'none';
      document.body.appendChild( div );
    }

    return div;
  }
})();

document.getElementById( 'loginBtn' ).onclick = function() {
  var loginLayer = createLoginLayer();
  loginLayer.style.display = 'block';
};


// 动静分离
var getSingle = function( fn ) {
  var result;

  return function() {
    return result || ( result = fn.apply( this, arguments ) );
  }
};

var createLoginLayer = function() {
  var div = document.createElement( 'div' );
  div.innerHTML = '我是登录浮窗';
  div.style.display = 'none';
  document.body.appendChild( div );
  return div;
};

var createSingleLoginLayer = getSingle( createLoginLayer );

document.getElementById( 'loginBtn' ).onclick = function() {
  var loginLayer = createSingleLoginLayer();
  loginLayer.style.display = 'block';
};

// 单例模式的用途不止创建对象
var bindEvent = getSingle(function() {
  document.getElementById( 'div1' ).addEventListener = function() {
    console.log( 'click' );
  }

  return true;
});

var render = function() {
  console.log( '开始渲染列表' );
  bindEvent();
};

render();
render();
render();

```