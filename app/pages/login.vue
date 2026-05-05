<template>
  <div class="auth-container">
    <div class="auth-card">
      <!-- Logo / Branding -->
      <div class="auth-header">
        <h1>Gatherly</h1>
        <p>Session Builder Portal</p>
      </div>

      <!-- Tab toggle: Login vs Sign Up -->
      <div class="auth-tabs">
        <button
          :class="{ active: mode === 'login' }"
          @click="mode = 'login'"
        >
          Sign In
        </button>
        <button
          :class="{ active: mode === 'signup' }"
          @click="mode = 'signup'"
        >
          Create Account
        </button>
      </div>

      <!-- Error / Success messages -->
      <div
        v-if="message"
        :class="['auth-message', messageType]"
      >
        {{ message }}
      </div>

      <!-- Form -->
      <div class="auth-form">
        <div class="form-group">
          <label>Email</label>
          <input
            v-model="email"
            type="email"
            placeholder="you@example.com"
            @keyup.enter="submit"
          >
        </div>

        <div class="form-group">
          <label>Password</label>
          <input
            v-model="password"
            type="password"
            placeholder="••••••••"
            @keyup.enter="submit"
          >
        </div>

        <div
          v-if="mode === 'signup'"
          class="form-group"
        >
          <label>Full Name (optional)</label>
          <input
            v-model="fullName"
            type="text"
            placeholder="Your name"
          >
        </div>

        <button
          class="auth-submit"
          :disabled="loading"
          @click="submit"
        >
          {{ loading ? 'Please wait…' : mode === 'login' ? 'Sign In' : 'Create Account' }}
        </button>

        <p
          v-if="mode === 'login'"
          class="auth-forgot"
        >
          <a @click="resetPassword">Forgot password?</a>
        </p>
      </div>
    </div>
  </div>
</template>

<script setup>
definePageMeta({ layout: false })

const supabase = useSupabaseClient()
const mode = ref('login')
const email = ref('')
const password = ref('')
const fullName = ref('')
const loading = ref(false)
const message = ref('')
const messageType = ref('info')

async function submit() {
  message.value = ''
  loading.value = true

  if (mode.value === 'login') {
    const { error } = await supabase.auth.signInWithPassword({
      email: email.value,
      password: password.value
    })
    if (error) {
      message.value = error.message
      messageType.value = 'error'
    } else {
      navigateTo('/dashboard')
    }
  } else {
    const { error } = await supabase.auth.signUp({
      email: email.value,
      password: password.value,
      options: {
        data: { full_name: fullName.value }
      }
    })
    if (error) {
      message.value = error.message
      messageType.value = 'error'
    } else {
      message.value = 'Account created! Check your email to confirm before signing in.'
      messageType.value = 'success'
    }
  }

  loading.value = false
}

async function resetPassword() {
  if (!email.value) {
    message.value = 'Enter your email address above first.'
    messageType.value = 'error'
    return
  }
  const { error } = await supabase.auth.resetPasswordForEmail(email.value, {
    redirectTo: `${window.location.origin}/auth/confirm`
  })
  message.value = error
    ? error.message
    : 'Password reset email sent — check your inbox.'
  messageType.value = error ? 'error' : 'success'
}
</script>

<style scoped>
.auth-container {
  min-height: 100vh;
  display: flex;
  align-items: center;
  justify-content: center;
  background: var(--parchment);
  padding: 1rem;
  position: relative;
}
.auth-card {
  background: var(--surface);
  border: 1px solid var(--border);
  border-radius: 16px;
  padding: 2.5rem 2rem;
  width: 100%;
  max-width: 420px;
  box-shadow: 0 4px 24px var(--shadow-md);
}
.auth-header {
  text-align: center;
  margin-bottom: 2rem;
}
.auth-header h1 {
  font-family: 'Lora', serif;
  font-size: 1.3rem;
  font-weight: 600;
  color: var(--ink);
  margin-bottom: 0.25rem;
}
.auth-header p {
  color: var(--ink-muted);
  font-size: 0.9rem;
}
.auth-tabs {
  display: flex;
  border-bottom: 2px solid var(--border);
  margin-bottom: 1.5rem;
}
.auth-tabs button {
  flex: 1;
  padding: 0.65rem;
  background: none;
  border: none;
  cursor: pointer;
  font-size: 0.95rem;
  color: var(--ink-muted);
  border-bottom: 2px solid transparent;
  margin-bottom: -2px;
  transition: all 0.2s;
}
.auth-tabs button.active {
  color: var(--sage);
  border-bottom-color: var(--sage);
  font-weight: 600;
}
.auth-message {
  padding: 0.75rem 1rem;
  border-radius: 8px;
  margin-bottom: 1rem;
  font-size: 0.9rem;
}
.auth-message.error { background: var(--terracotta-light); color: var(--terracotta); }
.auth-message.success { background: var(--sage-light); color: var(--sage); }
.auth-message.info { background: var(--gold-light); color: var(--gold); }
.form-group {
  margin-bottom: 1rem;
}
.form-group label {
  display: block;
  font-size: 0.85rem;
  font-weight: 600;
  color: var(--ink-soft);
  margin-bottom: 0.4rem;
}
.form-group input {
  width: 100%;
  padding: 0.7rem 0.9rem;
  border: 1.5px solid var(--border);
  border-radius: 8px;
  font-size: 0.95rem;
  outline: none;
  background: var(--parchment);
  color: var(--ink);
  transition: border-color 0.2s;
  box-sizing: border-box;
}
.form-group input:focus { border-color: var(--sage); }
.auth-submit {
  width: 100%;
  padding: 0.8rem;
  background: var(--sage);
  color: white;
  border: none;
  border-radius: 8px;
  font-size: 1rem;
  font-weight: 600;
  cursor: pointer;
  margin-top: 0.5rem;
  transition: background 0.2s;
}
.auth-submit:hover { filter: brightness(0.92); }
.auth-submit:disabled { background: var(--ink-muted); cursor: not-allowed; }
.auth-forgot {
  text-align: center;
  margin-top: 1rem;
  font-size: 0.85rem;
}
.auth-forgot a {
  color: var(--sage);
  cursor: pointer;
  text-decoration: underline;
}

@media (max-width: 480px) {
  .auth-card {
    padding: 2rem 1.25rem;
    border-radius: 12px;
  }
  .auth-header h1 {
    font-size: 1.15rem;
  }
}
</style>
