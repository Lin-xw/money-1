<template>
  <Layout class-prefix="layout">
    <NumberPad :value.sync="record.amount" @submit="saveRecord"/>
    <div class="box">
      <div class="notes icon">

        <FormItem field-name=""
                  placeholder="   在这里输入备注"
                  :value.sync="record.notes"
        />
      </div>
      <div class="createdAt">
        <FormItem field-name=""
                  type="date"
                  placeholder="在这里输入日期"
                  :value.sync="record.createdAt"
        />
      </div>
    </div>
    <Tags @update:value="record.tags = $event"/>
    <Tabs :data-source="recordTypeList"
          :value.sync="record.type"/>
  </Layout>
</template>

<script lang="ts">
  import Vue from 'vue';
  import NumberPad from '@/components/Money/NumberPad.vue';
  import FormItem from '@/components/Money/FormItem.vue';
  import Tags from '@/components/Money/Tags.vue';
  import {Component} from 'vue-property-decorator';
  import Tabs from '@/components/Tabs.vue';
  import recordTypeList from '@/constants/recordTypeList';

  @Component({
    components: {Tabs, Tags, FormItem, NumberPad},
  })
  export default class Money extends Vue {
    get recordList() {
      return this.$store.state.recordList;
    }

    recordTypeList = recordTypeList;

    record: RecordItem = {//默认初始值
      tags: [], notes: '', type: '-', amount: 0, createdAt:new Date().toISOString()
    };//添加日期默认初始值为当天

    created() {
      this.$store.commit('fetchRecords');
    }

    onUpdateNotes(value: string) {
      this.record.notes = value;
    }

    saveRecord() {
      if(!this.record.tags ||this.record.tags.length === 0){
        window.alert('请至少选择一个标签噢')
        return
      }
      this.$store.commit('createRecord', this.record);
      if (this.$store.state.createRecordError === null){
        window.alert('木木记下来啦');
        this.record.notes = '';
      }
    }
  }
</script>

<style lang="scss" scoped>
  ::v-deep .layout-content {
    display: flex;
    flex-direction: column-reverse;
  }
  .createdAt{
    height: 64px;
    width: 45%;
    float: right;
    padding: 12px 0;
  }
  .notes {
    text-align: right;
    height: 64px;
    float: left;
    width: 55%;
    padding: 12px 0;
  }
</style>

