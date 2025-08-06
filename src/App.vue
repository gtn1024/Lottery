<script setup>
import _ from 'lodash'
import {
  NButton,
  NCard,
  NForm,
  NFormItem,
  NGrid,
  NGridItem,
  NInput,
  NInputNumber,
  NModal,
  NModalProvider,
} from 'naive-ui'
import { onMounted, reactive, ref } from 'vue'

const formRef = ref(null)
const showModal = ref(false)
const config = reactive({
  logo: 'LOGO',
  title: '抽奖名称',
  listString: '',
  count: 5,
})
const tempConfig = ref({
  logo: '',
  title: '',
  listString: '',
  count: 5,
})
onMounted(() => {
  // 从localstorage获取数据
  const storedConfig = localStorage.getItem('config')
  if (storedConfig) {
    try {
      Object.assign(config, JSON.parse(storedConfig))
    }
    catch {
      localStorage.setItem('config', JSON.stringify(config))
    }
  }
  else {
    // 如果没有数据，设置默认值
    localStorage.setItem('config', JSON.stringify(config))
  }
})

function saveModal() {
  const list = tempConfig.value.listString?.split('\n').map(item => item.trim()).filter(Boolean) || []
  const count = tempConfig.value.count || 5
  if (list.length < count) {
    // eslint-disable-next-line no-alert
    alert(`列表项数量不足，至少需要 ${count} 项`)
    return
  }
  Object.assign(config, tempConfig.value)
  localStorage.setItem('config', JSON.stringify(config))
  showModal.value = false
}

const result = ref([])
const slicedResult = ref([])
const intervalId = ref(null)
function start() {
  const list = config.listString?.split('\n').map(item => item.trim()).filter(Boolean) || []
  const count = config.count || 5
  if (list.length < count) {
    // eslint-disable-next-line no-alert
    alert(`列表项数量不足，至少需要 ${count} 项`)
    return
  }
  result.value = [...list]
  if (!intervalId.value) {
    intervalId.value = setInterval(() => {
      const s = _.shuffle(result.value)
      result.value = s
      slicedResult.value = s.slice(0, count)
    }, 50)
  }
}
function end() {
  if (intervalId.value) {
    clearInterval(intervalId.value)
    intervalId.value = null

    let history = []
    try {
      history = JSON.parse(localStorage.getItem('history') || '[]')
    }
    catch {
    }
    history.push({
      config: { ...config },
      result: [...slicedResult.value],
      timestamp: Date.now(),
    })
    localStorage.setItem('history', JSON.stringify(history))
  }
}
</script>

<template>
  <NModalProvider>
    <div class="h-screen bg-gray-100 flex items-center justify-center">
      <div class="p-4 bg-white rounded-lg shadow w-3/4 h-3/4 flex flex-col">
        <div class="text-center mb-4 text-4xl">
          <img
            v-if="config.logo?.startsWith('http://') || config.logo?.startsWith('https://')"
            alt="logo"
            :src="config.logo"
          >
          <span v-else>{{ config.logo }}</span>
        </div>
        <div class="text-2xl text-center mb-2">
          {{ config.title }}
        </div>
        <div class="flex justify-center items-center">
          <div class="flex gap-2">
            <NButton size="small" @click="start">
              开始
            </NButton>
            <NButton size="small" :secondary="true" @click="end">
              停止
            </NButton>
            <NButton
              size="small" :secondary="true"
              @click="() => {
                Object.assign(tempConfig, config)
                showModal = true
              }"
            >
              设置
            </NButton>
            <!-- <NButton size="small" :secondary="true">
              历史记录
            </NButton> -->
          </div>
        </div>
        <div class="flex justify-center mt-4 w-full flex-1 overflow-hidden">
          <div v-if="!slicedResult.length && !intervalId" class="text-2xl">
            <span class="text-blue">抽奖结果将显示在这里</span>
          </div>
          <div v-else-if="intervalId" class="text-2xl">
            <span class="text-blue">抽奖中...</span>
          </div>
          <div v-else class="text-2xl w-full h-full overflow-y-auto">
            <NGrid cols="2 400:4 600:6" class="max-h-full">
              <NGridItem v-for="item in slicedResult" :key="item">
                <div class="px-2 text-center">
                  {{ item }}
                </div>
              </NGridItem>
            </NGrid>
          </div>
        </div>
      </div>
    </div>
    <NModal v-model:show="showModal">
      <NCard
        style="width: 600px"
        title="设置"
        :bordered="false"
        size="medium"
        role="dialog"
        aria-modal="true"
      >
        <NForm ref="formRef" :model="tempConfig">
          <NFormItem path="logo" label="Logo （文字或 URL）">
            <NInput v-model:value="tempConfig.logo" @keydown.enter.prevent />
          </NFormItem>
          <NFormItem path="title" label="标题">
            <NInput v-model:value="tempConfig.title" @keydown.enter.prevent />
          </NFormItem>
          <NFormItem path="title" label="抽取个数">
            <NInputNumber v-model:value="tempConfig.count" @keydown.enter.prevent />
          </NFormItem>
          <NFormItem path="listString" label="列表（一行一个）">
            <NInput v-model:value="tempConfig.listString" type="textarea" />
          </NFormItem>
        </NForm>
        <template #footer>
          <div class="flex justify-end gap-2">
            <NButton @click="() => showModal = false">
              取消
            </NButton>
            <NButton
              type="primary"
              @click="saveModal"
            >
              保存
            </NButton>
          </div>
        </template>
      </NCard>
    </NModal>
  </NModalProvider>
</template>
