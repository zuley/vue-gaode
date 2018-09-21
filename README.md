# 在Vue中完美的使用高德地图

> 最近项目中有使用到高德地图，搜了下发现饿了么的 vue-amap 比较知名。不过实际安装使用中发现还是有很多问题的，并不友好。不但要学习 amap 的文档，也还要学习原生高德API文档，还不如直接使用原生来的方便。

而这篇教程的目的就是，**怎么在vue中使用高德地图API**

文档：[Demo和使用文档](http://vue-gaode.rxshc.com)

## 运行项目
``` s
# 安装依赖
npm install
# 运行项目
npm run serve
```

## 实现思路

* 创建一个 mapDrag 的公共组件
* 在组件的 created 生命周期，载入高德地图API
* API载入完成即开始执行地图初始化
* 地图API暴露全局变量 window.AMap 可以直接使用
* 监听地图拖放事件，获得数据后通知自定义事件，对组件外部提供事件接口

## created 生命周期载入高德地图API

载入的方式类似于 jquery 的脚本加载，我这里也是使用了别人封装好的一个加载方法，各位直接使用或者自己改

``` javascript
async created () {
  // 已载入高德地图API，则直接初始化地图
  if (window.AMap && window.AMapUI) {
    this.initMap()
  // 未载入高德地图API，则先载入API再初始化
  } else {
    await remoteLoad(`http://webapi.amap.com/maps?v=1.3&key=${MapKey}`)
    await remoteLoad('http://webapi.amap.com/ui/1.0/main.js')
    this.initMap()
  }
}
```
## 初始化地图
在 methods 中创建一个 initMap 的方法供载入地图API之后调用。这里就可以使用任意高德API
``` javascript
initMap () {
  // 加载PositionPicker，loadUI的路径参数为模块名中 'ui/' 之后的部分
  let AMapUI = this.AMapUI = window.AMapUI
  let AMap = this.AMap = window.AMap
  AMapUI.loadUI(['misc/PositionPicker'], PositionPicker => {
    let mapConfig = {
      zoom: 16,
      cityName: MapCityName
    }
    if (this.lat && this.lng) {
      mapConfig.center = [this.lng, this.lat]
    }
    let map = new AMap.Map('js-container', mapConfig)
    // 加载地图搜索插件
    AMap.service('AMap.PlaceSearch', () => {
      this.placeSearch = new AMap.PlaceSearch({
        pageSize: 5,
        pageIndex: 1,
        citylimit: true,
        city: MapCityName,
        map: map,
        panel: 'js-result'
      })
    })
    // 启用工具条
    AMap.plugin(['AMap.ToolBar'], function () {
      map.addControl(new AMap.ToolBar({
        position: 'RB'
      }))
    })
    // 创建地图拖拽
    let positionPicker = new PositionPicker({
      mode: 'dragMap', // 设定为拖拽地图模式，可选'dragMap'、'dragMarker'，默认为'dragMap'
      map: map // 依赖地图对象
    })
    // 拖拽完成发送自定义 drag 事件
    positionPicker.on('success', positionResult => {
      // 过滤掉初始化地图后的第一次默认拖放
      if (!this.dragStatus) {
        this.dragStatus = true
      } else {
        this.$emit('drag', positionResult)
      }
    })
    // 启动拖放
    positionPicker.start()
  })
}
```

## 调用
``` html
<mapDrag @drag="dragMap" lat="22.574405" lng="114.095388"></mapDrag>
```
