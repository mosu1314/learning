# 享元模式
一种用于性能优化的模式，使用空间换时间的优化方式，对已产生对象回收再利用。

# 核心
运用共享技术来有效支持大量颗粒度的对象

# 内部状态与外部状态
区分内部和外部状态：
* 内部状态储存与对象内部
* 内部状态可以被一些对象共享
* 内部状态独立于具体的场景，通常不会改变
* 外部状态取决于具体的场景，并根据场景而变化，外部状态不能被共享

# 通用结构
* 只有当某种共享对象被真正需要时，它才从工厂中被创建出来
* 可以用一个管理器来记录对象相关的外部状态，使这些外部状态通过某个钩子和共享对象联系起来

# 享元模式文件上传
```js
// 剥离外部状态
var Upload = function( uploadType ) {
  this.uploadType = uploadType;
};

Upload.prototype.delFile = function( id ) {
  uploadMananger.setExternalState( id, this );

  if ( this.fileSize < 3000 ) {
    return this.dom.parentNode.removeChild( this.dom );
  }

  if ( window.confirm( '确定要删除该文件吗？' + this.fileName ) ) {
    return this.dom.parentNode.removeChild( this.dom );
  }
}

// 工厂进行对象实例化
var UploadFactory = (function() {
  var createFlyWeightObjs = {};

  return {
    create: function( uploadType ) {
      if ( createdFlyWeightObjs[ uploadType ] ) {
        return createFlyWeightObjs[ uploadType ];
      }

      return createdFlyWeightObjs[ uploadType ] = new Upload( uploadType );
    }
  }
})();

// 管理器封装外部状态
var uploadManager = (function() {
  var uploadDatabase = {};

  return {
    add: function( id, uploadType, fileName, fileSize ) {
      var flyWeightObj = UploadFactory.create( uploadType );

      var dom = document.createElement( 'div' );

      dom.innerHTML = `<span>文件名：${fileName}，文件大小：${fileSize}</span>
        <button class="delFile">删除</button> 
      `;

      dom.querySelector( '.delFile' ).onclick = function() {
        filWeightObj.delFile( id );
      };

      document.body.appendChild( dom );

      uploadDatabase[ id ] = {
        fileName: fileName,
        fileSize: fileSize,
        dom: dom
      };

      return flyWeightObj;
    },
    setExternalState: function( id, flyWeightObj ) {
      var uploadData = uploadDatabase[ id ];
      for ( var i in uploadData ) {
        flyWeightObj[ i ] = uploadData[ i ];
      }
    }
  }
})();


var id = 0;

window.startUpload = function( upload, files ) {
  for ( var i = 0, file; file = files[ i++ ] ) {
    var uploadObj = uploadManager.add( ++id, uploadType, file.fileName, file.fileSize );
  }
};

startUpload( 'plugin', [
  {
    fileName: '1.txt',
    fileSize: 1000
  },
  {
    fileName: '2.html',
    fileSize: 3000
  },
  {
    fileName: '3.txt',
    fileSize: 5000
  }
]);

startUpload( 'flash', [
  {
    fileName: '4.txt',
    fileSize: 1000
  },
  {
    fileName: '5.html',
    fileSize: 3000
  },
  {
    fileName: '6.txt',
    fileSize: 5000
  }
]);

```

# 享元模式的适用性
* 一个程序中使用了大量的相似对象
* 由于使用了大量对象，造成很大的内存开销
* 对象的大多数状态都可以变为外部状态
* 剥离出对象的外部状态之后，可以用相对较少的共享对象取代大量对象


# 内部状态和外部状态极端情况
有多少中内部状态的组合，系统中便最多存在多少个共享对象，而外部状态储存在共享对象的外部，在必要时被传入共享来组装成一个完整的对象，考虑到两种极端的情况，即对象没有外部状态和没有内部状态的时候。

## 没有内部状态的享元
```js
// 生产共享对象的工厂变成了一个单例工厂
var uploadFactory = (function() {
  var uploadObj;
  return {
    create: function() {
      if ( uploadObj ) {
        return uploadObj;
      }
      return uploadObj = new Upload();
    }
  }
})();
```


## 没有外部状态的享元 -- 对象池

```js
var tooTipFactory = (function() {
  var toolTipPool = [];

  return {
    create: function() {
      if ( toolTipPool.length === 0 ) {
        var div = document.createElement( 'div' );
        document.body.appendChild( div );
        return div;
      } else {
        return toolTipPool.shift();
      }
    },
    recover: function( tooltipDom ) {
      return toolTipPool.push( tooltipDom );
    }
  }
})();

var ary = [];

for ( var i = 0, str; str = [ 'A', 'B' ][ i++ ] ) {
  var toolTip = toolTipFactory.create();
  toolTip.innerHTML = str;
  ary.push( toolTip );
}

for ( var i = 0, toolTip; toolTip = ary[ i++ ]; ) {
  toolTipFactory.recover( toolTip );
}

for ( var i = 0, str; str = [ 'A', 'B', 'C', 'D', 'E', 'F' ][ i++ ]; ) {
  var toolTip = toolTipFactory.create();
  toolTip.innnerHTML = str;
}

// 通用对象池实现
var objectPoolFactory = function( createObjFn ) {
  var objectPool = [];
  
  return {
    create: function() {
      var obj = objectPool.length === 0 ?  createObjFn.apply( this, arguments ) : objectPool.shift();

      return obj;
    },
    recover: function( obj ) {
      objectPool.push( obj );
    }
  }
};

var iframeFactory = objectPoolFactory( function() {
  var iframe = document.createElement( 'iframe' );
  document.body.appendChild( iframe );

  iframe.onload = function() {
    iframe.onload = null;
    iframeFactory.recover( iframe );
  }

  return iframe;
} );

var iframe1 = iframeFactory.create();
iframe1.src = 'http://baidu.com';

var iframe2 = iframeFactory.create();
iframe2.src = 'http://QQ.com';

setTimeout( function() {
  var iframe3 = iframeFactory.create();
  iframe3.src = 'http://163.com';
}, 3000 );
```
