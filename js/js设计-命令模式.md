命令模式的由来，其实是回调函数的一个面向对象的替代品

应用场景：有时候需要向某些对象发送请求，但是并不知道请求的接收者是谁，也不知道被请求的操作是什么，此时希望用一种松耦合的方式来设计软件，以便解开按钮和负责具体行为对象之间的耦合。

```html
<body>
  <button id="button1">点击按钮1</button>
  <button id="button2">点击按钮2</button>
  <button id="button3">点击按钮3</button>
</body>
<script>
  var button1 = document.getElementById( 'botton1' );
  var button2 = document.getElementById( 'botton2' );
  var button3 = document.getElementById( 'botton3' );
</script> 
```
```js
// 中间人
var setCommand = function( botton, cammand ) {
  button.onclick = function() {
    command.execute();
  }
};

// 请求接收者
var MenuBar = {
  refresh: function() {
    console.log( '刷新菜单目录' );
  }
};

var SubMenu = {
  add: function() {
    console.log( '增加子菜单' );
  },
  del: function() {
    console.log( '删除子菜单' );
  }
};

// 命令类
var RefreshMenuBarCommand = function( receiver ) {
  this.receiver = receiver;
};

RefreshMenuBarCommand.prototype.execute = function() {
  this.receiver.refresh();
};

var AddSubMenuCommand = function( receiver ) {
  this.receiver = receiver;
};

AddSubMenuCommand.prototype.execute = function() {
  this.receiver.add();
};

var DelSubMenuCommand = function( receiver ) {
  this.receiver = receiver;
};

DelSubMenuCommand.prototype.execute = function() {
  console.log( '删除子菜单' );
};

// 把命令接收者传入到command对象中，并且把command对象安装到button上面：
var refreshMenuBarCommand = new RefreshMenuBarCommand( MenuBar );
var addSubMenuCommand = new AddSubMenuCommand( SubMenu );
var delSubMenuCommand = new DelSubMenuCommand( SubMenu );

setCommand( button1, refreshMenuBarCommand );
setCommand( button2, addSubMenuCommand );
setCommand( button3, delSubMenuCommand );
```

JavaScript中的命令模式
```js
var setCommand = function( button, func ) {
  button.onclick = function() {
    func();
  };
};

var MenuBar = {
  refresh: function() {
    console.log( '刷新菜单界面' );
  }
};

var RefreshMenuBarCommand = function( receiver ) {
  return function() {
    receiver.refresh();
  }
};

var refeshMenuBarCommand = RefreshMenuBarCommand( MenuBar );

setCommand( button1, refreshMenuBarCommand );

// 执行函数改为调用execute方法
var RefreshMenuBarCommand = function( receiver ) {
  return {
    execute: function() {
      receiver.refresh();
    }
  }
};

var setCommand = function( button, command ) {
  button.onclick = function() {
    command.execute();
  }
};

var refreshMenuBarCommand = RefreshMenuBarCommand( MenuBar );
setCommand( button1, refreshMenuBarCommand );

// 撤销命令
var ball = document.getElementById( 'ball' );
var pos = document.getElementById( 'pos' );
var moveBtn = document.getElementById( 'moveBtn' );
var cancelBtn = document.getElementById( 'cancelBtn' );

var MoveCommand = function( receiver, pos ) {
  this.receiver = receiver;
  this.pos = pos;
  this.oldPos = null;
};

MoveCommand.prototype.execute = function() {
  this.receiver.start( 'left', this.pos, 1000, 'strongEaseOut' );
  this.oldPos = this.receiver.dom.getRoundingClientRect()[ this.receiver.propertyName ];
  // 记录小球开始移动前的位置
};

MoveCommand.prototype.undo = function() {
  this.receiver.start( 'left', this.oldPos, 1000, 'strongEaseOut' );
  // 回到小球移动前记录的位置
}

var moveCommand;

moveBtn.onclick = function() {
  var animate = new Animate( ball );
  moveCommand = new MoveCommand( animate, pos.value );
  moveCommand.execute();
};

cancelBtn.onclick = function() {
  // 撤销命令
  moveCommand.undo();
}

```
```html
// 撤销和重做
<html>
  <body>
    <button id="replay" >播放录像</button>
  </body>
  <script>
    var Ryu = {
      attack: function() {
        console.log( '攻击' );
      },
      defense: function() {
        console.log( '防御' );
      },
      jump: function() {
        console.log( '跳跃' );
      },
      crouch: function() {
        console.log( '蹲下' );
      }
    };
    
    // 创建命令
    var makeCommand = function( receiver, state ) {
      return function() {
        receiver[ state ]();
      }
    };

    var commands = {
      '119': 'jump',
      '115': 'crounch',
      '97': 'defense',
      '100': 'attack'
    };

    // 保存命令的堆栈
    var commandStack = [];

    document.onkeypress = function( ev ) {
      var keyCode = ev.keyCode,
          command = makeCommand( Ryu, commands[ keyCode ]);
    };

    document.getElementById( 'replay' ).onclick = function() {
      var command;
      while( command = commandStack.shift()) {
        command();
      }
    };
  </script>
</html>
```

## 宏命令
```js
var closeDoorCommand = function() {
  execute: function() {
    console.log( '关门' );
  }
};

var openPcCommand = {
  execute: function() {
    console.log( '开电脑' );
  }
};

var openQQCommand = function() {
  execute: function() {
    console.log( '登录QQ' );
  }
};

var MacroCommand = function() {
  return {
    commandsList: [],
    add: function( command ) {
      this.commandsList.push( command );
    },
    execute: function() {
      for ( var i = 0, command; command = this.commandsList[ i++ ]; ) {
        command.execute();
      }
    }
  }
};

var macroCommand = MacroCommand();
macroCommand.add( closeDoorCommand );
macroCommand.add( openPcCommand );
macroCommand.add( openQQCommand );

macroCommand.execute();
```

命令模式在js中是一种隐形模式