# 职责链模式
使多个对象都有机会处理请求，从而避免请求的发送者和接收者之间的耦合关系。

# 灵活和拆分的职责链模式节点
```js
var order500 = function( orderType, pay, stock ) {
  if ( orderType === 1 && pay === true ) {
    console.log( '500元定金预购，得到100优惠券' );
  } else {
    return 'nextSuccessor';
  }
};

var order200 = function( orderType, pay, stock ) {
  if ( orderType === 2 && pay === true ) {
    console.log( '200 元定金预购，得到50优惠券' );
  } else {
    return 'nextSuccessor';
  }
};

var orderNormal = function( orderType, pay, stock ) {
  if ( stock > 0 ) {
    console.log( '普通购买，无优惠券' );
  } else {
    console.log( '手机库存不足' );
  }
};

var Chain = function( fn ) {
  this.fn = fn;
  this.successor = null;
};

Chain.prototype.setNextSuccessor = function( successor ) {
  return this.successor = successor;
};

Chain.prototype.passRequest = function() {
  var ret = this.fn.apply( this, arguments );
  if ( ret === 'nextSuccessor' ) {
    return this.successor && this.successor.passRequest.apply( this.successor, arguments );
  }
  return ret;
};

// 职责链的节点
var chainOrder500 = new Chain( order500 );
var chainOrder200 = new Chain( order200 );
var chainOrderNormal = new Chain( orderNormal );

// 指定节点在职责链中的顺序
chainOrder500.setNextSuccessor( chainOrder200 );
chainOrder200.setNextSuccessor( chainOrderNormal );

// 把请求传递给第一个节点
chainOrder500.passRequest( 1, true, 500 );
chainOrder500.passRequest( 2, true, 500 );
chainOrder500.passRequest( 3, true, 500 );
chainOrder500.passRequest( 1, false, 0 );
```

# 异步的职责链
```js
Chain.prototype.next = function() {
  return this.successor && this.successor.passRequest.apply( this.successor, arguments );
};

var fn1 = new Chain(function() {
  console.log( 1 );
  return 'nextSuccessor';
});

var fn2 = new Chain(function() {
  console.log( 2 );
  var self = this;
  setTimeout(function() {
    self.next();
  }, 1000);
});

var fn3 = new Chain(function() {
  console.log( 3 );
});

fn1.setNextSuccessor( fn2 ).setNextSuccessor( fn3 );
fn1.passRequest();
```

# 职责链模式的优缺点
优：
* 减少条件分支语句
* 节点对象可以灵活地拆分重组
* 可以手动指定起始节点，请求并不是非得从链中的第一个节点开始传递

缺：
* 不能保证某个请求一定会被链中的节点处理
* 可能在某次请求过程中，大部分节点并没有起到实质性的作用，从性能上考虑，避免过长的职责链


# 用AOP实现职责链
```js
Function.prototype.after = function( fn ) {
  var self = this;
  return function() {
    var ret = self.apply( this, arguments );
    if ( ret === 'nextSuccessor' ) {
      return fn.apply( this, arguments );
    }

    return ret;
  }
};

var order = order500yuan.after( order200yuan ).after( orderNormal );

order( 1, true, 500 );
order( 2, true, 500 );
order( 1, false, 500 );

// 用职责链模式获取文件上传对象
var getActiveUploadObj = function() {
  try { 
    return new ActiveXObject( "TXFTNActiveX.FTNUpload" );
  } catch( e ) {
    return 'nextSuccessor';
  }
};

var getFlashUploadObj = function() {
  if ( supportFlash() ) {
    var str = '<object type="application/x-shockwave-flash"></object>';
    return $( str ).appendTo( $('body' ) );
  }
  return 'nextSuccessor';
};

var getFormUploadObj = function() {
  return $( '<form><input name="file" type="file"/></form>' ).appendTo( $('body') );
};

var getUploadObj = getActiveUploadObj.after( getFlashUploadObj ).after( getFormUploadObj );

console.log( getUploadObj() );

```