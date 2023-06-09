<script setup lang="ts">
import { computed, ref } from 'vue'
import type { FormInst } from 'naive-ui'
import { NModal, useMessage } from 'naive-ui'
import { useChatStore, useDrawStore } from '@/store'
import { exchangeCode } from '@/api/auth'
import { checkChat } from '@/api/user'

const props = defineProps<{
  visible: boolean
}>()

const emit = defineEmits<{
  (e: 'update:visible', visible: boolean): void
}>()

const message = useMessage()
const chatStore = useChatStore()
const drawStore = useDrawStore()

const form = ref<FormInst | null>(null)
const model = ref({
  code: '',
})
const rules = {
  code: [
    {
      required: true,
      message: '请输入兑换码',
    },
  ],
}
const submit = (e: MouseEvent) => {
  e.preventDefault()
  form.value?.validate((errors) => {
    if (!errors) {
      exchangeCode(model.value.code).then((res) => {
        if (res.data) {
          message.success('兑换成功')
          checkChat().then((resp) => {
            location.hash.includes('chat') ? chatStore.updateRemainingMessages() : drawStore.updateRemainingMessages()
          })
        	emit('update:visible', false)
        }
        else {
          message.error(res.msg || '提交失败')
        }
      })
    }
    else { message.error('提交失败') }
  })
}

const show = computed({
  get() {
    return props.visible
  },
  set(value) {
    emit('update:visible', value)
  },
})

const handleClose = () => {
  emit('update:visible', false)
}
</script>

<template>
  <NModal v-model:show="show" title="兑换码兑换" preset="card" style="width: 95%; max-width: 650px" @update:show="handleClose">
    <n-card
      :bordered="false"
      size="huge"
      role="dialog"
      aria-modal="true"
    >
      <n-form ref="form" :model="model" :rules="rules">
        <n-form-item path="age" label="兑换码">
          <n-input v-model:value="model.code" @keydown.enter.prevent />
        </n-form-item>
      </n-form>
      <template #footer>
        <n-button round type="primary" @click="submit">
          提交
        </n-button>
      </template>
    </n-card>
  </NModal>
</template>

<style scoped lang="less">
</style>
