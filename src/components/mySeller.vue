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
      allDate: null // 服务器返回的数据
    }
  },
  mounted () {
    this.initChart()
    this.getData()
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
      this.updateChart()
    },
    // 更新图表
    updateChart () {
      // 获取表中数据
      const sellerNames = this.allDate.map((item) => {
        return item.name
      })
      const sellerValues = this.allDate.map((item) => {
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
    }
  }
}
</script>

<style lang='' scoped>
</style>
