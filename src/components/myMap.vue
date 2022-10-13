<template>
   <div class="com-container" @dblclick="reverMap">
    <div class="com-chart" ref="map_ref"></div>
   </div>
</template>
<script>
import axios from 'axios'
import { getProvinceMapInfo } from '@/utils/map_utils'
// import china from '../../public/static/map/china.json'
// window.$echarts.registerMap('china', china)
export default {
  data () {
    return {
      chartInstance: null,
      allDate: null,
      mapData: {} // 所获取的矢量数据
    }
  },
  mounted () {
    this.initChart()
    this.getData()
    window.addEventListener('resize', this.screenAdapter)
    this.screenAdapter()
  },
  destroyed () {
    window.removeEventListener('resize', this.screenAdapter)
  },
  methods: {
    async initChart () {
      this.chartInstance = this.$echarts.init(this.$refs.map_ref, 'chalk')
      // 获取中国地图的矢量数据
      // http://localhost:8080/static/map/china.json
      // 由于地图矢量图的数据并不是位于Koa2中，所以不能使用http
      const ret = await axios.get('http://localhost:8080/static/map/china.json')
      this.$echarts.registerMap('china', ret.data)
      const initOption = {
        title: {
          text: '‖ 商家分布',
          left: 20,
          top: 20
        },
        geo: {
          type: 'map',
          map: 'china',
          top: '5%',
          bottom: '5%',
          itemStyle: {
            areaColor: '#2E728F',
            borderColor: '#333'
          }
        },
        legend: {
          left: '5%',
          bottom: '5%',
          orient: 'vertical'
        }
      }
      this.chartInstance.setOption(initOption)
      this.chartInstance.on('click', async arg => {
        const provinceInfo = getProvinceMapInfo(arg.name)
        console.log(provinceInfo)
        // 获取这个省份的矢量数据
        if (!this.mapData[provinceInfo.key]) {
          const ret = await axios.get('http://localhost:8080' + provinceInfo.path)
          this.mapData[provinceInfo.key] = ret.data
          this.$echarts.registerMap(provinceInfo.key, ret.data)
        }
        const changeOption = {
          geo: {
            map: provinceInfo.key
          }
        }
        this.chartInstance.setOption(changeOption)
      })
    },
    async getData () {
      // 获取服务器数据
      const { data: ret } = await this.$http.get('map')
      this.allDate = ret
      // console.log(this.allDate)
      this.updateChart()
    },
    updateChart () {
      // 处理图表所需的数据
      // 图例数据
      const lengendArr = this.allDate.map(item => {
        return item.name
      })
      const seriesArr = this.allDate.map(item => {
        // return这个对象就代表的是一个类别下的所有散点数据
        // 如果想在地图中显示散点的数据，需要给图表添加一个数据 coordinateSytem:geo
        return {
          type: 'effectScatter',
          rippleEffect: {
            scale: 5,
            brushType: 'stroke'
          },
          name: item.name,
          data: item.children,
          coordinateSystem: 'geo'
        }
      })
      const dataOption = {
        legend: {
          data: lengendArr
        },
        series: seriesArr
      }
      this.chartInstance.setOption(dataOption)
    },
    screenAdapter () {
      const titleFontSize = this.$refs.map_ref.offsetWidth / 100 * 3.6
      const adapterOption = {
        title: {
          textStyle: {
            fontSuze: titleFontSize
          }
        },
        length: {
          itemWidth: titleFontSize / 2,
          itemHeight: titleFontSize / 2,
          itemGap: titleFontSize / 2,
          textStyle: {
            fontSize: titleFontSize / 2
          }
        }
      }
      this.chartInstance.setOption(adapterOption)
      this.chartInstance.resize()
    },
    // 回到中国地图
    reverMap () {
      const reverOption = {
        geo: {
          map: 'china'
        }
      }
      this.chartInstance.setOption(reverOption)
    }
  }
}
</script>
<style lang='less' scoped>
</style>
