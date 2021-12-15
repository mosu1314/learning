# 适配器模式
解决两个软件实体间的接口不兼容的问题


# 适配器应用
```js
var googleMap = {
  show: function() {
    console.log( '开始渲染谷歌地图' );
  }
};

var baiduMap = {
  display: function() {
    console.log( '开始渲染百度地图' );
  }
};

var baiduMapAdapter = {
  show: function() {
    return baiduMap.display();
  }
};

renderMap( googleMap );
renderMap( baiduMapAdapter );

// 第二种
var guangdongCity = {
  shenzhen: 11,
  guangzhou: 12,
  zhhai: 13
};

var getGuangdongCity = function() {
  var guangdongCity = [
    {
      name: 'shenzhen',
      id: 11
    },
    {
      name: 'guangzhou',
      id: 12
    }
  ] 
  return guangdongCity;
};

var render = function( fn ) {
  console.log( '开始渲染广东省地图' );
  document.write( JSON.stringify( fn() ) );
};

var addressAdapter = function( oldAddressfn ) {
  
  var address = {},
      oldAddress = oldAddressfn();

      for ( var i = 0, c; c = oldAddress[ i++ ]; ) {
        address[ c.name ] = c.id;
      }
  
  return function() {
    return address;
  }
};

render( addressAdapter( getGuangdongCity ) );
```
