# 状态模式
**允许一个对象在其内部状态改变时改变它的行为，对象看起来似乎修改了它的类。**

事物内部状态的改变往往会带来事物的行为改变。

关键：是把事物的每种状态都封装成单独的类，跟此种状态有关的行为都被封装在这个类的内部。

# 状态模式的通用结构
```js
var Lignt = function() {
  this.offLightState = new OffLightState( this );
  this.weakLightState = new WeakLightState( this );
}
```