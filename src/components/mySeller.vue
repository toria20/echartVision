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
  },
  destroyed () {
    // 销毁定时器
    clearInterval(this.timer)
  },
  methods: {
    // 初始化echart实例对象
    initChart () {
      this.chartInstance = this.$echarts.init(this.$refs.seller_ref)
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
      const option = {
        xAxis: {
          type: 'value'
        },
        yAxis: {
          type: 'category',
          data: sellerNames
        },
        series: [
          {
            type: 'bar',
            data: sellerValues
          }
        ]
      }
      this.chartInstance.setOption(option)
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
    }
  }
}
</script>

<style lang='' scoped>
</style>
