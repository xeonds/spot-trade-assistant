<template>
  <!-- 使用透传将v-model绑定到dialog元素上，从而控制组件显示 -->
  <el-dialog :title="props.title">
    <el-form
      :model="data"
      style="display: flex; flex-wrap: wrap"
      :rules="props.rules"
      ref="ruleFormRef"
    >
      <el-form-item
        :label="item.label"
        :label-width="160"
        v-for="item in props.col"
        :key="item.label"
        style="width: 24rem"
        :prop="item.prop"
        @click="() => emit('click', item)"
      >
        <el-text v-if="item.type == 'label'" />
        <el-date-picker
          v-model="data[item.prop]"
          type="date"
          placeholder="选择日期"
          v-if="item.type == 'date'"
        />
        <el-input v-model="data[item.prop]" v-if="item.type == 'string'" />
        <el-input
          v-model.number="data[item.prop]"
          type="number"
          v-if="item.type == 'number'"
        />
        <el-radio-group v-model="data[item.prop]" v-if="item.type == 'select'">
          <el-radio
            v-for="option in item.options"
            :key="option.label"
            :label="option.value"
            >{{ option.label }}</el-radio
          >
        </el-radio-group>
        <el-select
          v-model="data[item.prop]"
          multiple
          placeholder="选择"
          style="width: 240px"
          v-if="item.type == 'multi-select'"
        >
          <el-option
            v-for="option in item.options"
            :key="option.value"
            :label="option.label"
            :value="option.value"
          />
        </el-select>
        <el-select
          v-model="data[item.prop]"
          placeholder="选择"
          v-if="item.type == 'single-select-cascader'"
        >
          <el-option-group
            v-for="group in item.options"
            :key="group.label"
            :label="group.label"
          >
            <el-option
              v-for="item in group.options"
              :key="item.value"
              :label="item.label"
              :value="item.value"
            ></el-option
          ></el-option-group>
        </el-select>
        <el-select
          v-model="data[item.prop]"
          placeholder="选择"
          style="width: 240px"
          @change="(val:any) => handlechange(item.flag, val,item.bind)"
          v-if="item.type == 'single-select'"
        >
          <el-option
            v-for="option in item.options"
            :key="option.value"
            :label="option.label"
            :value="option.value"
          />
        </el-select>
      </el-form-item>
    </el-form>
    <template #footer>
      <span class="dialog-footer">
        <el-button @click="emit('close')" type="primary" plain>
          取消
        </el-button>
        <el-button @click="() => submitForm(ruleFormRef)" type="primary"
          >确定</el-button
        >
      </span>
    </template>
  </el-dialog>
</template>

<script lang="ts" setup>
let ruleFormRef = ref()
let props = defineProps(['visible', 'title', 'col', 'rules'])
//rules  表单校验规则
let emit = defineEmits(['submit', 'close', 'click', 'write'])
//write 用于标志选项变换 用于加载
let data = reactive(<any>{})

/**
 * 选项变化
 */
const handlechange = (flag: string, val: any, bind: any) => {
  if (flag) {
    for (let i = 0; i < bind.length; i++) {
      data[bind[i]] = ''
    }
    emit('write', flag, val)
  }
}
/**
 * 表单清空
 */
const clear = () => {
  for (var key in data) {
    data[key] = ''
  }
}

const submitForm = async (formEl: any) => {
  if (!formEl) return
  await formEl.validate((valid: any, fields: any) => {
    if (valid) {
      emit('submit', data)
    } else {
      console.log('error submit!', fields)
    }
  })
}

defineExpose({
  clear
})
</script>

<style lang="less" scoped>
.table-header {
  margin-bottom: 10px;
}

.header-title {
  font-size: 1.5rem;
  text-align: right;
  color: var(--el-color-primary);
  font-weight: bolder;
}
</style>
