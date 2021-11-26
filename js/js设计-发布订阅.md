发布-订阅模式又叫观察者模式，它定义对象间的一种一对多的依赖关系，当一个对象的状态发生改变时，所有依赖于它的对象都将得到通知。在JavaScript开发中，一般用事件模型来替代传统的发布-订阅。

## DOM事件
```js
// 订阅
document.body.addEventListener( 'click', function() {
  console.log( 2 );
}, false);
// 模拟用户点击 
// 发布
document.body.click();
```


## 发布订阅模式的通用实现
实现先订阅后发布
```js
// 发布-订阅的功能提取
var event = {
  clientList: [],
  listen: function( key, fn ) {
    if ( !this.clientList[ key ] ) {
      this.clientList[ key ] = [];
    }
    // 订阅的消息添加进缓存列表
    this.clientList[ key ] = [];
  },
  trigger: function() {
    var key = Array.prototype.shift.call( arguments ),
        fns = this.clientList[ key ];
    // 如果没有绑定对应的消息
    if ( !fns || fns.length === 0 ) {
      return false;
    }

    for ( var i = 0, fn; fn = fns[ i++ ]; ) {
      // arguments 是 trigger时带上的参数
      fn.apply( this, arguments );
    }
  }
};


// 取消订阅的事件
event.remove = function( key, fn ) {
  var fns = this.clientList[ key ];

  // 如果 key 对应的消息没有被人订阅，则直接返回
  if ( !fns ) {
    return false;
  }

  // 如果没有传入具体的回调函数，表示需要取消key对应消息的所有订阅
  if ( !fn ) {
    fns && ( fns.length = 0 );
  } else {
    for ( var l = fns.length - 1; l >= 0; l-- ) {
      var _fn = fns[ l ];
      if ( _fn === fn ) {
        // 删除订阅者的回调函数
        fns.splice( l, 1 );
      }
    }
  }
};


// 定义installEvent函数，这个函数可以给所有的对象都动态安装发布-订阅功能：
var installEvent = function( obj ) {
  for ( var i in event ) {
    obj[ i ] = event[ i ];
  }
};

```

## 真实例子 --- 网站登录
```js
$.ajax('http://xxx.com?login', function( data ) {
  login.trigger( 'loginSucc', data );
});

// 各个模块监听登录成功的消息
var header = (function() {
  login.listen( 'loginSucc', function( data ) {
    hreader.setAvatar( data.avatar );
  });
  return {
    setAvatar: function( data ) {
      console.log( '设置header模块的头像' );
    }
  }
})();

var nav = (function() {
  login.listen( 'loginSucc', function( data ) {
    nav.setAvatar( data.avatar );
  });

  return {
    setAvatar: function( avatar ) {
      console.log( '设置 nav模块的头像' );
    }
  }
})();

// 新加一个其他模块
var address = (function() {
  login.listen( 'loginSucc', function( obj ) {
    address.refresh( obj );
  });

  return {
    refresh: function( avatar ) {
      console.log( '刷新收货地址列表' );
    }
  }
})();
```


## 全局发布订阅
同样在程序中，发布-订阅模式可以用一个全局的 Event 对象来实现
```js
var Event = (function() {
  
  var clientList = [],
      listen,
      trigger,
      remove;

  listen = function( key, fn ) {
    if ( !clientList[ key ] ) {
      clientList[ key ] = [];
    }
    clientList[ key ].push( fn );
  };

  trigger = function() {
    var key = Array.prototype.shift.call( arguments ),
        fns = clientList[ key ];
    
    if ( !fns || fns.length === 0 ) {
      return false;
    }

    for ( var i = 0, fn; fn = fns[ i++ ] ) {
      fn.apply( this, arguments );
    }
  };

  remove = function( key, fn ) {
    var fns = clientList[ key ];
    if ( !fns ) {
      return false;
    }

    if ( !fn ) {
      fn && ( fns.length = 0 );
    } else {
      for ( var l = fns.length - 1; l >= 0; l-- ) {
        var _fn = fns[ l ];
        if ( _fn === fn ) {
          fns.splice( l, 1 );
        }
      }
    }
  }

  return {
    listen: listen,
    trigger: trigger,
    remove: remove
  }
})();

Event.listen( 'squareMeter88', function( price ) {
  console.log( '价格= ' + price );
});

Event.trigger( 'squareMeter88', 20000000);
```

## 模块间通信
模块之间如果用了太多的全局发布-订阅模式来通信，那么模块与模块之间的联系就被隐藏到了背后。我们最终会搞不清楚消息来自哪个模块，或者消息会流向那些模块，这又会给我们的维护带来一些麻烦，也许某个模块的作用就是暴露一些接口给其他模块调用。


## 先发布再订阅
```js
// 解决事件名冲突的情况

/************** 先发布后订阅 ************/

Event.trigger( 'click', 1 );

Event.listen( 'click', function( a ) {
  console.log( a );
});

/************** 使用命名空间 ************/
Event.create( 'namespace1' ).listen( 'click', function( a ) {
  console.log( a );  // => 1
});

Event.create( 'namespace1' ).trigger( 'click', 1 );

Event.create( 'namespace2' ).listen( 'click', function( a ) {
  console.log( a );  // => 2
});

Event.create( 'namespace2' ).trigger( 'click', 2 );

// 代码实现
var Event = (function() {

  var global = this,
      Event,
      _default = 'default';
  
  Event = function() {
    var _listen,
        _trigger,
        _remove,
        _slice = Array.prototype.slice,
        _shift = Array.prototype.shift,
        _unshift = Array.prototype.unshift,
        namespaceCache = {},
        _create,
        find,
        each = function( ary, fn ) {
          var ret;
          for ( var i = 0, l = ary.length; i < l; i++ ) {
            var n = ary[ i ];
            ret = fn.call( n, i, n );
          }

          return ret;
        };
    
    _listen = function( key, fn, cache ) {
      if ( !cache[ key ] ) {
        cache[ key ] = [];
      }
      cache[ key ].push( fn );
    };

    _trigger = function() {
      var cache = _shift.call( arguments );
          key = _shift.call( arguments ),
          args = arguments,
          _self = this,
          ret,
          stack = cache[ key ];

      if ( !stack || !stack.length ) {
        return;
      }

      return each( stack, function() {
        return this.apply( _self, args );
      })
    };

    _create = function( namespace ) {
      var namespace = namespace || _default;
      var cache = {},
          offlineStack = [],
          ret = {
            listen: function( key, fn, last ) {
              _listen( key, fn, cache );
              if ( offlineStack === null ) {
                return;
              }
              if ( last === 'last' ) {
                offlineStack.length && offlineStack.pop()();
              } else {
                each( offlineStack, function() {
                  this():
                });

                offlineStack = null;
              }
            },
            one: function( key, fn, last ) {
              _remove( key, cache );
              this.listen( key, fn, last );
            },
            remove: function( key, fn ) {
              _remove( key, cache, fn );
            },
            trigger: function() {
              var fn,
                  args,
                  _self = this;
                
              _unshift.call( arguments, cache );
              args = arguments;
              fn = function() {
                return _trigger.apply( _self, args );
              };

              if ( offlineStack ) {
                return offlineStack.push( fn );
              }

              return fn();
            }
          },
        return namespace ?
            ( namespaceCache[ namespace ] ? namespaceCache[ namespace ] : 
                  namespaceCache[ namespace ] = ret ) : ret;
      };
    return {
      create: _create,
      one: function( key, fn, last ) {
        var event = this.create();
        event.one( key, fn, last );
      },
      remove: function( key, fn ) {
        var event = this.create();
        event.remove( key, fn );
      },
      listen: function( key, fn, last ) {
        var event = this.create();
        event.listen( key, fn, last );
      },
      trigger: function() {
        var event = this.create();
        event.trigger.apply( this, arguments );
      }
    }
  }
  return Event;
})();
```

推模型是指事件发生时，发布者一次性把所有更改的状态和数据都推送给订阅者<br />
拉模型是发布者仅仅通知订阅者事件已经发生了，此外发布者要提供一些公开的接口供订阅者来主动拉取数据，好处是可以让订阅者“按需获取”，但同时又可能让发布者变成一个“门户大开”的对象，同时增加了代码量和复杂度

## 缺点
* 消耗一定的时间和内存
* 对象之间的必要联系，会导致程序难以跟踪维护和理解
* 多个发布者和订阅者嵌套在一起的时候，跟踪一个bug不是件轻松地事