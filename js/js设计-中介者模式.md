# 中介者模式
使网状的多对多关系变成了相对简单的一对多关系


# 泡泡堂游戏例子
```js
/*********** 玩家 ***********/
function Player( name, teamColor ) {
  this.name = name;
  this.teamColor = teamColor;
  this.state = 'alive';
}

Player.prototype.win = function() {
  console.log( this.name + ' won ' );
};

Player.prototype.lose = function() {
  console.log( this.name + ' lost ' );
};

/********* 玩家死亡 ***********/
Player.prototype.die = function() {
  this.state = 'dead';
  playerDirector.ReceiveMessage( 'playerDead', this );
};

/********* 移除玩家 ************/
Player.prototype.remove = function() {
  playerDirector.ReceiveMessage( 'removePlayer', this );
};

/********* 玩家换队 ************/
Player.prototype.changeTeam = function( color ) {
  playerDirector.ReceiveMessage( 'changeTeam', this, color );
};

var playerFactory = function( name, teamColor ) {
  var newPlayer = new Player( name, teamColor );
  playerDirector.ReceiveMessage( 'addPlayer', newPlayer );

  return newPlayer;
};

// 开放一些接收消息的接口
var playerDirector = ( function() {
  var players = {},
      operations = {};

  /*********** 新增一个玩家 *************/
  operations.addPlayer = function( player ) {
    var teamColor = player.teamColor;
    players[ teamColor ] = players[ teamColor ] || [];

    players[ teamColor ].push( player );
  };

  /********** 移动一个玩家 *************/
  operations.removePlayer = function( player ) {
    var teamColor = player.teamColor,
        teamPlayers = players[ teamColor ] || [];
    
    for ( var i = teamPlayers.length - 1; i >= 0; i-- ) {
      if ( teamPlayers[ i ] === player ) {
        teamPlayers.splice( i, 1 );
      }
    }
  }

  /********** 玩家换队 ****************/
  operations.changeTeam = function( player, newTeamColor ) {
    operations.removePlayer( player );
    player.teamColor = newTeamColor;
    operations.addPlayer( player );
  };

  operations.playerDead = function( player ) {
    var teamColor = player.teamColor,
        teamPlayers = players[ teamColor ];

    var all_dead = true;

    for ( var i = 0, player; player = teamPlayers[ i++ ]; ) {
      if ( player.state !== 'dead' ) {
        all_dead = true;
        break;
      }
    }

    if ( all_dead === true ) {
      for ( var i = 0, player; player = teamPlayers[ i++ ] ) {
        player.lose();
      }

      for ( var color in players ) {
        if ( color !== teamColor ) {
          var teamPlayers = players[ color ];
          for ( var i = 0, player; player = teamPlayers[ i++ ]; ) {
            player.win();
          }
        }
      }

    }
  };

  var ReceiveMessage = function() {
    var message = Array.prototype.shift.call( arguments );
    operations[ message ].apply( this, arguments );
  };

  return {
    ReceiveMessage: ReceiveMessage
  }
} )();

```

# 购买商品的例子
```js
var goods = {
  'red|32G': 3,
  'red|16G': 0,
  'blue|32G': 1,
  'blue|16G': 6
};

var mediator = ( function() {

  var colorSelect = document.getElementById( 'colorSelect' ),
      memorySelect = document.getElementById( 'memorySelect' ),
      numberInput = document.getElementById( 'numberInput' ),
      colorInfo = document.getElementById( 'colorInfo' ),
      memoryInfo = document.getElementById( 'memoryInfo' ),
      numberInfo = document.getElementById( 'nubmerInfo' ),
      nextBtn = document.getElementById( 'nextBtn' );
  
  return {
    changed: function( obj ) {
      var color = selectSelect.value,
          memory = memorySelect.value,
          number = numberInput.value,
          stock = goods[ color + '|' + memory ];
      
      if ( obj === colorSelect ) {
        colorInfo.innerHTML = color;
      } else if ( obj === memorySelect ) {
        numberInfo.innerHTML = number;
      } else if ( obj === numberInput ) {
        numberInfo.innerHTML = number;
      }

      if ( !color ) {
        nextBtn.disabled = true;
        nextBtn.innerHTML = '请选择手机颜色';
        return;
      }

      if ( !memory ) {
        nextBtn.disabled = true;
        nextBtn.innerHTML = '请选择内存大小';
        return;
      }

      if ( Number.isInterger( number - 0 ) && number > 0 ) {
        nextBtn.disabled = true;
        nextBtn.innerHTML = '请输入正确的购买数量';
        return;
      }

      nextBtn.disabled = false;
      nextBtn.innerHTML = '放入购物车';
    }
  }
} )();

// 事件函数
colorSelect.onchange = function() {
  mediator.changed( this );
};

memorySelect.onchange = function() {
  mediator.changed( this );
};

numberInput.onInput = function() {
  mediator.changed( this );
};

```

# 中介者缺点
系统中会新增一个中介者对象，因为对象之间交互的复杂性，转移成了中介者对象的复杂性，使得中介者对象经常是巨大的。中介者对象自身往往就是一个难以维护的对象。

