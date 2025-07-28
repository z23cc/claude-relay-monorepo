<template>
  <div class="bg-gradient-to-br from-orange-50 via-orange-50 to-amber-50 min-h-screen">
    <!-- 背景装饰 -->
    <div class="absolute inset-0 overflow-hidden">
      <div class="absolute top-20 right-20 w-64 h-64 bg-orange-200 rounded-full mix-blend-multiply filter blur-xl opacity-30 animate-pulse"></div>
      <div class="absolute bottom-20 left-20 w-64 h-64 bg-orange-200 rounded-full mix-blend-multiply filter blur-xl opacity-30 animate-pulse"></div>
    </div>

    <!-- 顶部导航 -->
    <nav class="relative z-10 bg-white/80 backdrop-blur-sm border-b border-orange-100 sticky top-0">
      <div class="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8">
        <div class="flex justify-between h-16">
          <div class="flex items-center space-x-4">
            <NuxtLink to="/admin/dashboard" 
                     class="flex items-center space-x-2 text-orange-600 hover:text-orange-700 px-3 py-2 rounded-xl hover:bg-white/50 transition duration-200">
              <svg class="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M15 19l-7-7 7-7"></path>
              </svg>
              <span>返回</span>
            </NuxtLink>
            <div class="h-6 w-px bg-orange-200"></div>
            <div>
              <h1 class="text-xl font-bold bg-gradient-to-r from-orange-600 to-orange-600 bg-clip-text text-transparent">
                添加模型供应商
              </h1>
              <p class="text-xs text-gray-500">接入第三方AI模型服务</p>
            </div>
          </div>
          <div class="flex items-center">
            <button @click="logout" 
                    class="text-gray-500 hover:text-orange-600 px-3 py-2 rounded-xl hover:bg-white/50 transition duration-200">
              退出登录
            </button>
          </div>
        </div>
      </div>
    </nav>

    <div class="relative z-10 max-w-4xl mx-auto py-8 px-4 sm:px-6 lg:px-8">
      <!-- 供应商选择 -->
      <div class="mb-8">
        <h2 class="text-2xl font-bold text-gray-900 mb-2">选择模型供应商</h2>
        <p class="text-gray-600 mb-6">选择您要接入的AI模型供应商，只需填入API Key即可快速接入</p>
        <div class="grid grid-cols-1 md:grid-cols-3 gap-6">
          <div v-for="(config, key) in providerConfigs" :key="key" 
               @click="selectProvider(key)"
               class="bg-white/80 backdrop-blur-sm border-2 border-orange-200 rounded-2xl p-6 cursor-pointer hover:shadow-lg hover:border-orange-300 transition duration-300 group">
            <div class="flex flex-col items-center text-center space-y-4">
              <div :class="config.icon" class="w-16 h-16 bg-gradient-to-br rounded-2xl flex items-center justify-center">
                <svg class="w-8 h-8 text-white" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                  <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M19 11H5m14 0a2 2 0 012 2v6a2 2 0 01-2 2H5a2 2 0 01-2-2v-6a2 2 0 012-2m14 0V9a2 2 0 00-2-2M5 11V9a2 2 0 012-2m0 0V5a2 2 0 012-2h6a2 2 0 012 2v2M7 7h10"></path>
                </svg>
              </div>
              <div>
                <h3 class="text-lg font-bold text-gray-900 group-hover:text-orange-600 transition duration-200">
                  {{ config.name }}
                </h3>
                <p class="text-sm text-gray-600 mt-1">
                  {{ config.description }}
                </p>
              </div>
              <div class="flex items-center space-x-1 text-xs text-orange-600 font-medium">
                <span>选择</span>
                <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                  <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M9 5l7 7-7 7"></path>
                </svg>
              </div>
            </div>
          </div>
        </div>
      </div>

      <!-- 配置表单 -->
      <div v-if="selectedProvider" class="bg-white/80 backdrop-blur-sm shadow-xl rounded-2xl border border-orange-100">
        <div class="px-6 py-6">
          <div class="flex items-center space-x-3 mb-6">
            <div :class="providerConfigs[selectedProvider].icon" 
                 class="w-12 h-12 bg-gradient-to-br rounded-2xl flex items-center justify-center">
              <svg class="w-6 h-6 text-white" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M19 11H5m14 0a2 2 0 012 2v6a2 2 0 01-2 2H5a2 2 0 01-2-2v-6a2 2 0 012-2m14 0V9a2 2 0 00-2-2M5 11V9a2 2 0 012-2m0 0V5a2 2 0 012-2h6a2 2 0 012 2v2M7 7h10"></path>
              </svg>
            </div>
            <div>
              <h3 class="text-xl font-bold text-gray-900">{{ providerConfigs[selectedProvider].name }}</h3>
              <p class="text-sm text-gray-600">{{ providerConfigs[selectedProvider].description }}</p>
            </div>
          </div>
          
          <form @submit.prevent="handleSubmit" class="space-y-6">
            <div>
              <label for="api-key" class="block text-sm font-medium text-gray-700 mb-2">
                API Key <span class="text-red-500">*</span>
              </label>
              <div class="relative">
                <div class="absolute inset-y-0 left-0 pl-3 flex items-center pointer-events-none">
                  <svg class="h-5 w-5 text-gray-400" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                    <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M15 7a2 2 0 012 2m4 0a6 6 0 01-7.743 5.743L11 17H9v2H7v2H4a1 1 0 01-1-1v-2.586a1 1 0 01.293-.707l5.964-5.964A6 6 0 1721 9z"></path>
                  </svg>
                </div>
                <input type="password" 
                       v-model="form.apiKey"
                       required
                       class="block w-full pl-10 pr-3 py-3 border border-gray-200 rounded-xl text-gray-900 placeholder-gray-500 focus:outline-none focus:ring-2 focus:ring-orange-500 focus:border-transparent transition duration-200"
                       placeholder="请输入您的 API Key">
              </div>
              <p class="mt-2 text-xs text-gray-500">
                {{ providerConfigs[selectedProvider].helpText }}
              </p>
            </div>

            <!-- 模型选择 -->
            <div>
              <label for="model" class="block text-sm font-medium text-gray-700 mb-2">
                模型 <span class="text-red-500">*</span>
              </label>
              <div v-if="providerConfigs[selectedProvider].models.length > 0">
                <select v-model="form.model"
                        required
                        class="block w-full px-3 py-3 border border-gray-200 rounded-xl text-gray-900 focus:outline-none focus:ring-2 focus:ring-orange-500 focus:border-transparent transition duration-200">
                  <option value="">请选择模型</option>
                  <option v-for="model in providerConfigs[selectedProvider].models" :key="model" :value="model">
                    {{ model }}
                  </option>
                </select>
              </div>
              <div v-else>
                <input type="text" 
                       v-model="form.model"
                       required
                       class="block w-full px-3 py-3 border border-gray-200 rounded-xl text-gray-900 placeholder-gray-500 focus:outline-none focus:ring-2 focus:ring-orange-500 focus:border-transparent transition duration-200"
                       placeholder="请输入模型名称，例如：gpt-3.5-turbo">
              </div>
            </div>

            <!-- 可选的自定义名称 -->
            <div>
              <label for="custom-name" class="block text-sm font-medium text-gray-700 mb-2">
                自定义名称 <span class="text-gray-400">(可选)</span>
              </label>
              <input type="text" 
                     v-model="form.customName"
                     class="block w-full px-3 py-3 border border-gray-200 rounded-xl text-gray-900 placeholder-gray-500 focus:outline-none focus:ring-2 focus:ring-orange-500 focus:border-transparent transition duration-200"
                     :placeholder="`例如：我的${providerConfigs[selectedProvider].name}账号`">
            </div>

            <!-- 按钮 -->
            <div class="flex justify-end space-x-4 pt-6 border-t border-gray-200">
              <button type="button" @click="selectedProvider = null"
                      class="px-6 py-3 border border-orange-300 text-orange-600 rounded-xl hover:bg-orange-50 transition duration-200 font-medium">
                重新选择
              </button>
              <button type="submit" 
                      :disabled="loading"
                      class="px-6 py-3 bg-gradient-to-r from-orange-500 to-orange-600 text-white rounded-xl hover:from-orange-600 hover:to-orange-700 transform hover:scale-105 transition duration-200 shadow-lg font-medium disabled:opacity-50 disabled:cursor-not-allowed">
                {{ loading ? '保存中...' : '保存供应商' }}
              </button>
            </div>
          </form>
        </div>
      </div>
      
      <!-- 错误提示 -->
      <div v-if="error" class="mt-4 p-4 bg-red-50 border border-red-200 rounded-xl">
        <p class="text-sm text-red-600">{{ error }}</p>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import type { AddProviderRequest } from '../../../../shared/types/admin'
import { API_ENDPOINTS } from '../../../../shared/constants/endpoints'
useHead({
  title: '添加模型供应商 - Claude Code 管理中心',
  meta: [
    { name: 'description', content: '添加笭三方AI模型供应商' }
  ]
})

const config = useRuntimeConfig()
const router = useRouter()

const selectedProvider = ref<string | null>(null)
const loading = ref(false)
const error = ref('')

const form = ref({
  apiKey: '',
  customName: '',
  model: ''
})

// 检查认证状态
onMounted(() => {
  if (sessionStorage.getItem('admin_authenticated') !== 'true') {
    router.push('/admin')
    return
  }
})

const selectProvider = (providerKey: string) => {
  selectedProvider.value = providerKey
  form.value.customName = providerConfigs[providerKey].name
  form.value.model = '' // 清空模型选择
  error.value = ''
  
  // 滚动到表单
  nextTick(() => {
    const formElement = document.querySelector('form')
    if (formElement) {
      formElement.scrollIntoView({ behavior: 'smooth' })
    }
  })
}

const handleSubmit = async () => {
  if (!selectedProvider.value || !form.value.apiKey || !form.value.model) {
    error.value = '请填写所有必填字段'
    return
  }

  loading.value = true
  error.value = ''

  try {
    const providerConfig = providerConfigs[selectedProvider.value]
    const request: AddProviderRequest = {
      name: form.value.customName || providerConfig.name,
      type: selectedProvider.value as 'qwen' | 'openai',
      endpoint: providerConfig.endpoint,
      apiKey: form.value.apiKey,
      model: form.value.model
    }

    const response = await $fetch<{ success: boolean; data: any }>(
      API_ENDPOINTS.ADMIN_PROVIDERS,
      {
        method: 'POST',
        baseURL: config.public.apiBaseUrl,
        body: request
      }
    )

    if (response.success) {
      showNotification('模型供应商添加成功！', 'success')
      setTimeout(() => {
        router.push('/admin/dashboard')
      }, 1500)
    }
  } catch (err: any) {
    console.error('Add provider error:', err)
    if (err.data?.error?.message) {
      error.value = err.data.error.message
    } else {
      error.value = '添加供应商失败，请重试'
    }
  } finally {
    loading.value = false
  }
}

const logout = async () => {
  sessionStorage.removeItem('admin_authenticated')
  sessionStorage.removeItem('admin_username')
  await router.push('/admin')
}

const showNotification = (message: string, type: 'success' | 'error' = 'info') => {
  const notification = document.createElement('div')
  notification.className = `fixed top-4 right-4 z-50 px-6 py-3 rounded-xl text-white font-medium shadow-lg transform translate-x-full transition-transform duration-300 ${
    type === 'success' ? 'bg-emerald-500' : type === 'error' ? 'bg-red-500' : 'bg-orange-500'
  }`
  notification.textContent = message
  document.body.appendChild(notification)
  
  setTimeout(() => {
    notification.classList.remove('translate-x-full')
  }, 100)
  
  setTimeout(() => {
    notification.classList.add('translate-x-full')
    setTimeout(() => {
      document.body.removeChild(notification)
    }, 300)
  }, 3000)
}
</script>