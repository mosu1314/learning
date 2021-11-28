组合模式将对象组合成树，以表示“部分-整体”的层次结构。除了用来表示树形结构之外，组合模式的另一个好处是通过对象的多态性表现，使得用户对单个对象和组合对象的使用具有一致性。

## 宏命令
```html
<html>
  <body>
    <button id="button">按我</button>
  </body>
  <script>
    var MacroCammand = function() {
      return {
        commandsList: [],
        add: function( command ) {
          this.commandList.push( command );
        },
        execute: function() {
          for ( var i = 0, command; command = this.commandList[ i++ ]; ) {
            command.execute();
          }
        }
      }
    }

    var OpenAcCommand = {
      execute: function() {
        console.log( '打开空调' );
      }
    }
/****** 家里的电视和音响是连接在一起的，所以可以用一个宏命令来组合打开电视和打开音响的命令 ******/

    var openTvCommand = {
      execute: function() {
        console.log( '打开电视' );
      }
    }

    var openSoundCommand = {
      execute: function() {
        console.log( '打开音响' );
      }
    }

    var macroCommand1 = MacroCommand();
    macroCommand1.add( openTvCommand );
    macroCommand1.add( openSoundCommand );

/***** 关门、打开电脑和打登录QQ命令 *****/

    var closeDoorCommand = {
      execute: function() {
        console.log( '关门' );
      }
    };

    var openPcCommand = {
      execute: function() {
        console.log( '打开电脑' );
      }
    };

    var openQQCommand = {
      execute: function() {
        console.log( '登录QQ' );
      }
    }

    var macroCommand2 = MacroCommand();
    macroCommand2.add( closeDoorCommand );
    macroCommand2.add( openPcCommand );
    macroCommand2.add( openQQCommand );

/***** 现在把所有的命令组合成一个”超级命令“ ******/

var macroCommand = MacroCommand();
macroCommand.add( openAcCommand );
macroCommand.add( macroCommand1 );
macroCommand.add( macroCommand2 );

/***** 最后给遥控器绑定”超级命令“ ******/
    var setCommand = (function() {
      document.getElementById( 'button' ).onclick = function() {
        command.execute();
      }
    })( macroCommand );
  </script>
</html>
```

## 组合模式例子 --- 扫描文件夹

```js
/***** Folder *****/
var Folder = function( name ) {
  this.name = name;
  this.files = [];
};

Folder.prototype.add = function( file ) {
  this.files.push( file );
};

Folder.prototype.scan = function() {
  console.log( '开始扫描文件夹：' + this.name );
  for ( var i = 0, file, files = this.files; file = files[ i++ ]; ) {
    file.scan();
  }
};

/***** File *****/
var File = function( name ) {
  this.name = name;
};

File.prototype.add = function() {
  throw new Error( '文件下面不能再添加文件' );
};

```

* 组合模式不是父子关系
  是一种HAS_A聚合关系，而不是IS-A
* 对叶对象操作的一致性
* 双向映射关系
* 用职责链模式提高组合模式性能

引用父对象
```js
var Folder = function( name ) {
  this.name = name;
  this.parent = null;
  this.files = [];
};

Folder.prototype.add = function( file ) {
  file.parent = this;
  this.files.push( file );
};

Folder.prototype.scan = function() {
  console.log( '开始扫描文件夹：' + this.name );
  for ( var i = 0, file, files = this.files; file = files[ i++ ] ) {
    file.scan();
  }
}

Folder.prototype.remove = function() {
  if ( !this.parent ) { // 根节点或者树外的游离节点
    return;
  }

  for ( var files = this.parent.files, l = files.length - 1; l >= 0; l-- ) {
    var file = files[ l ];
    if ( file = this ) {
      files.splice( l, 1 );
    }
  } 
};


//  File类的实现
var File = function( name ) {
  this.name = name;
  this.parent = null;
};

File.prototype.add = function() {
  throw new Error( '不能添加在文件夹下面' );
};

File.prototype.scan = function() {
  console.log( '开始扫描文件：' + this.name );
};

File.prototype.remove = function() {
  // 根节点或者树外的游离节点
  if ( !this.parent ) {
    return;
  }
  for ( var files = this.parent.files, l = files.length - 1; l >= 0; l-- ) {
    var file = files[ l ];
    if ( file === this ) {
      files.splice( l, 1 );
    }
  }
}

var folder = new Folder( '学习资料' );
var folder1 = new Folder( 'JavaScript' );
var file1 = new File( '深入浅出Node.js' );
folder1.add( new File( 'JavaScript设计模式与开发实践' ));
folder.add( folder1 );
folder.add( file1 );

folder.remove();  // 移除文件夹
folder.scan(); 
```


## 何时使用组合模式
* 表示对象的部分-整体层次结构
* 客户希望统一树中的所有的对象


