<template>
  <Layout class="layout">
    <div class="chart-wrapper" ref="chartWrapper">
    <Chart class="chart" :options="x"/>
    </div>
    <Tabs class-prefix="type" :data-source="recordTypeList" :value.sync="type"/>
    <ol v-if="groupedList.length>0">
      <li v-for="(group, index) in groupedList" :key="index">
        <h3 class="title">{{ beautify(group.title) }} <span>￥{{ group.total }}</span></h3>
        <ol>
          <li v-for="item in group.items" :key="item.id"
              class="record"
          >
            <span>{{ tagString(item.tags) }}</span>
            <span class="notes">{{ item.notes }}</span>
            <span>￥{{ item.amount }} </span>
          </li>
        </ol>
      </li>
    </ol>
    <div v-else class="noResult">
      当前没有收支记录
    </div>
  </Layout>
</template>
<script lang="ts">
//引入
import Vue from 'vue';
import {Component} from 'vue-property-decorator';
import Tabs from '@/components/Tabs.vue';
import recordTypeList from '@/constants/recordTypeList';
import dayjs from 'dayjs';
import clone from '@/lib/clone';
import Chart from '@/components/Chart.vue'
import _ from 'lodash';
import day from 'dayjs';

//放入组件里
@Component({
  components: {Tabs, Chart},
})
export default class Statistics extends Vue {
  tagString(tags: Tag[]) {
    return tags.length === 0 ? '无' : tags.map(t => t.name).join('，');
  }
  //让图表滚轮初始在最尾端
  mounted(){
    const div = (this.$refs.chartWrapper as HTMLDivElement);
    div.scrollLeft = div.scrollWidth;
  }


  beautify(string: string) {
    const day = dayjs(string);
    const now = dayjs();
    if (day.isSame(now, 'day')) {
      return '今天';
    } else if (day.isSame(now.subtract(1, 'day'), 'day')) {
      console.log('hi');
      return '昨天';
    } else if (day.isSame(now.subtract(2, 'day'), 'day')) {
      return '前天';
    } else if (day.isSame(now, 'year')) {
      return day.format('M月D日');
    } else {
      return day.format('YYYY年M月D日');
    }
  }

  //声明 y
  get y() {
    //当前最新一天，日期
    const today = new Date();
    //用于装数组

    const array = [];
    //首先获取30个日期
    //第一天属于0天，所以不减天数
    //然后找到这30天中有数据的createdAt
    for (let i = 0; i <= 29; i++) {
      //date是个字符串
      //每个日期用当前的日期减去i天，然后把它变成'YYYY-MM-DD'模式
      const dateString = day(today)
          .subtract(i,'day').format('YYYY-MM-DD');
      //得到deteString之后，找到这一天对应的金额
      const found = __.find(this.recordList, {
        createdAt: dateString
      });
      //找到后把时间存到数组的date里面，然后金额存在的话就获取amount，不存在就为0
      array.push({
        date: dateString, value: found ?.amount
      });
    }
    //push完后对数据进行排序
    array.sort((a,b)=>{
      //如果第一项的date大于第二项的date就返回 1，如果他们相等就返回 0，再其他的就返回 -1
      if(a.date > b.date){
        return 1;
      } else if (a.date === b.date){
        return 0;
      }else {
        return -1;
      }
    });
    return array;
  }

  //声明 x
  get x() {
    //keys就是所有的日期
    const keys = this.y.map(item=>item.date)
    //values就是所有日期的金额
    const values = this.y.map(item => item.value);
    return {
      //解决图表两边多余空白
      grid: {
        left: 0,
        right: 0,
      },
      xAxis: {
        type: 'category',
        data: keys,
        axisTick: {alignWithLabel: true},
        axisLine: {lineStyle: {color: '#666'}},
        axisLabel: {
          formatter: function (value: string, index: number){
            //去除前五个字符
            return value.substr(5);
          }
        }
      },
      yAxis: {
        type: 'value',
        show: false
      },
      tooltip:{
        show: true,
        position: 'top',
        triggerOn: 'click',
        formatter: '{c}'
      },
      series: [{
        symbol: 'circle',
        symbolSize: 12,
        itemStyle: {borderWidth: 1, color: '#666', borderColor: '#666'},
        data: values,
        type: 'line'
      }],
    }
  }

  get recordList() {
    return (this.$store.state as RootState).recordList;
  }

  get groupedList() {
    const {recordList} = this;

    const newList = clone(recordList)
        //createdAt of undefined 的解决办法
        //增加无收入也可以点击进
        .filter(r => r.type === this.type)
        .sort((a, b) => dayjs(b.createdAt).valueOf() - dayjs(a.createdAt).valueOf());
    if (newList.length === 0) {
      return [] as Result;
    }
    type Result = { title: string, total?: number, items: RecordItem[] }[]
    const result: Result = [{title: dayjs(newList[0].createdAt).format('YYYY-MM-DD'), items: [newList[0]]}];
    for (let i = 1; i < newList.length; i++) {
      const current = newList[i];
      const last = result[result.length - 1];
      if (dayjs(last.title).isSame(dayjs(current.createdAt), 'day')) {
        last.items.push(current);
      } else {
        result.push({title: dayjs(current.createdAt).format('YYYY-MM-DD'), items: [current]});
      }
    }
    result.map(group => {
      group.total = group.items.reduce((sum, item) => {
        return sum + item.amount;
      }, 0);
    });
    return result;
  }

  beforeCreate() {
    this.$store.commit('fetchRecords');
  }

  type = '-';
  recordTypeList = recordTypeList;
}
</script>

<style scoped lang="scss">
.echarts {
  max-width:100%;
  height: 400px;
}


.layout {
  display: flex;
  flex-direction: column;
  height: 100vh;
}

.noResult {
  padding: 16px;
  text-align: center;
}

::v-deep {

  .type-tabs-item {
    background: #e9f0e8;
    color: #696969;

    &.selected {
      background: #93b68d;
      color: white;

      &::after {
        display: none;
      }
    }
  }

  .interval-tabs-item {

    height: 48px;
  }
}

%item {
  padding: 8px 16px;
  line-height: 24px;
  display: flex;
  justify-content: space-between;
  align-content: center;
}

.title {
  @extend %item;
}

.record {
  background: #e8eee6;
  @extend %item;
}

.notes {
  margin-right: auto;
  margin-left: 16px;
  color: #999;
}
.chart{
  width: 430%;
  &-wrapper{
    overflow: auto;
    &::-webkit-scrollbar {
      //隐藏滚动条
      display: none;
    }
  }
}
</style>

