<template>
  <Table
    :contain_find="true"
    :contain_command="true"
    :table_data="data"
    :col="property"
    :search_item="search_item"
    :command="command"
    @handle="handle"
    name="合同模板"
    color="#e7e6e6"
    id="finance1"
  ></Table>
</template>

<script lang="ts" setup>
import Table from '../../components/main-table.vue'
import { reactive } from 'vue'

let data = reactive([
  {
    name: '341134',
    date: '2023-11-9-9',
    sex: 'male',
    test: 'sdfas',
    test2: 'dfas',
    test34: 'fdas'
  },
  {
    name: '341134',
    date: '2023-11-9-9',
    sex: 'male',
    test: 'sdfas',
    test2: 'dfas',
    test34: 'fdas'
  },
  {
    name: '341134',
    date: '2023-11-9-9',
    sex: 'male',
    test: 'sdfas',
    test2: 'dfas',
    test34: 'fdas'
  },
  {
    name: '341134',
    date: '2023-11-9-9',
    sex: 'male',
    test: 'sdfas',
    test2: 'dfas',
    test34: 'fdas'
  }
])
let search_item = reactive(['贸易类型', '订单模式', '交收方式', '购/销'])
let property = reactive(['name', 'date', 'sex', 'test', 'test2', 'test34'])
let command = reactive(['添加', '删除', '修改'])

const handle = (a: number) => {
  console.log(a)
}
</script>
