<template>
  <div >
    <label class="formItem">
      <span class="name">{{this.fieldName}}</span>
      <template v-if="type === 'date'">
        <input :type="type ||'text'"
               :value="x(value)"
               @input="onValueChanged($event.target.value)"
               :placeholder="this.placeholder">
      </template>
      <template v-else>
        <input :type="type ||'text'"
               :value="value"
               @input="onValueChanged($event.target.value)"
               :placeholder="this.placeholder">
      </template>
    </label>
  </div>
</template>

<script lang="ts">
  import Vue from 'vue';
  import {Component, Prop, Watch} from 'vue-property-decorator';
  import dayjs from 'dayjs'

  @Component
  export default class FormItem extends Vue {
    @Prop({default: ''}) readonly value!: string;

    @Prop({required: true}) fieldName!: string;
    @Prop() placeholder?: string;
    @Prop() type?: string;

    onValueChanged(value: string) {
      this.$emit('update:value', value);
    }
    //声明一个方法 x
    //x 接受一个isoString:string
    //然后把它变成另个一个String
    x(isoString:string){
      return dayjs(isoString).format('YYYY-MM-DD');
    }
  }
</script>

<style lang="scss" scoped>
  .formItem {
    width: 100%;
    font-size: 14px;
    padding-left: 16px;
    display: flex;
    align-items: center;
    input {
      border: none;
      text-align: left;
      height: 40px;
      width: 160px;
      flex-grow: 1;
      background: transparent;
      padding-right: 16px;
    }

  }
</style>
