<template>
  <Table
    id="table-model"
    :col="props.col"
    :table_data="props.data"
    :contain_top="true"
    :contain_command="true"
    :command="props.command"
    :height="props.height"
    :name="props.name"
    :hasfold="false"
    :enable_select="false"
    @menu="menu"
    @handle="(index:any) => emits('handle', index)"
    @select="(val, id) => emits('select', val, id)"
    @load="emits('load')"
    @expand-change="
      (row) => {
        emits('expand-change', row)
      }
    "
    :contain_extend="props.extend"
    :contain_extend2="props.extend2"
    :selectable="props.selectable"
  >
    <template #top>
      <slot name="top"></slot>
    </template>
    <template #command>
      <slot name="command"></slot>
    </template>
    <template #table-extend-end2="Slotprops">
      <slot name="extend2" v-bind:row="Slotprops.row"></slot>
    </template>
    <template #table-extend-end3="Slotprops">
      <slot name="extend3" v-bind:row="Slotprops.row"></slot>
    </template>
    <template #fold_content>
      <slot name="fold_content"></slot>
    </template>
  </Table>
</template>

<script lang="ts" setup>
import Table from './main-table.vue'
const emits = defineEmits(['menu', 'handle', 'select', 'load', 'expand-change'])

/**
 * @id 表格id
 * @col 表格列
 * @name 表格标题
 * @data 表格数据
 * @command 表格命令
 */

const props = defineProps([
  'id',
  'col',
  'name',
  'data',
  'height',
  'command',
  'extend',
  'extend2',
  'selectable'
])
const menu = (row: any, col: any, event: any) => {
  event.preventDefault()
  emits('menu', props.name, row, col, event)
}
</script>
