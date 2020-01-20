<template>
  <el-row :gutter="10">
    <el-col :xs="24"
            :sm="15"
            style="position:relative">
      <el-amap ref="map"
               vid="amapDemo"
               :amap-manager="amapManager"
               :center="center"
               :zoom="zoom"
               :plugin="plugin"
               :events="events"
               class="map-container"
               :style="{height: height+'px'}">
        <el-amap-marker v-for="marker in markers"
                        :position="marker"></el-amap-marker>
      </el-amap>
      <el-amap-search-box class="search-box"
                          :search-option="searchOption"
                          :on-search-result="onSearchResult"></el-amap-search-box>
    </el-col>
    <el-col :xs="24"
            :sm="8">
      <el-card class="box-card"
               shadow="hover"
               :style="{height: height+'px'}">
        <el-row slot="header"
                type="flex"
                align="middle">
          <el-col>当前位置信息</el-col>
          <el-col style="text-align:right">
            <el-button size="small"
                       type="primary"
                       icon="el-icon-location"
                       @click="handleChoose">选择该位置</el-button>
          </el-col>
        </el-row>
        <el-row class="card-body">
          <el-row class="card-body-item">
            <el-col :span="5">经纬度：</el-col>
            <el-col :span="18">
              {{ positionResult ? positionResult.position.lat + ',' + positionResult.position.lng : '' }}
              <!-- {{ pos ? pos.position.lat + ',' + pos.position.lng : '' }} -->
            </el-col>
          </el-row>
          <el-row class="card-body-item">
            <el-col :span="5">地址：</el-col>
            <el-col :span="18">
              {{ positionResult ? positionResult.address : '' }}
            </el-col>
          </el-row>
        </el-row>
      </el-card>
    </el-col>
  </el-row>
</template>
<script>

// NPM 方式
import { AMapManager } from 'vue-amap';

let amapManager = new AMapManager();

export default {
  name: 'AMapPosition',
  props: {
    zoom: {
      type: Number,
      default: 12
    },
    height: {
      type: Number,
      default: 300
    },
    choosePosition: {
      type: Function,
      default: null
    },
  },
  created () {
    // console.log(this.positionResult);

    // if (this.positionResult) {
    //   this.center = [this.positionResult.position.lng, this.positionResult.position.lat]
    // }
  },
  data () {
    return {
      //AMAP地图
      amapManager,
      // zoom: 12,
      center: [115.3648660, 26.3318960],
      positionResult: null,
      events: {
        init: (map) => {

          let _this = this;

          //加载PositionPicker，loadUI的路径参数为模块名中 'ui/' 之后的部分
          AMapUI.loadUI(['misc/PositionPicker'], function (PositionPicker) {
            // 地图中心选点
            var positionPicker = new PositionPicker({
              mode: 'dragMap',//设定为拖拽地图模式，可选'dragMap'、'dragMarker'，默认为'dragMap'
              map: map, //依赖地图对象
              iconStyle: {//自定义外观
                url: '//webapi.amap.com/ui/1.0/assets/position-picker2.png',//图片地址
                size: [48, 48],  //要显示的点大小，将缩放图片
                ancher: [24, 40],//锚点的位置，即被size缩放之后，图片的什么位置作为选中的位置
              }
            });
            //TODO:事件绑定、结果处理等
            positionPicker.on('success', function (positionResult) {
              _this.positionResult = positionResult;
            });
            positionPicker.on('fail', function (positionResult) {
              // ...
            });
            positionPicker.start();

            //输入提示
            var autoOptions = {
              input: "tipinput"
            };
            var auto = new AMap.Autocomplete(autoOptions);
            var placeSearch = new AMap.PlaceSearch({
              map: map
            });  //构造地点查询类
            AMap.event.addListener(auto, "select", select);//注册监听，当选中某条记录时会触发
            function select (e) {
              placeSearch.setCity(e.poi.adcode);
              placeSearch.search(e.poi.name);  //关键字查询查询
            }
          });

        },
        'moveend': () => {
        },
        'zoomchange': () => {
        },
        'click': (e) => {
        }
      },
      searchOption: {},
      markers: [],
      plugin: ['ToolBar', 'Geolocation', 'Marker']
    };
  },
  methods: {
    getMap () {
      // amap vue component
      // console.log(amapManager._componentMap);
      // // gaode map instance
      // console.log(amapManager._map);
    },
    onSearchResult (pois) {
      let latSum = 0;
      let lngSum = 0;
      if (pois.length > 0) {
        pois.forEach(poi => {
          // console.log(poi);

          let { lng, lat } = poi;
          lngSum += lng;
          latSum += lat;
          this.markers.push([poi.lng, poi.lat]);
        });
        let center = {
          lng: lngSum / pois.length,
          lat: latSum / pois.length
        };
        this.center = [center.lng, center.lat];
      }
    },
    handleChoose () {
      this.$emit('choosePosition', this.positionResult)
    },
    setCenter (lng, lat) {
      this.center = [lng, lat];
    }
  }
};
</script>

<style>
.map-container {
  width: 100%;
  margin: 0 auto;
  position: relative;
}
.search-box {
  width: 200px !important;
  height: 35px !important;
  position: absolute !important;
  top: 10px;
  right: 15px;
}
.box-card {
  line-height: normal !important;
}
.card-header {
  /* display: flex;
  justify-content: space-between;
  align-items: center; */
}
.card-body {
}
.card-body-item {
  margin-bottom: 18px;
}
</style>
