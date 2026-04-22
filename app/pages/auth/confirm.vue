<template>
  <div class="confirm-container">
    <UColorModeButton class="mode-toggle" />
    <div class="confirm-card">
      <div
        v-if="loading"
        class="loading"
      >
        Verifying…
      </div>
      <div
        v-else-if="error"
        class="error-state"
      >
        <h2>Something went wrong</h2>
        <p>{{ error }}</p>
        <a href="/login">Back to login</a>
      </div>
      <div
        v-else
        class="success-state"
      >
        <h2>✅ Email confirmed!</h2>
        <p>Your account is active. Redirecting you to the dashboard…</p>
      </div>
    </div>
  </div>
</template>

<script setup>
definePageMeta({ layout: false })

const supabase = useSupabaseClient()
const loading = ref(true)
const error = ref('')

onMounted(async () => {
  const { error: err } = await supabase.auth.getSession()
  if (err) {
    error.value = err.message
  } else {
    setTimeout(() => navigateTo('/dashboard'), 2000)
  }
  loading.value = false
})
</script>

<style scoped>
.confirm-container {
  min-height: 100vh;
  display: flex;
  align-items: center;
  justify-content: center;
  background: var(--parchment);
  position: relative;
}
.mode-toggle {
  position: absolute;
  top: 1rem;
  right: 1rem;
}
.confirm-card {
  background: var(--surface);
  border: 1px solid var(--border);
  border-radius: 16px;
  padding: 3rem 2rem;
  text-align: center;
  max-width: 400px;
  box-shadow: 0 4px 24px var(--shadow-md);
  color: var(--ink);
  font-family: 'DM Sans', sans-serif;
}
.confirm-card h2 {
  font-family: 'Lora', serif;
  margin-bottom: 0.5rem;
  color: var(--ink);
}
.confirm-card p {
  color: var(--ink-soft);
  margin-top: 0.5rem;
}
.loading { color: var(--ink-muted); }
.error-state h2 { color: var(--terracotta); }
.success-state h2 { color: var(--sage); }
a { color: var(--sage); }
</style>
