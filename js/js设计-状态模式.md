# 状态模式
**允许一个对象在其内部状态改变时改变它的行为，对象看起来似乎修改了它的类。**

事物内部状态的改变往往会带来事物的行为改变。

关键：是把事物的每种状态都封装成单独的类，跟此种状态有关的行为都被封装在这个类的内部。

# 状态模式的通用结构
```js
var Lignt = function() {
  this.offLightState = new OffLightState( this );
  this.weakLightState = new WeakLightState( this );
	this.strongLightState = new StrongLightState( this );
	this.superStrongLightState = new SuperStrongLightState( this );
};

Light.prototype.init = function() {
	var button = document.createElement( 'button' ),
	self = this;
	this.button = document.body.appendChild( button );
	this.button.innerHTML = '开关';
	this.currState = this.offLightState;

	this.button.onclick = function() {
		self.currState.buttonWasPressed();
	};
};

// 添加状态
var offLightState = function( light ) {
	this.light = light;
};

offLightState.prototype.buttonWasPressed = function() {
	console.log( '弱光' );
	this.light.setState( this.light.weakLightState );
};

// 缺少抽象类的变通方法
var State = function() {};

State.prototype.buttonWasPressed = function() {
	throw new Error( '父类的 buttonWasPressed 方法必须被重写' );
};

var SuperStrongLightState = function( light ) {
	this.light = light;
};

SuperStrongLightState.prototype = new State();

SuperStrongLightState.prototype.buttonWasPressed = function() {
	console.log( '关灯' );
	this.light.setState( this.light.offLightState );
};
```

# 另一个状态模式示例 --- 文件上传
```js
window.external.upload = function( state ) {
	console.log( state );
};

var plugin = (function() {
	var plugin = document.createElement( 'embed' );
	plugin.style.display = 'none';
	plugin.type = 'application/txftn-webkit';

	plugin.sign = function() {
		console.log( '开始文件扫描' );
	};

	plugin.pause = function() {
		console.log( '暂停文件上传' );
	};

	plugin.uploading = function() {
		console.log( '开始文件上传' );
	};

	plugin.del = function() {
		console.log( '删除文件上传' );
	};

	plugin.done = function() {
		console.log( '文件上传完成' );
	};
	
	document.body,appendChild( plugin );

	return plugin;
})();

var Upload = function( fileName ) {
	this.plugin = plugin;
	this.fileName = fileName;
	this.button1 = null;
	this.button2 = null;
	this.signState new SignState( this );
	this.uploadingState = new UploadingState( this );
	this.pauseState = new PauseState( this );
	this.doneState = new DoneState( this );
	this.errorState = new ErrorState( this );
	this.currState = this.signState;
};

Upload.prototype.init = function() {
	var that = this;
	
	this.dom = document.createElement( 'div' );
	this.dom.innerHTML = 
		'<span>文件名称：' + this.fileName + '</span>\
		<button data-action="button1">扫描中</button>\
		<button data-action="button2">删除</button>';
	
	document.body.appendChild( this.dom );

	this.button1 = this.dom.querySelector( '[data-action="button1"]' );
	this.button2 = this.dom.querySelector( '[data-action="button2"]' );

	this.bindEvent();
};

Upload.prototype.bindEvent = function() {
	var self = this;
	this.button1.onclick = function() {
		self.currState.clickHandler1();
	};
	this.button2.onclick = function() {
		self.currState.clickHandler2();
	};
};

Upload.prototype.sign = function() {
	this.plugin.sign();
	this.currState = tis.signState;
};


Upload.prototype.uploading = function() {
	this.button1.innnerHTML = '正在上传，点击暂停';
	this.plugin.uploading();
	this.currState = this.uploadingState;
};

Upload.prototype.pause = function() {
	this.button1.innerHTML = '已暂停，点击继续上传';
	this.plugin.pause();
	this.currState = this.pauseState;
};

Upload.prototype.done = function() {
	this.button1.innerHTML = '上传完成';
	this.plugin.done();
	this.currState = this.doneState;
};

Upload.prototype.error = function() {
	this.button1.innerHTML = '上传失败';
	this.currState = this.errorState;
};

Upload.prototype.del = function() {
	this.plugin.del();
	this.dom.parentNode.removeChild( this.dom );
};


var StateFactory = (function() {

	var state = function() {};

	State.prototype.clickHandler1 = function() {
		throw new Error( '子类必须重写父类的clickHandler1方法' );
	};

	State.prototype.clickHandler2 = function() {
		throw new Error( '子类必须重写父类的clickHandler2方法' );
	};

	return function( param ) {

		var F = function( uploadObj ) {
			this.uploadObj = uploadObj;
		};

		F.prototype = new State();

		for ( var i in param ) {
			F.prototype[ i ] = param[ i ];
		}

		return F;
	}
})();

var SignState = StateFactory({
	clickHandler1: function() {
		console.log( '扫描中，点击无效...' );
	},
	clickHandler2: function() {
		console.log( '文件正在上传，不能删除' );
	}
});

var UploadingState = StateFactory({
	clickHandler1: function() {
		this.uploadObj.pause();
	},
	clickHandler2: function() {
		console.log( '文件正在上传中，不能删除' );
	}
});

var PauseState = StateFactory({
	clickHandler1: function() {
		this.uploadObj.uploading();
	},
	clickHandler1: function() {
		this.uploadObj.del();
	}
});

var DoneState = StateFactory({
	clickHandler1: function() {
		console.log( '文件已完成上传，点击无效' );
	},
	clickHandler2: function() {
		this.uploadObj.del();
	}
});

var ErrorState = StateFactory({
	clickHandler1: function() {
		console.log( '文件上传失败，点击无效' );
	},
	clickHandler2: function() {
		this.uploadObj.del();
	}
});

// test
var uploadObj = new Upload( 'JavaScript 设计模式与开发实践' );
uploadObj.init();

window.external.upload = function( state ) {
	uploadObj[ state ]();
};

window.external.upload( 'sign' );

setTimeout( function() {
	window.external.upload( 'uploading' );
}, 1000 );

setTimeout( function() {
	window.external.upload( 'done' );
}, 5000 );

```