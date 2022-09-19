<!-- 针对于/sellerpage 这条路而显示出来的
在这个组件中，通过子组件注册的方式，要显示seller.vue这个组件
-->
<template>
   <div class="com-page">
    <seller/>
   </div>
</template>

<script>
import mySeller from '@/components/mySeller.vue'
export default {
  components: {
    seller: mySeller
  },
  data () {
    return {}
  },
  methods: {}
}
</script>

<style lang="" scoped>
</style>
