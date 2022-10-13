<template>
   <div class="com-container">
    <div class="com-chart" ref="hot_ref"></div>
    <span class="iconfont arr-left" @click="toLeft" :style="comStyle">&#xe6ef;</span>
    <span class="iconfont arr-right" @click="toRight" :style="comStyle">&#xe6ed;</span>
    <span class="cat-name" :style="comStyle">{{catname}}</span>
   </div>
</template>
<script>
export default {
  data () {
    return {
      chartInstance: null,
      alldata: null,
      currentIndex: 0, // 当前所展示的一级分类数据
      titleFontSize: 0
    }
  },
  computed: {
    catname () {
      if (!this.alldata) {
        return ''
      } else {
        return this.alldata[this.currentIndex].name
      }
    },
    comStyle () {
      return {
        fontSize: this.titleFontSize + 'px'
      }
    }
  },
  mounted () {
    this.initchart()
    this.getData()
    window.addEventListener('resize', this.screenAdapter)
    this.screenAdapter()
  },
  destroyed () {
    window.removeEventListener('resize', this.screenAdapter)
  },
  methods: {
    initchart () {
      this.chartInstance = this.$echarts.init(this.$refs.hot_ref, 'chalk')
      const initOption = {
        title: {
          text: '‖ 热销商品的占比',
          left: 20,
          top: 20
        },
        legend: {
          top: '15%',
          icon: 'circle'
        },
        tooltip: {
          show: true,
          formatter: (arg) => {
            const thirdCategory = arg.data.children
            let total = 0
            thirdCategory.forEach(item => {
              total += item.value
            })
            let retStr = ''
            thirdCategory.forEach(item => {
              retStr += `
              ${item.name}:${parseFloat(item.value / total * 100).toFixed(2) + '%'}
              <br/>`
            })
            return retStr
          }
        },
        series: [
          {
            type: 'pie',
            label: {
              show: false
            },
            emphasis: {
              label: {
                show: true
              }
            }
          }
        ]
      }
      this.chartInstance.setOption(initOption)
    },
    async getData () {
      // 获取服务器的数据，对this.allData进行赋值以后，调用update方法更新图表
      const { data: ret } = await this.$http.get('hotproduct')
      this.alldata = ret
      // console.log(this.alldata)
      this.updateChart()
    },
    updateChart () {
      // 处理数据
      const legendData = this.alldata[this.currentIndex].children.map(item => {
        return item.name
      })
      const seriesData = this.alldata[this.currentIndex].children.map(item => {
        return {
          name: item.name,
          value: item.value,
          children: item.children
        }
      })
      const dataOption = {
        legend: {
          data: legendData
        },
        series: [
          {
            data: seriesData
          }
        ]
      }
      this.chartInstance.setOption(dataOption)
    },
    screenAdapter () {
      this.titleFontSize = this.$refs.hot_ref.offsetWidth / 100 * 3.6
      const adapterOption = {
        title: {
          textStyle: {
            fontSize: this.titleFontSize
          }
        },
        legend: {
          itemWidth: this.titleFontSize / 2,
          itemHeight: this.titleFontSize / 2,
          itemGap: this.titleFontSize / 2,
          textStyle: {
            FontSize: this.titleFontSize / 2
          }
        },
        series: [
          {
            radius: this.titleFontSize * 4.5,
            center: ['50%', '50%']
          }
        ]
      }
      this.chartInstance.setOption(adapterOption)
    },
    toLeft () {
      this.currentIndex--
      if (this.chartInstance < 0) {
        this.currentIndex = this.alldata.length - 1
      }
      this.updateChart()
    },
    toRight () {
      this.currentIndex++
      if (this.currentIndex > this.alldata.length - 1) {
        this.currentIndex = 0
      }
      this.updateChart()
    }
  }
}
</script>
<style lang='less' scoped>
.arr-left {
    position: absolute;
    left: 10%;
    top: 50%;
    transform: translateY(-50%);
    cursor:pointer;
    color: white;
}
.arr-right {
    position: absolute;
    right: 10%;
    top: 50%;
    transform: translateY(-50%);
    cursor:pointer;
    color: white;
}
.cat-name {
    position: absolute;
    left: 80%;
    bottom: 20%;
}
</style>
