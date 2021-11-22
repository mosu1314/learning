* 策略模式定义：定义一系列的算法，把它们一个个封装起来，并且使它们可以互相替换。

```js
// 使用策略模式计算奖金
var calculateBonus = function( performanceLevel, salary ) {

  if ( performanceLevel === 'S' ) {
    return salary * 4;
  }

  if ( performanceLevel === 'A' ) {
    return salary * 3;
  }

  if ( performancelLevel === 'B' ) {
    return salary * 2;
  }
}

calculateBonus( 'B', 20000 );
calculateBonus( 'S', 6000 );

// 使用组合函数重构
var performanceS = function( salary ) {
  return salary * 4;
};

var performanceA = function( salary ) {
  return salary * 3;
};

var performanceB = function( salary ) {
  return salary * 2;
};

var calculateBonus = function( performanceLevel, salary ) {

  if ( performanceLevel === 'S' ) {
    return performanceS( salary );
  }

  if ( performanceLevel === 'A' ) {
    return performanceA( salary );
  }

  if ( performanceLevel === 'B' ) {
    return performanceB( salary );
  }
};

calculateBonus( 'A', 10000 );

// 使用策略模式重构代码
var PerformanceS = function() {};

PerformanceS.prototype.calculate = function( salary ) {
  return salary * 4;
};

var PerformanceA = function() {};

PerformanceA.prototype.calculate = function( salary ) {
  return salary * 3;
};

var PerformanceB = function() {};

PerformanceB.prototype.calculate = function( salary ) {
  return salary * 2;
};

// 接下来定义奖金类Bonus;
var Bonus = function() {
  this.salary = null;
  this.strategy = null;
};

Bonus.prototype.setSalary = function( salary ) {
  this.salary = salary;
};

Bonus.prototype.setStrategy = function( strategy ) {
  this.strategy = strategy;
};

Bonus.prototype.getBonus = function() {
  if ( !this.strategy ) {
    throw new Error( '未设置 strategy 属性' );
  }
  return this.strategy.calculate( this.salary );
}


// JavaScript版本的策略模式
var strategies = {
  'S': function( salary ) {
    return salary * 4;
  },
  'A': function( salary ) {
    return salary * 3;
  },
  'B': function( salary ) {
    return salary * 2;
  }
};

var calculateBonus = function( level, salary ) {
  return strategies[ level ]( salary );
};

console.log( calculateBonus( 'S', 20000 ) );
console.log( calculateBonus( 'A', 10000 ) );

// 多态在策略模式中的体现
var tween = {
  linear: function( t, b, c, d ) {
    return c * t / d + b;
  },
  easeIn: function( t, b, c, d ) {
    return c * ( t /= d ) * t + b;
  },
  strongEaseIn: function( t, b, c, d ) {
    return c * ( t /= d ) * t * t * t * t + b;
  },
  strongEaseOut: function( t, b, c, d ) {
    return c * ( ( t = t / d - 1 ) * t * t * t * t + 1 ) + b;
  },
  sineaseIn: function( t, b, c, d ) {
    return c * ( ( t = t / d - 1) * t * t + 1 ) + b;
  }
};

var Animate = function( dom ) {
  this.dom = dom;
  this.startTime = 0;
  this.startPos = 0;
  this.endPos = 0;
  this.propertyName = null;
  this.easing = null;
  this.duration = null;
}

Animate.prototype.start = function( propertyName, endPos, duration, easing) {
  // 动画启动时间
  this.startTime = +new Date();
  // dom 节点初始位置
  this.startPos = this.dom.getBoundingClientRect()[ propertyName ];
  // dom 节点需要被改变的CSS属性名
  this.propertyName = propertyName;
  // dom 节点目标位置
  this.endPos = endPos;
  // 动画持续时间
  this.duration = duration;
  // 缓动算法
  this.easing = tween[ easing ];

  var self = this;
  var timeId = setInterval(function() {
    if ( self.step() === false ) {
      clearInterval( timeId );
    }
  }, 19);
}

// propertyName：要改变的CSS属性名，比如 'left'、'top'，分别表示左右移动和上下移动
// endPos：小球运动的目标位置
// duration：动画持续时间
// easing：缓动算法

Animate.prototype.step = function() {
  var t = +new Date();
  if ( t >= this.startTime + this.duration ) {
    this.upate( this.endPos );
    return false;
  }
  var pos = this.easing(t - this.startTime, this.startPos, this.endPos - this.startPos, this.duration)
  this.update( pos );  
}

Animate.prototype.update = function( pos ) {
  this.dom.style[ this.propertyName ] = pos + 'px';
};

var div = document.getElementById( 'div' );
var animate = new Animate( div );

animate.start( 'left', 500, 1000, 'strongEaseOut' );


// 用策略模式重构表单校验

var strategies = {
  isNonEmpty: function( value, errorMsg ) {
    if ( value === '' ) {
      return errorMsg;
    }
  },
  minLength: function( value, length, errorMsg ) {
    if ( value.length < length ) {
      return errorMsg;
    }
  },
  isMobile: function( value, errorMsg ) {
    if ( !/(^1[3|5|8][0-9]{9}$)/.test( value )) {
      return errorMsg;
    }
  }
}

var Validator = function() {
  this.cache = [];
};

Validator.prototype.add = function( dom, rule, errorMsg ) {
  var ary = rule.split( ':' );
  this.cache.push(function() {
    var strategy = ary.shift();
    ary.unshift( dom.value );
    ary.push( errorMsg );
    return strategies[ strategy ].apply( dom, ary );
  });
};

Validator.prototype.start = function() {
  for ( var i = 0, validatorFunc; validatorFunc = this.cache[ i++ ]; ) {
    var msg = validatorFunc();
    if ( msg ) {
      return msg;
    }
  }
}


var validateFunc = function() {
  var validator = new Validator();

  validator.add( registerForm.userName, 'isNonEmpty', '用户名不能为空' );
  validator.add( registerForm.password, 'minLength:6', '密码长度不能少于6位');
  validator.add( registerForm.phoneNumber, 'isMobile', '手机号码格式不正确');

  var errorMsg = validator.start();
  return errorMsg;
};

var registerForm = document.geetElementById( 'registerForm' );
registerForm.onsubmit = function() {
  var errorMsg = validataFunc();
  if ( errorMsg ) {
    console.log( errorMsg );
    return false;
  }
};

// 多种校验
/************** 策略对象 ***************/
var strategies = {
  isNonEmpty: function( value, errorMsg ) {
    if ( value  === '' ) {
      return errorMsg;
    }
  },
  minLength: function( value, length, errorMsg ) {
    if ( value.length < length ) {
      return errorMsg;
    }
  },
  isMobile: function( value, errorMsg ) {
    if ( !/(^1[3|5|8][0-9]{9}$)/.test( value )) {
      return errorMsg;
    }
  }
}

/************** validator 类 *************/
var Validator = function() {
  this.cache = [];
};

Validator.prototype.add = function( dom, rules ) {
  var self = this;

  for ( var i = 0, rule; rule = rules[ i++ ]) {
    (function() {
      var strategyAry = rule.strategy.split( ':' );
      var errorMsg = rule.errorMsg;

      self.cache.push(function() {
        var strategy = strategyAry.shift(),
            value = dom.value;
        strategyAry.unshift( value );
        strategyAry.push( errorMsg );
        return strategies[ strategy ].apply( value, strategyAry );
      })
    })( rule );
  }
};

Validator.prototype.start = function() {
  for ( var i = 0, validatorFunc; validatorFunc = this.cache[ i++ ];) {
    var errorMsg = validatorFunc();
    if ( errorMsg ) {
      return errorMsg;
    }
  }
};

/**************** 客户调用代码 ******************/

var registerForm = document.getElementById( 'registerForm' );

var validataFunc = function() {
  var validator = new Validator();

  validator.add( registerForm.userName, [
    {
      strategy: 'isNonEmpty',
      errorMsg: '用户名不能为空'
    },
    {
      strategy: 'minLength:10',
      errorMsg: '用户名长度不能小于10位'
    }
  ])

  validator.add( registerForm.password, [{
    strategy: 'minLength:6',
    errorMsg: '密码长度不能小于6位'
  }]);

  validator.add( registerForm.phoneNumber, [{
    strategy: 'isMobile',
    errorMsg: '手机号码格式不正确'
  }]);

  var errorMsg = validator.start();

  return errorMsg;
}

registerForm.onsubmit = function() {
  var errorMsg = validataFunc();

  if ( errorMsg ) {
    console.log( errorMsg );
    return false;
  }
}

```

* 策略模式利用组合、委托和多态等技术和思想，可以有效地避免多重条件选择语句。
* 策略模式提供了对开放-封闭原则的完美支持，将算法封装在独立的strategy中，使得它们易于切换，易于理解，易于扩展
* 策略模糊中的算法也可以复用在系统的其他地方，从而避免许多重复的复制粘贴工作
* 在策略模式中利用组合和委托来让Context拥有算法的能力，这也是继承的一种更轻便的替代方案