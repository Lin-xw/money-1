<template>
  <div class="wrapper" ref="wrapper"></div>
</template>

<script lang="ts">
//引入
import {Component, Prop, Vue, Watch} from 'vue-property-decorator';
import echarts, {EChartOption, ECharts} from 'echarts';

@Component
export default class Chart extends Vue {
  //接受 Statistics.vue 的 options
  @Prop() options?: EChartOption;
  chart?: ECharts;

  //渲染
  mounted() {
    //传一个元素
    if (this.options === undefined) {
      return console.error('options 为空');
    }
    this.chart = echarts.init(this.$refs.wrapper as HTMLDivElement);
    this.chart.setOption(this.options);
  }
  @Watch('options')
  onOptionsChange(newValue: EChartOption) {
    this.chart!.setOption(newValue);
  }
}
</script>

<style scoped lang="scss">
.wrapper { //展示图表
  height: 400px;
}

</style>