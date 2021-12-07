# 模板方法模式
* 抽象父类
  * 子类的算法框架
  * 实现一些公共方法
  * 封装了子类中所有方法的执行顺序（重要指标）
* 实现子类
  * 继承了整个算法结构
  * 重写父类的方法


# Coffee or tea
```js
// 泡杯咖啡
var Coffee = function() {};

Coffee.prototype.boilWater = function() {
  console.log( '把水煮沸' );
};

Coffee.prototype.brewCoffeeGriend = function() {
  console.log( '用沸水冲泡咖啡' );
};

Coffee.prototype.pourInCup = function() {
  console.log( '把咖啡倒进杯子' );
};

Coffee.prototype.addSugarAndMilk = function() {
  console.log( '加糖和牛奶' );
};

Coffee.prototype.init = function() {
  this.boilWater();
  this.brewCoffeeGriends();
  this.pourInCup();
  this.addSugarAndMilk();
};

var coffee = new Coffee();
coffee.init();

// 泡一壶茶
var Tea = function() {};

Tea.prototype.boilWater = function() {
  console.log( '把水煮沸' );
};

Tea.prototype.steepTeaBag = function() {
  console.log( '用沸水浸泡茶叶' );
};

Tea.prototype.pourInCup = function() {
  console.log( '把茶水倒进被子' );
};

Tea.prototype.addLemon = function() {
  console.log( '加柠檬' );
};

Tea.prototype.init = function() {
  this.boilWater();
  this.steepTeaBag();
  this.pourInCup():
  this.addLemon():
};

var tea = new Tea();
tea.init();

// 分离共同点
// Beverage
var Beverage = function() {};

Beverage.prototype.boilWater = function() {
  console.log( '把水煮沸' );
};

Beverage.prototype.brew = function() {};

Beverage.prototype.pourInCup = function() {};

Beverage.prototype.addCondiments = function() {};

Beverage.prototype.init = function() {
  this.boilWater();
  this.brew();
  this.pourInCup();
  this.addCoundiments();
}


// Coffee子类
var Coffee = function() {};
Coffee.prototype = new Beverage();

Coffee.prototype.brew = function() {
  console.log( '用沸水冲泡咖啡' );
};

Coffee.prototype.pourInCup = function() {
  console.log( '把咖啡倒进杯子' );
};

Coffee.prototype.addCondiments = function() {
  console.log( '加糖和牛奶' );
};


// tea子类
var Tea = function() {};

Tea.prototype = new Beverage();

Tea.prototype.brew = function() {
  console.log( '用沸水浸泡茶叶' );
};

Tea.prototype.pourInCup = function() {
  console.log( '把茶倒进杯子' );
};

Tea.prototype.addCondiments = function() {
  console.log( '加柠檬' );
};
```


# 抽象类
只能用于继承，表示一种契约，为它的子类定义这些公共接口。

# JavaScript没有抽象类的缺点和解决方案
Todo 用鸭子类型来模拟接口检查<br />
使用方法时，抛出异常
```js
Beverage.prototype.brew = function() {
  throw new Error( '子类必须重写brew方法' );
};

Beverage.prototype.pourInCup = function() {
  throw new Error( '子类必须重写pourInCup方法' );
};

Beverage.prototype.addCondiments = function() {
  throw new Error( '子类必须重写addCondiments' );
};
// 简单易实现
// 缺点：错误信息滞后
```

# 钩子方法
父类默认实现接口

# 好莱坞原则
别调用我们，我们会调用你<br />
发布-订阅，回调函数使用到<br />
子类继承父类之后，父类通知子类执行这些方法，高层组件调用底层组件。


# 函数式实现模板方法模式
```js
var Beverage = function( param ) {

  var boilWater = function() {
    console.log( '把水煮沸' );
  };

  var brew = param.brew || function() {
    throw new Error( '必须传递brew方法' );
  };

  var pourInCup = params.pourInCup || function() {
    throw new Error( '必须传递pourInCup' );
  };

  var addCondiments = params.pourInCup || function() {
    throw new Error( '必须传递addCondiments方法' );
  };

  var F = function() {};

  F.prototype.init = function() {
    boilWater();
    brew();
    pourInCup();
    addCondiments();
  };

  return F;
};

var Coffee = Beverage( {
  brew: function() {
    console.log( '用沸水冲泡咖啡' );
  },
  pourInCup: function() {
    console.log( '把咖啡倒进杯子' );
  },
  addCondiments: function() {
    console.log( '加糖和牛奶' );
  }
} );

var Tea = Beverage({
  brew: function() {
    console.log( '用沸水浸泡茶叶' );
  },
  pourInCup: function() {
    console.log( '把茶杯倒进杯子' );
  },
  addCondiments: function() {
    console.log( '加柠檬' );
  }
});

var coffee = new Coffee();
coffee.init();

var tea = new Tea();
tea.init();
```
