<template>
  <Layout class="layout">
    <Chart :options="x"/>
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

//放入组件里
@Component({
  components: {Tabs, Chart},
})
export default class Statistics extends Vue {
  tagString(tags: Tag[]) {
    return tags.length === 0 ? '无' : tags.map(t => t.name).join('，');
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

  //声明x
  get x() {
    return {
      xAxis: {
        type: 'category',
        data: [
            '1', '2', '3', '4', '5', '6', '7',
          '8', '9', '10', '11', '12', '13', '14',
          '15', '16', '17', '18', '19', '20', '21',
          '22', '23', '24', '25', '26', '27',
          '28','29','30'
        ]
      },
      yAxis: {
        type: 'value'
      },
      tooltip:{
        show: true,
        triggerOn: 'click'
      },
      series: [{

        data: [120, {
          value: 200,
          itemStyle: {
            color: '#a90000'
          }
        }, 150, 80, 70, 110, 130],
        type: 'line'
      }]
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
        console.log(sum);
        console.log(item);
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
</style>

