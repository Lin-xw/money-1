<template>
  <div class="wrapper" ref="wrapper"></div>
</template>

<script lang="ts">
//引入
import {Component, Prop, Vue} from 'vue-property-decorator';
import echarts, {EChartOption} from 'echarts';

@Component
export default class Chart extends Vue {
  //接受 Statistics.vue 的 options
  @Prop() options?: EChartOption;

  //渲染
  mounted() {
    //传一个元素
    const chart =  echarts.init(this.$refs.wrapper as HTMLDivElement)
    chart.setOption(this.options!)
  }
}
</script>

<style scoped lang="scss">
.wrapper { //展示图表
  height: 400px;
}

</style>