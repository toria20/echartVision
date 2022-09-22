<!--商家销量统计的横向柱状图-->
<template>
   <div class="com-container">
    <div class="com-chart" ref="seller_ref"></div>
   </div>
</template>
<script>
export default {
  data () {
    return {
      chartInstance: null,
      allDate: null, // 服务器返回的数据
      currentPage: 1, // 当前页面
      totalPage: 0, // 一共的页面数
      timer: null // 定时器
    }
  },
  mounted () {
    this.initChart()
    this.getData()
    window.addEventListener('resize', this.screenAdapter)
    // 主动进行适配
    this.screenAdapter()
  },
  destroyed () {
    // 销毁定时器
    clearInterval(this.timer)
    // 销毁监听器
    window.removeEventListener('resize', this.screenAdapter)
  },
  methods: {
    // 初始化echart实例对象
    initChart () {
      this.chartInstance = this.$echarts.init(this.$refs.seller_ref, 'chalk')
      // 对图标初始化的控制
      const initOption = {
        title: {
          text: '‖ 商家销售统计',
          left: 20,
          top: 20
        },
        grid: {
          top: '20%',
          left: '3%',
          right: '6%',
          bottom: '3%',
          containLabel: true
        },
        xAxis: {
          type: 'value'
        },
        yAxis: {
          type: 'category'
        },
        tooltip: {
          trigger: 'axis',
          axisPointer: {
            type: 'line',
            z: 0,
            lineStyle: {
              color: '#2D3443'
            }
          }
        },
        series: [
          {
            type: 'bar',
            label: {
              show: true,
              position: 'right',
              textStyle: {
                color: 'white'
              }
            },
            itemStyle: {
              // 指明颜色渐变的方向
              // 指明不同百分比之下颜色的值
              color: new this.$echarts.graphic.LinearGradient(0, 0, 1, 0, [
                {
                  offset: 0,
                  color: '#5052EE'
                },
                // 百分之100状态下的颜色
                {
                  offset: 1,
                  color: '#AB6EE5'
                }
              ])
            }
          }
        ]
      }
      this.chartInstance.setOption(initOption)
    },
    // 获取服务器数据
    async getData () {
      // http:127.0.0.1:8888/api/seller
      const { data: ret } = await this.$http.get('seller')
      this.allDate = ret
      // 对数据进行排序
      this.allDate.sort((a, b) => {
        return a.value - b.value
      })
      // 每五个元素显示在一页
      this.totalPage = this.allDate.length % 5 === 0 ? this.allDate.length / 5 : this.allDate.length / 5 + 1
      this.updateChart()
      // 启动定时器
      this.startInterval()
    },
    // 更新图表
    updateChart () {
      const start = (this.currentPage - 1) * 5
      const end = this.currentPage * 5
      this.showDate = this.allDate.slice(start, end)
      // 获取表中数据
      const sellerNames = this.showDate.map((item) => {
        return item.name
      })
      const sellerValues = this.showDate.map((item) => {
        return item.value
      })
      const dataOption = {
        yAxis: {
          data: sellerNames
        },
        series: [
          {
            data: sellerValues
          }
        ]
      }
      this.chartInstance.setOption(dataOption)
    },
    startInterval () {
      if (this.timer) {
        clearInterval(this.timer)
      }
      this.timer = setInterval(() => {
        this.currentPage++
        if (this.currentPage > this.totalPage) {
          this.currentPage = 1
        }
        this.updateChart()
      }, 2000)
    },
    // 当浏览器窗口发生变化时，会调用方法来完成屏幕的适配
    screenAdapter () {
      const titleFontSize = this.$refs.seller_ref.offsetWidth / 100 * 3.5
      // 分辨率大小相关的配置
      const adapterOption = {
        title: {
          textStyle: {
            fontSize: titleFontSize
          }
        },
        tooltip: {
          axisPointer: {
            lineStyle: {
              width: titleFontSize,
              color: '#2D3443'
            }
          }
        },
        series: [
          {
            barWidth: titleFontSize,
            itemStyle: {
              barBorderRadius: [0, titleFontSize / 2, titleFontSize / 2, 0]
            }
          }
        ]
      }
      this.chartInstance.setOption(adapterOption)
      // 手动调用图标的resize方法才能有效果
      this.chartInstance.resize()
    }
  }
}
</script>

<style lang='' scoped>
</style>
