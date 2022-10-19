<template>
   <div class="com-container" @dblclick="reverMap">
    <div class="com-chart" ref="map_ref"></div>
   </div>
</template>
<script>
import { mapState } from 'vuex'
import axios from 'axios'
import { getProvinceMapInfo } from '@/utils/map_utils'
// import china from '../../public/static/map/china.json'
// window.$echarts.registerMap('china', china)
export default {
  data () {
    return {
      chartInstance: null,
      allData: null,
      mapData: {} // 所获取的矢量数据
    }
  },
  created () {
    // 在组件创建完成之后 进行回调函数的注册
    this.$socket.registerCallBack('rankData', this.getData)
  },
  mounted () {
    this.initChart()
    // this.getData()
    this.$socket.send({
      action: 'getData',
      socketType: 'mapData',
      chartName: 'map',
      value: ''
    })
    window.addEventListener('resize', this.screenAdapter)
    this.screenAdapter()
  },
  destroyed () {
    window.removeEventListener('resize', this.screenAdapter)
    this.$socket.unRegisterCallBack('mapData')
  },
  methods: {
    async initChart () {
      this.chartInstance = this.$echarts.init(this.$refs.map_ref, this.theme)
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
    getData (ret) {
      // 获取服务器数据
      // const { data: ret } = await this.$http.get('map')
      this.allData = ret
      // console.log(this.allData)
      this.updateChart()
    },
    updateChart () {
      // 处理图表所需的数据
      // 图例数据
      const lengendArr = this.allData.map(item => {
        return item.name
      })
      const seriesArr = this.allData.map(item => {
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
  },
  computed: {
    ...mapState(['theme'])
  },
  watch: {
    theme () {
      console.log('主题切换了m')
      this.chartInstance.dispose() // 销毁当前的图表
      this.initChart() // 重新以最新的主题名称初始化图表对象
      this.screenAdapter() // 完成屏幕的适配
      this.updateChart() // 更新图表的展示
    }
  }
}
</script>
<style lang='less' scoped>
</style>
