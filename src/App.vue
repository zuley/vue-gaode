<template>
  <div id="app">
    <div class="page-header">
      <h1>在Vue中完美的使用高德地图</h1>
      <p>在vue中使用高德地图API，全兼容。</p>
      <a href="https://github.com/zuley/vue-gaode" target="_blank" class="btn">View on GitHub</a>
      <a href="http://www.rxshc.com" target="_blank" class="btn">By 猪不乐意</a>
    </div>
    <div class="g-wraper" v-hljs>
      <div class="m-part">
        <h3 class="title">拖拽选址 - DEMO</h3>
        <mapDrag @drag="dragMap" class="mapbox"></mapDrag>
        <ul class="info">
          <li><span>经度：</span>{{ dragData.lng }}</li>
          <li><span>纬度：</span>{{ dragData.lat }}</li>
          <li><span>地址：</span>{{ dragData.address }}</li>
          <li><span>最近的路口：</span>{{ dragData.nearestJunction }}</li>
          <li><span>最近的路：</span>{{ dragData.nearestRoad }}</li>
          <li><span>最近的POI：</span>{{ dragData.nearestPOI }}</li>
        </ul>
      </div>
      <div class="m-part">
        <h3 class="title">教程说明</h3>
        <p>最近项目中有使用到高德地图，搜了下发现饿了么的 vue-amap 比较知名。不过实际安装使用中发现还是有很多问题的，并不友好。不但要学习 amap 的文档，也还要学习原生高德API文档，还不如直接使用原生来的方便。</p>
        <p>而这篇教程的目的就是，<b>怎么在vue中使用高德地图API</b></p>
        <p>源码：<a href="https://github.com/zuley/vue-gaode" target="_blank">View on GitHub</a>（为方便大家测试使用的是我本人的key，仅用于测试不要用于线上环境）</p>
      </div>
      <div class="m-part">
        <h3 class="title">实现思路</h3>
        <p>以一个拖放地图的例子来解释，DEMO在上方</p>
        <ol>
          <li>创建一个 mapDrag 的公共组件</li>
          <li>在组件的 created 生命周期，载入高德地图API</li>
          <li>API载入完成即开始执行地图初始化</li>
          <li>地图API暴露全局变量 window.AMap 可以直接使用</li>
          <li>监听地图拖放事件，获得数据后通知自定义事件，对组件外部提供事件接口</li>
        </ol>
      </div>
      <div class="m-part">
        <h3 class="title">created 生命周期载入高德地图API</h3>
        <p>载入的方式类似于 jquery 的脚本加载，我这里也是使用了别人封装好的一个加载方法，各位直接使用或者自己改</p>
        <pre>
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
}</pre>
      </div>
      <div class="m-part">
        <h3 class="title">初始化地图</h3>
        <p>在 methods 中创建一个 initMap 的方法供载入地图API之后调用。这里就可以使用任意高德API</p>
        <pre>
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
}</pre>
      </div>
      <div class="m-part">
        <h3 class="title">调用</h3>
        <pre>&lt;mapDrag @drag=&quot;dragMap&quot; lat=&quot;22.574405&quot; lng=&quot;114.095388&quot;&gt;&lt;/mapDrag&gt;</pre>
      </div>
    </div>
    <div class="m-footer">本教程由<a href="http://www.rxshc.com" target="_blank">猪不乐意</a>编写</div>
  </div>
</template>

<script>
import mapDrag from './components/mapDrag'
export default {
  name: 'app',
  components: {
    mapDrag
  },
  data () {
    return {
      dragData: {
        lng: null,
        lat: null,
        address: null,
        nearestJunction: null,
        nearestRoad: null,
        nearestPOI: null
      }
    }
  },
  methods: {
    dragMap (data) {
      this.dragData = {
        lng: data.position.lng,
        lat: data.position.lat,
        address: data.address,
        nearestJunction: data.nearestJunction,
        nearestRoad: data.nearestRoad,
        nearestPOI: data.nearestPOI
      }
    }
  }
}
</script>

<style>
body{ margin: 0; }
.page-header{
  color: #fff; text-align: center; background: #159957;
  background-image: -webkit-linear-gradient(330deg,#155799,#159957);
  background-image: linear-gradient(120deg,#155799,#159957);
  padding: 3rem 4rem; margin-bottom: 30px;
}
.page-header h1{ margin: 0; font-size: 40px; }
.page-header p{ color: #ccc; margin: 0; margin-bottom: 30px; }
.page-header a{ display: inline-block; border: 1px solid #fff; margin-right: 10px; line-height: 40px; padding: 0 20px; border-radius: 4px; color: #fff; text-decoration: none; transition: all .3s; }
.page-header a:hover{ background: #fff; color: #333; }
.g-wraper{ width: 1000px; margin: 0 auto; color: #666; font-size: 16px; line-height: 30px; }
.m-part{ margin-bottom: 30px; }
.m-part::after{ content: ''; display: block; clear: both; }
.m-part .title{ font-size: 30px; line-height: 60px; margin-bottom: 10px; color: #333; }
.m-part .mapbox{ width: 600px; height: 400px; margin-bottom: 20px; float: left; }
.m-part .info{ margin: 0; padding: 0; list-style: none; line-height: 30px; margin-left: 620px; }
.m-part .info span{ display: block; color: #999; }
.m-part ol{ line-height: 40px; margin-left: 0; padding-left: 0; }
.m-part pre{ padding: 10px 20px; line-height: 30px; border-radius: 3px; box-shadow: 0 0 15px rgba(0,0,0,.5); }
.m-footer{ background: #eee; line-height: 60px; text-align: center; color: #999; font-size: 12px; }
.m-footer a{ margin:  0 5px; color: #999; text-decoration: none; }
</style>
