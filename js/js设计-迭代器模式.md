迭代器模式：提供一个方法顺序访问一个聚合对象中的各个元素，而又不需要暴露该对象的内部表示。<br />
迭代器模式可以迭代的过程从业务逻辑中分离出俩，在使用迭代器模式之后，即使不关心对象的内部构造，也可以按顺序访问其中的每个元素。<br />

## 实现自己的迭代器
```js
var each = function( ary, callback ) {
  for ( var i = 0, l = ary.length; i < l; i++ ) {
    callback.call( ary[ i ], i, ary[ i ] );
  }
};

each( [ 1, 2, 3 ], function( i, n ) {
  console.log( [ i, n ] );
});
```

## 内部迭代器和外部迭代器
需求：判断2个数组里面元素的值是否完全相等

```js
// 内部迭代器
var compare = function( ary1, ary2 ) {
  if ( ary1.length !== ary2.length ) {
    throw new Error( 'ary1和ary2不相等' );
  }
  each( ary1, function( i, n ) {
    if ( n !== ary2[ i ] ) {
      throw new Error( 'ary1和ary2不相等' );
    }
  });
  console.log( 'ary1和ary2相等' );
};

compare( [ 1, 2, 3 ], [ 1, 2, 4 ] ); // throw new Error( 'ary1和ary2不相等' )

// 外部迭代器
var Iterator = function( obj ) {
  var current = 0;

  var next = function() {
    current += 1;
  }

  var isDone = function() {
    return current >= obj.length;
  }

  var getCurrItem = function() {
    return obj[ current ];
  }

  return {
    next: next,
    isDone: isDone,
    getCurrItem: getCurrItem,
    length: obj.length
  }
};

// 改写 compare 函数：
var compare = function( iterator1, iterator2 ) {
  if ( iterator1.length !== iterator2.length ) {
    console.log( 'iterator1和iterator2不相等' );
  }

  while( !iterator1.isDone() && !iterator2.isDone() ) {
    if ( iterator1.getCurrItem() !== iterator2.getCurrItem() ) {
      throw new Error ( 'iterator1和iterator不相等' );
    }
    iterator1.next();
    iterator2.next();
  }

  console.log( 'iterator1和iterator2相等' );
}
var iterator1 = Iterator( [ 1, 2, 3 ] );
var iterator2 = Iterator( [ 1, 2, 3 ] );

compare( iterator1, iterator2 ); // 输出：iterator1和iterator2相等

```


## 迭代类数组对象和字面量对象
```js
$.each = function( obj, callback ) {
  var value,
      i = 0,
      length = obj.length,
      isArray = isArrayLike( obj );

  if ( isArray ) {
    for ( ; i < length; i++ ) {
      value = callback.call( obj[ i ], i, obj[ i ]);

      if ( value === false ) {
        break;
      }

    }
  } else {
    for ( i in obj ) {
      value = callback.call( obj[ i ], i, obj[ i ]);
      if ( value === false ) {
        break;
      }
    }
  }
};
```

## 倒序迭代器
```js
var reverseEach = fucntion( ary, callback ) {
  for ( var l = ary.length - 1; l >= 0; l-- ) {
    callback( l, ary[ l ] );
  }
}

reverseEach( [ 0, 1, 2 ], function( i, n ) {
  console.log( n );
});
```

## 中止迭代器
```js
var each = function( ary, callback ) {
  for ( var i = 0, l = ary.length; i < l; i++ ) {
    if ( callback( i, ary[ i ] ) === false ) {
      break;
    }
  }
};

each( [ 1, 2, 3, 4, 5 ], function( i, n ) {
  if ( n > 3 ) {
    return false;
  }
  console.log( n );
} );
```

## 迭代器模式的应用举例
减少了分支语句的纠缠
```js
va getActiveUploadObj = function() {
  try {
    return new ActiveXObject( 'TXFTNActiveX.FTNUpload' ); // IE上传控件
  } catch ( e ) {
    return false;
  }
};

var getFlashUploadObj = function() {
  if ( supportFlash() ) {
    var str = '<object type="application/x-shockwave-flash"></object>';
    return $( str ).appendTo( $('body') );
  }
}

var getFormUploadObj = function() {
  var str = '<input name="file" type="file" class="ui-file" />';
  return $( str ).appendTo( $('body') );
}

var iteratorUploadObj = function() {
  for ( var i = 0, fn; fn = arguments[ i++ ]; ) {
    var uploadObj = fn();
    if ( uploadObj !== false ) {
      return uploadObj;
    }
  }
};

var uploadObj = iteratorUploadObj( getActiveUploadObj, getFlashUploadObj, getFormUploadObj );
```