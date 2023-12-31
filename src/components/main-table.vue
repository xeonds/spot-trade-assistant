<template>
  <el-card class="main">
    <template #header v-if="props.contain_top">
      <div class="card-header" @click="cancel_select">
        <el-row>
          <slot name="top"></slot>
        </el-row>
        <el-row class="card-nav">
          <div class="operation">
            <el-button
              type="primary"
              plain
              class="op-button"
              v-for="(item, index) in command"
              :key="index"
              @click.stop="emits('handle', index)"
              >{{ item }}</el-button
            >
            <slot name="command"></slot>
          </div>
          <div class="title">{{ props.name }}</div>
        </el-row>
      </div>
    </template>
    <div class="table-area">
      <el-table
        v-el-table-infinite-scroll="() => emits('load')"
        border
        :header-row-class-name="props.id"
        :highlight-current-row="props.enable_select"
        :data="table_data"
        row-key="id"
        align="left"
        :height="props.height ? props.height + 'vh' : '180'"
        @cell-contextmenu="(row: any, col: any, _: any, event: any,) => {
          emits('menu', row, col, event)
        }
          "
        @row-click="(row: any, col: any) => emits('click_row', row, col)"
        @selection-change="handleSelectionChange"
        @expand-change="(row:any) => emits('expand-change', row)"
        ref="main"
        :header-cell-style="{
          color: '#000'
        }"
        :cell-style="{
          padding: '2px',
          color: '#000'
        }"
      >
        <slot name="table-extend-start"></slot>
        <el-table-column v-if="props.selectable" type="selection" width="55" />
        <!-- 折叠显示列 -->
        <AFTableColumn type="expand" v-if="props.hasfold" :resizable="false">
          <template #default="props">
            <!-- 折叠插槽 -->
            <slot name="fold_content"></slot>
            <!-- 遍历col数组 -->
            <template v-for="(item, index) in col" :key="item.prop">
              <!-- 找到需要折叠的列 -->
              <template v-if="col[index].fold">
                <div class="table_fold">
                  <el-descriptions :title="col[index].label">
                    <el-descriptions-item
                      v-for="item2 in col[index].son_labels"
                      :label="item2.label"
                      :key="item2.prop"
                    >
                      <el-tag
                        style="margin-inline: 0.4rem"
                        v-if="typeof item2.prop == 'string'"
                      >
                        {{ props.row[item2.prop] }}</el-tag
                      >
                      <!-- 最大支持两层嵌套的prop -->
                      <template v-else-if="typeof item2.prop == 'object'">
                        <div class="tag-container">
                          <el-tag
                            v-for="item3 in props.row[item2.prop[0]]"
                            :key="item3"
                          >
                            {{ item3[item2.prop[1]] }}</el-tag
                          >
                        </div>
                      </template>
                    </el-descriptions-item>
                  </el-descriptions>
                </div>
              </template>
            </template>
          </template>
        </AFTableColumn>
        <!-- 默认显示列 -->
        <template v-for="(item, index) in col" :key="`col_${item.label}`">
          <!-- 状态调整 -->
          <AFTableColumn
            v-if="
              col[index].prop == 'status' &&
              props.status_change &&
              !col[index].fold
            "
            label="状态"
          >
            <template #default="scope">
              <el-switch
                active-value="1"
                style="height: 0.9vh"
                inactive-value="0"
                v-model="table_data[scope.$index].status"
                @change="change_status(table_data[scope.$index].id)"
              />
            </template>
          </AFTableColumn>
          <!-- 普通显示 -->
          <AFTableColumn
            v-else-if="
              !col[index].fold &&
              col[index].hidden != true &&
              !col[index].children
            "
            :prop="col[index].prop"
            :label="item.label"
            align="center"
            :filters="col[index].filters"
            :filter-method="col[index].filter_method"
          >
          </AFTableColumn>
          <el-table-column
            v-else-if="
              !col[index].fold &&
              col[index].hidden != true &&
              col[index].children
            "
            :label="item.label"
            align="center"
          >
            <AFTableColumn
              v-for="item2 in col[index].children"
              :key="`col_${item2.label}`"
              :prop="item2.prop"
              :label="item2.label"
              align="center"
              :filters="item2.filters"
              :filter-method="item2.filter_method"
            />
          </el-table-column>
        </template>
        <slot name="table-extend-end"></slot>
        <!-- 状态表 -->

        <!-- 表格尾部按键插槽 -->
        <AFTableColumn
          :resizable="false"
          :label="props.contain_extend"
          v-if="props.contain_extend"
        >
          <template #default="scope">
            <slot name="table-extend-end2" v-bind:row="scope"></slot>
          </template>
        </AFTableColumn>
      </el-table>
    </div>
  </el-card>
</template>

<script lang="ts" setup>
import { reactive, onMounted } from 'vue'
import Sortable from 'sortablejs'
import AFTableColumn from './AFTableColumn.vue'

let main = ref()
let emits = defineEmits([
  'handle',
  'menu',
  'click_row',
  'load',
  'cancel_select',
  'select',
  'expand-change'
])

const cancel_select = () => {
  if (props.enable_select) {
    main.value.setCurrentRow()
    emits('cancel_select', props.name)
  }
}

let props = defineProps([
  'command',
  'name',
  'color',
  'contain_top',
  'table_data',
  'col',
  'id',
  'status_change',
  'width',
  'hasfold',
  'enable_select',
  'height',
  'contain_extend',
  'contain_extend2',
  'selectable'
])

interface COL {
  label: string
  prop: string
  fold: boolean
  son_labels: any
}

/* eslint-disable */
// eslint-disable-next-line vue/no-setup-props-destructure
let table_data = reactive(<any>props.table_data)
let col: COL[] = reactive(<any>props.col)
let command = reactive(<any>props.command)

//列拖拽
const columnDrop = () => {
  const wrapperTr: HTMLElement = <HTMLElement>(
    document.querySelector('.' + props.id)
  )
  Sortable.create(wrapperTr, {
    animation: 180,
    delay: 0,
    onEnd: (evt: any) => {
      const oldItem = col[evt.oldIndex]
      col.splice(evt.oldIndex, 1)
      col.splice(evt.newIndex, 0, oldItem)
      console.log(col)
    }
  })
}

const handleSelectionChange = (val: any) => {
  emits('select', val, props.id)
}

onMounted(() => {
  if (!props.hasfold) {
    columnDrop()
  }
})

//状态表切换功能
const change_status = (id: string) => {
  props.status_change(id)
}
</script>

<style lang="less">
.el-card__header {
  background-color: #f7f6f4;
  padding-top: 0.5rem;
  padding-bottom: 0.5rem;
}
.el-card__body {
  padding: 0px;
}
</style>

<style lang="less" scoped>
.main {
  box-shadow: none;

  .card-header {
    .title {
      font-size: 1.4rem;
      font-family: NAME, sans-serif;
      color: var(--el-color-primary);
    }
    .card-nav {
      display: flex;
      flex-flow: row nowrap;
      justify-content: space-between;
      .operation {
        display: flex;
        flex-flow: row nowrap;
        justify-content: center;
        align-items: center;
        .op-button {
          margin-right: 0.5rem;
        }
      }
    }
  }
}
.table_fold {
  padding-inline: 1rem;
}
.tag-container {
  display: flex;
  flex-flow: row wrap;
  justify-content: flex-start;
  align-items: center;
  .el-tag {
    margin: 0.4rem;
  }
}
</style>
