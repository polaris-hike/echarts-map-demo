<template>
  <div class="echarts">
    <div style="width: 100; height: 100%" ref="sctterMap"></div>
    <div class="mapChoose">
      <span v-for="(item, index) in parentInfo" :key="item.code">
        <span class="title" @click="chooseArea(item, index)">{{
          item.cityName == "全国" ? "中国" : item.cityName
        }}</span>
        <span class="icon" v-show="index + 1 != parentInfo.length">></span>
      </span>
    </div>
  </div>
</template>

<script>
import echarts from "echarts";
import resize from "./mixins/resize";
export default {
  name: "sctterMap",
  mixins: [resize],
  data() {
    return {
      myCharts: null,
      geoJson: {
        features: [],
      },
      parentInfo: [
        {
          cityName: "全国",
          code: 100000,
        },
      ],
    };
  },
  mounted() {
    this.getGeoJson(100000);
  },
  methods: {
    getGeoJson(adcode) {
      let that = this;
      AMapUI.loadUI(["geo/DistrictExplorer"], (DistrictExplorer) => {
        var districtExplorer = new DistrictExplorer();
        districtExplorer.loadAreaNode(adcode, function (error, areaNode) {
          if (error) {
            console.error(error);
            return;
          }
          let Json = areaNode.getSubFeatures();
          if (Json.length > 0) {
            that.geoJson.features = Json;
          } else if (Json.length === 0) {
            that.geoJson.features = that.geoJson.features.filter(
              (item) => item.properties.adcode == adcode
            );
            if (that.geoJson.features.length === 0) return;
          }
          that.getMapData();
        });
      });
    },
    //获取数据
    getMapData() {
      let mapData = this.geoJson.features.map((item) => {
        return {
          name: item.properties.name,
          value: Math.random() * 1000,
          cityCode: item.properties.adcode,
        };
      });
      mapData = mapData.sort(function (a, b) {
        return b.value - a.value;
      });
      this.initEcharts(mapData);
    },
    initEcharts(mapData) {
      var min = mapData[mapData.length - 1].value;
      var max = mapData[0].value;
      if (mapData.length === 1) {
        min = 0;
      }
      this.myChart = echarts.init(this.$refs.sctterMap);
      echarts.registerMap("Map", this.geoJson); //注册
      this.myChart.setOption(
        {
          tooltip: {
            trigger: "item",
            formatter: (p) => {
              let val = p.value;
              if (!val) {
                val = 0;
              }
              let txtCon = p.name + ":" + val.toFixed(2);
              return txtCon;
            },
          },
          visualMap: {
            show: false,
            min: min,
            max: max,
            left: "3%",
            bottom: "5%",
            calculable: true,
            seriesIndex: [0],
            inRange: {
              color: ["#105389", "#3a8abc", "#0D96F1"],
            },
            textStyle: {
              color: "#24CFF4",
            },
          },
          series: [
            {
              name: "地图",
              type: "map",
              map: "Map",
              roam: true, //是否可缩放
              zoom: 1.1, //缩放比例
              data: mapData,
              label: {
                normal: {
                  show: true,
                  color: "rgb(249, 249, 249)", //省份标签字体颜色
                  formatter: (p) => {
                    switch (p.name) {
                      case "内蒙古自治区":
                        p.name = "内蒙古";
                        break;
                      case "西藏自治区":
                        p.name = "西藏";
                        break;
                      case "新疆维吾尔自治区":
                        p.name = "新疆";
                        break;
                      case "宁夏回族自治区":
                        p.name = "宁夏";
                        break;
                      case "广西壮族自治区":
                        p.name = "广西";
                        break;
                      case "香港特别行政区":
                        p.name = "香港";
                        break;
                      case "澳门特别行政区":
                        p.name = "澳门";
                        break;
                      default:
                        break;
                    }
                    return p.name;
                  },
                },
                emphasis: {
                  show: true,
                  color: "#f75a00",
                },
              },
              itemStyle: {
                normal: {
                  areaColor: "#24CFF4",
                  borderColor: "#53D9FF",
                  borderWidth: 1.3,
                  shadowBlur: 15,
                  shadowColor: "rgb(58,115,192)",
                  shadowOffsetX: 7,
                  shadowOffsetY: 6,
                },
                emphasis: {
                  areaColor: "#8dd7fc",
                  borderWidth: 1.6,
                  shadowBlur: 25,
                },
              },
            },
            {
              name: "散点",
              type: "effectScatter",
              coordinateSystem: "geo",
              rippleEffect: {
                brushType: "fill",
              },
              itemStyle: {
                normal: {
                  color: "#F4E925",
                  shadowBlur: 10,
                  shadowColor: "#333",
                },
              },
              data: [
                {
                  name: "北京市",
                  value: [116.4, 39.9, "200"],
                },
              ],
              // symbolSize: 8,
              symbolSize: function (val) {
                let value = val[2];
                if (value == max) {
                  return 27;
                }
                return 10;
              },
              showEffectOn: "render", //加载完毕显示特效
            },
          ],
        },
        true
      );
      let that = this;
      this.myChart.off("click");
      this.myChart.on("click", (params) => {
        if (
          that.parentInfo[that.parentInfo.length - 1].code ==
          params.data.cityCode
        ) {
          return;
        }
        let data = params.data;
        that.parentInfo.push({
          cityName: data.name,
          code: data.cityCode,
        });
        that.getGeoJson(data.cityCode);
      });
    },

    //选择切换市县
    chooseArea(val, index) {
      if (this.parentInfo.length === index + 1) {
        return;
      }
      this.parentInfo.splice(index + 1);
      this.getGeoJson(this.parentInfo[this.parentInfo.length - 1].code);
    },
  },
};
</script>
<style lang="scss" scoped>
.echarts {
  width: 100%;
  height: 100%;
  position: relative;
  background: url("../assets/bg1.jpg") no-repeat;
  background-size: 100% 100%;
}

.mapChoose {
  position: absolute;
  left: 20px;
  top: 55px;
  color: #eee;

  .title {
    padding: 5px;
    border-top: 1px solid rgba(147, 235, 248, 0.8);
    border-bottom: 1px solid rgba(147, 235, 248, 0.8);
    cursor: pointer;
  }

  .icon {
    font-family: "simsun";
    font-size: 25px;
    margin: 0 11px;
  }
}
</style>