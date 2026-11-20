<script setup lang="ts">
import { ref } from 'vue'
import { useRoute, useRouter } from 'vue-router'
import { ApiError } from '@/api/http'
import { useAuth } from '@/composables/useAuth'
import { toast } from '@/composables/useToast'

const router = useRouter()
const route = useRoute()
const { login } = useAuth()

const identifier = ref('')
const password = ref('')
const showPassword = ref(false)
const remember = ref(true)
const submitting = ref(false)
const formError = ref<string | null>(null)

/** 演示账号：登录页显式写出来，省得每次翻文档 */
const DEMO = { account: 'admin', password: 'admin123' }

function fillDemo(): void {
  identifier.value = DEMO.account
  password.value = DEMO.password
  formError.value = null
}

/**
 * 真登录：调后端 /api/auth/login，成功后回到来源路由（?redirect=）。
 * 校验放在前端做一遍是为了即时反馈，真正的判定仍在后端。
 */
async function submit(): Promise<void> {
  formError.value = null

  if (!identifier.value.trim()) {
    formError.value = '请输入邮箱地址或用户名'
    return
  }
  if (password.value.length < 5) {
    formError.value = '密码至少 5 位'
    return
  }

  submitting.value = true
  try {
    await login(identifier.value, password.value, remember.value)
    toast('登录成功', {
      tone: 'success',
      description: `欢迎回来，${identifier.value.trim()}`
    })
    const redirect = typeof route.query.redirect === 'string' ? route.query.redirect : '/explore'
    await router.replace(redirect)
  } catch (cause) {
    if (cause instanceof ApiError && cause.status === 401) {
      formError.value = '邮箱或密码不正确'
    } else {
      formError.value = cause instanceof Error ? cause.message : String(cause)
    }
  } finally {
    submitting.value = false
  }
}

/** 背景漂浮图标：位置固定写死，避免每次渲染抖动 */
const bubbles = [
  { icon: 'mdi-robot-outline', x: 8, y: 62, s: 52 },
  { icon: 'mdi-chart-line', x: 24, y: 78, s: 46 },
  { icon: 'mdi-robot-happy-outline', x: 55, y: 66, s: 44 },
  { icon: 'mdi-creation', x: 62, y: 72, s: 40 },
  { icon: 'mdi-robot-vacuum', x: 70, y: 53, s: 50 },
  { icon: 'mdi-atom', x: 88, y: 48, s: 44 },
  { icon: 'mdi-brain', x: 32, y: 40, s: 38 },
  { icon: 'mdi-flash-outline', x: 78, y: 84, s: 40 }
]
</script>

<template>
  <div class="login">
    <div class="login__bg">
      <span
        v-for="(b, i) in bubbles"
        :key="i"
        class="login__bubble"
        :style="{
          left: `${b.x}%`,
          top: `${b.y}%`,
          width: `${b.s}px`,
          height: `${b.s}px`,
          fontSize: `${b.s * 0.45}px`
        }"
      >
        <i :class="['mdi', b.icon]" aria-hidden="true" />
      </span>
    </div>

    <section class="login__card">
      <span class="login__card-accent" />

      <div class="login__brand">
        <span class="login__logo">
          <i class="mdi mdi-robot-happy-outline" aria-hidden="true" />
        </span>
        <h1 class="login__title">AI智能体应用开发平台</h1>
        <p class="login__subtitle">构建未来，从智能开始</p>
      </div>

      <form class="login__form" @submit.prevent="submit">
        <div class="login__field">
          <label class="login__label" for="login-email">邮箱地址或用户名</label>
          <v-text-field
            id="login-email"
            v-model="identifier"
            prepend-inner-icon="mdi-email-outline"
            type="text"
            autocomplete="username"
            density="comfortable"
            hide-details
          />
        </div>

        <div class="login__field">
          <label class="login__label" for="login-password">密码</label>
          <v-text-field
            id="login-password"
            v-model="password"
            prepend-inner-icon="mdi-lock-outline"
            :append-inner-icon="showPassword ? 'mdi-eye-outline' : 'mdi-eye-off-outline'"
            :type="showPassword ? 'text' : 'password'"
            autocomplete="current-password"
            density="comfortable"
            hide-details
            @click:append-inner="showPassword = !showPassword"
          />
        </div>

        <div class="login__options">
          <label class="login__remember">
            <input v-model="remember" type="checkbox" />
            <span>记住我</span>
          </label>
          <a class="login__link" href="#" @click.prevent>忘记密码?</a>
        </div>

        <p v-if="formError" class="login__error" role="alert">{{ formError }}</p>

        <button type="submit" class="login__submit" :disabled="submitting">
          <span v-if="!submitting">登录</span>
          <span v-else class="login__spinner" aria-hidden="true" />
        </button>
      </form>

      <div class="login__divider">
        <span>或者使用</span>
      </div>

      <div class="login__oauth">
        <button type="button" class="login__oauth-btn login__oauth-btn--google">
          <i class="mdi mdi-google" aria-hidden="true" />
          <span>Google</span>
        </button>
        <button type="button" class="login__oauth-btn">
          <i class="mdi mdi-github" aria-hidden="true" />
          <span>GitHub</span>
        </button>
      </div>

      <p class="login__demo">
        演示账号 <code>{{ DEMO.account }} / {{ DEMO.password }}</code>
        <button type="button" class="login__demo-fill" @click="fillDemo">填入</button>
      </p>

      <p class="login__footer">
        还没有账户?
        <a class="login__link" href="#" @click.prevent>立即注册</a>
      </p>
    </section>
  </div>
</template>

<style scoped>
.login {
  position: relative;
  min-height: 100dvh;
  display: flex;
  align-items: center;
  justify-content: center;
  padding: var(--spacing-6);
  background: var(--gradient-auth);
  overflow: hidden;
}

.login__bg {
  position: absolute;
  inset: 0;
  pointer-events: none;
}

.login__bubble {
  position: absolute;
  display: inline-flex;
  align-items: center;
  justify-content: center;
  border-radius: 50%;
  background: rgba(255, 255, 255, 0.1);
  color: rgba(255, 255, 255, 0.85);
  transform: translate(-50%, -50%);
}

.login__card {
  position: relative;
  width: 400px;
  max-width: 100%;
  padding: var(--spacing-8) var(--spacing-8) var(--spacing-6);
  background: #fff;
  border-radius: var(--radius-lg);
  box-shadow: 0 24px 60px rgba(43, 28, 90, 0.28);
  overflow: hidden;
  text-align: center;
}

.login__card-accent {
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  height: 3px;
  background: linear-gradient(90deg, #f2e1f9 0%, #e3b8f5 45%, #cfa6f0 100%);
}

.login__brand {
  display: flex;
  flex-direction: column;
  align-items: center;
  margin-bottom: var(--spacing-6);
}

.login__logo {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  width: 56px;
  height: 56px;
  border-radius: 50%;
  background: linear-gradient(135deg, #8b78e6 0%, #6b4fa8 100%);
  color: #fff;
  font-size: 28px;
  margin-bottom: var(--spacing-3);
  box-shadow: 0 6px 16px rgba(107, 79, 168, 0.35);
}

.login__title {
  font-size: 18px;
  font-weight: 600;
  color: var(--color-primary);
  letter-spacing: 0.2px;
}

.login__subtitle {
  margin-top: var(--spacing-1);
  font-size: 12px;
  color: var(--color-text-tertiary);
}

.login__form {
  display: flex;
  flex-direction: column;
  gap: var(--spacing-3);
  text-align: left;
}

/* 标签在框上方（原型就是外置小标签，不是 M3 浮动标签），所以不传 label 给 VTextField */
.login__field {
  display: flex;
  flex-direction: column;
  gap: 4px;
}

.login__label {
  font-size: 12px;
  line-height: 16px;
  color: var(--color-text-secondary);
}

.login__form :deep(.v-field) {
  border-radius: var(--radius-sm);
}

.login__form :deep(.v-field__input) {
  font-size: 13px;
  min-height: 40px;
}

.login__form :deep(.v-field__prepend-inner .v-icon) {
  font-size: 18px;
  color: var(--color-text-tertiary);
}

.login__options {
  display: flex;
  align-items: center;
  justify-content: space-between;
  margin: var(--spacing-1) 0 var(--spacing-3);
}

.login__remember {
  display: inline-flex;
  align-items: center;
  gap: var(--spacing-2);
  font-size: 13px;
  color: var(--color-text-secondary);
  cursor: pointer;
}

.login__remember input {
  width: 16px;
  height: 16px;
  accent-color: var(--color-primary);
  cursor: pointer;
}

.login__link {
  font-size: 13px;
  color: var(--color-primary);
  text-decoration: none;
}

.login__link:hover {
  text-decoration: underline;
}

.login__submit {
  width: 100%;
  height: 40px;
  border: 0;
  border-radius: var(--radius-sm);
  background: var(--gradient-auth);
  color: #fff;
  font-size: 14px;
  font-weight: 600;
  letter-spacing: 2px;
  cursor: pointer;
  transition: filter var(--motion-fast) var(--motion-ease);
}

.login__submit:hover {
  filter: brightness(1.06);
}

.login__divider {
  position: relative;
  display: flex;
  align-items: center;
  justify-content: center;
  margin: var(--spacing-5) 0 var(--spacing-4);
  color: var(--color-text-tertiary);
  font-size: 12px;
}

.login__divider::before {
  content: '';
  position: absolute;
  left: 0;
  right: 0;
  top: 50%;
  height: 1px;
  background: var(--color-divider);
}

.login__divider span {
  position: relative;
  padding: 0 var(--spacing-2);
  background: #fff;
}

.login__oauth {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: var(--spacing-3);
}

.login__oauth-btn {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  gap: var(--spacing-2);
  height: 38px;
  border: 1px solid var(--color-border);
  border-radius: var(--radius-sm);
  background: #fff;
  color: var(--color-text-secondary);
  font-size: 13px;
  cursor: pointer;
  transition: border-color var(--motion-fast) var(--motion-ease);
}

.login__oauth-btn:hover {
  border-color: var(--color-primary-light);
}

.login__oauth-btn--google {
  color: #db4437;
}

.login__footer {
  margin-top: var(--spacing-5);
  font-size: 13px;
  color: var(--color-text-tertiary);
}
.login__error {
  margin: 0 0 var(--spacing-2);
  padding: 6px 10px;
  border-radius: var(--radius-sm);
  background: var(--status-danger-bg);
  color: var(--status-danger);
  font-size: 12px;
  line-height: 18px;
}

.login__spinner {
  width: 14px;
  height: 14px;
  border: 2px solid rgba(255, 255, 255, 0.7);
  border-right-color: transparent;
  border-radius: 50%;
  animation: login-spin 0.7s linear infinite;
}

@keyframes login-spin {
  to {
    transform: rotate(360deg);
  }
}

.login__submit:disabled {
  cursor: not-allowed;
  filter: saturate(0.7);
}

.login__demo {
  margin-top: var(--spacing-4);
  font-size: 12px;
  color: var(--color-text-tertiary);
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 6px;
}

.login__demo code {
  font-family: var(--font-family-mono);
  padding: 1px 6px;
  border-radius: var(--radius-xs);
  background: var(--color-surface-low);
  color: var(--color-text-secondary);
}

.login__demo-fill {
  border: 0;
  background: transparent;
  color: var(--color-primary);
  font-family: inherit;
  font-size: 12px;
  cursor: pointer;
  text-decoration: underline;
}
</style>
