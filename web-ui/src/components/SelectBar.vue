<template>
  <el-select :value="$t(typeName+'.'+value)" @input="change($event)">
    <el-option v-for="item in typeOptions" :key="item.value" :label="$t(typeName+'.'+item.label)" :value="item.value">
    </el-option>
  </el-select>
</template>

<script>
  export default {
    props: {
      //默认选中内容
      value: {
        default: 0
      },
      //下框列表
      typeOptions: {
        type:Array,
        default:()=> [
          {value: '0', label: '0'},
          {value: '1', label: '1'},
          {value: '2', label: '2'},
          {value: '3', label: '3'},
          {value: '4', label: '4'},
          {value: '5', label: '5'},
          {value: '6', label: '6'},
          {value: '7', label: '7'},
          {value: '8', label: '8'},
          {value: '9', label: '9'},
          {value: '10', label: '10'},
          {value: '11', label: '11'},
          {value: '12', label: '12'},
          {value: '13', label: '13'},
          {value: '14', label: '14'},
          {value: '15', label: '15'},
          {value: '16', label: '16'},
          {value: '17', label: '17'},
          {value: '18', label: '18'},
          {value: '19', label: '19'},
          {value: '20', label: '20'},
          {value: '21', label: '21'},
          {value: '22', label: '22'},
          {value: '23', label: '23'},
          {value: '24', label: '24'},
          {value: '25', label: '25'},
          {value: '26', label: '26'},
        ]
      },
      //循环字段名
      typeName:{
        type:String,
        default:"type"
      }
    },
    methods: {
      change(val) {
        this.$emit('change', val);
      }
    },
  }
</script>

<style lang="less">
.el-select .el-input .el-input__inner{
  background: #F2F7FF;
  color: #000000;
  font-size: 14px;
  border: 1px solid #F0F0F8;
  border-radius: 6px;
}
</style>
