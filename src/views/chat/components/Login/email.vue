<script setup lang="ts">

import {NButton, NInput, NModal, NCheckbox, useMessage} from "naive-ui";
import {computed, ref} from "vue";
import {fetchUser, fetchVerify} from "@/api";
import {useAuthStore} from "@/store";

interface Props {
  visible: boolean
}

interface LoginResponse {
  token: string
}

const props = defineProps<Props>()
const authStore = useAuthStore()
const ms = useMessage()
const loading = ref(false)

const username = ref('')
const password = ref('')
const checked = ref(false)

const disabledCheck = computed(() => !username.value.trim() || !password.value.trim() || loading.value)

async function handleVerify() {
  const userName = username.value.trim()
  const pwd = password.value.trim()

  if (!userName || !pwd)
    return

  if (!checked.value) {
    ms.error('请先阅读并同意用户协议和隐私政策')
    return
  }


  try {
    loading.value = true
    const {data} = await fetchUser<LoginResponse>(userName, pwd)
    authStore.setToken(data.token)
    ms.success('success')
    window.location.reload()
  } catch (error: any) {
    ms.error(error.message ?? 'error')
    authStore.removeToken()
  } finally {
    loading.value = false
  }
}

function handlePress(event: KeyboardEvent) {
  if (event.key === 'Enter' && !event.shiftKey) {
    event.preventDefault()
    handleVerify()
  }
}
</script>

<template>
  <NModal :show="visible" style="width: 90%; max-width: 640px">
    <div class="p-10 bg-white rounded dark:bg-slate-800">
      <div class="space-y-6">
        <header class="space-y-2">
          <h2 class="text-2xl font-bold text-center text-slate-800 dark:text-neutral-200">
            请您先登录
          </h2>
          <p class="text-base text-center text-slate-500 dark:text-slate-500">
            {{ $t('common.unauthorizedTips') }}
          </p>
        </header>
        <div class="space-y-8">
          <NInput v-model:value="username" type="text" placeholder="手机号/邮箱"/>
          <NInput v-model:value="password" show-password-on="mousedown" type="password" placeholder="密码"
                  @keypress="handlePress"/>
        </div>
        <NCheckbox v-model:checked="checked" class="float-right pb-4">
          我已经阅读并同意<a class="text-slate-500">《用户协议》</a>和<a class="text-slate-500">《隐私政策》</a>
        </NCheckbox>
        <NButton
            block
            type="primary"
            :disabled="disabledCheck"
            :loading="loading"
            @click="handleVerify"
        >
          {{ $t('common.verify') }}
        </NButton>
      </div>
    </div>
  </NModal>
</template>

<style scoped>

::v-deep(.n-input .n-input__placeholder) {
  line-height: var(--n-height, 34px);
}
</style>
