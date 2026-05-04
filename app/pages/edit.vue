<template>
  <div class="builder-page">
    <!-- Thin top bar -->
    <div class="builder-bar">
      <button
        class="btn-back"
        @click="goBack"
      >
        ← Dashboard
      </button>
      <div class="bar-right">
        <UColorModeButton />
        <div class="bar-mode-group">
          <button
            :class="['bar-mode-btn', { active: currentMode === 'build' }]"
            @click="setMode('build')"
          >
            Build
          </button>
          <button
            :class="['bar-mode-btn', { active: currentMode === 'member' }]"
            @click="setMode('member')"
          >
            View
          </button>
          <button
            class="bar-mode-btn bar-mode-present"
            @click="triggerPresent"
          >
            Present
          </button>
        </div>
        <div class="bar-mode-group bar-file-group">
          <button
            class="bar-mode-btn"
            @click="triggerUpload"
          >
            Upload
          </button>
          <button
            class="bar-mode-btn"
            @click="triggerDownload"
          >
            Download
          </button>
          <button
            class="bar-mode-btn bar-mode-save"
            @click="saveSession"
          >
            Save
          </button>
        </div>
      </div>
    </div>

    <!-- Save modal -->
    <div
      v-if="showSaveModal"
      class="save-overlay"
      @click.self="showSaveModal = false"
    >
      <div class="save-modal">
        <h3 class="save-modal-title">
          Save Session
        </h3>
        <p class="save-modal-desc">
          Give this session a name so you can find it later.
        </p>
        <input
          ref="nameInput"
          v-model="sessionNameInput"
          class="save-modal-input"
          type="text"
          placeholder="e.g. Week 3 — Romans Study"
          maxlength="80"
          @keyup.enter="confirmSave"
        >
        <div class="save-modal-actions">
          <button
            class="save-btn-cancel"
            @click="showSaveModal = false"
          >
            Cancel
          </button>
          <button
            class="save-btn-confirm"
            :disabled="saving"
            @click="confirmSave"
          >
            {{ saving ? 'Saving…' : 'Save' }}
          </button>
        </div>
      </div>
    </div>

    <!-- Builder iframe -->
    <iframe
      ref="builderFrame"
      src="/builder/index.html"
      class="builder-frame"
      @load="onFrameLoad"
    />
  </div>
</template>

<script setup>
definePageMeta({ middleware: 'auth', layout: false })

const supabase = useSupabaseClient()
const colorMode = useColorMode()
const route = useRoute()
const builderFrame = ref(null)
const nameInput = ref(null)

const showSaveModal = ref(false)
const sessionNameInput = ref('')
const saving = ref(false)
let pendingBlocks = []
let pendingUserId = ''

// Dark-mode CSS variable overrides to inject into the iframe document
const DARK_VARS = `
  :root {
    --parchment:        #1e1a14;
    --parchment-dark:   #161210;
    --surface:          #261f17;
    --ink:              #f0eadf;
    --ink-soft:         #c8b8a0;
    --ink-muted:        #7a6a58;
    --sage:             #7da882;
    --sage-light:       #1a2a1b;
    --sage-mid:         #3a5a3c;
    --gold:             #d4a520;
    --gold-light:       #2a2010;
    --gold-soft:        #3a2e14;
    --terracotta:       #c06a3c;
    --terracotta-light: #2a1a10;
    --border:           #3e3228;
    --border-dark:      #4e4030;
    --shadow:           rgba(0,0,0,0.35);
    --shadow-md:        rgba(0,0,0,0.50);
    color-scheme: dark;
  }
`

function applyThemeToIframe() {
  const doc = builderFrame.value?.contentDocument
  if (!doc) return

  // Remove any previously injected theme style
  doc.getElementById('__nuxt-theme')?.remove()

  if (colorMode.value === 'dark') {
    const style = doc.createElement('style')
    style.id = '__nuxt-theme'
    style.textContent = DARK_VARS
    doc.head.appendChild(style)
    doc.documentElement.classList.add('dark')
  } else {
    doc.documentElement.classList.remove('dark')
  }
}

async function onFrameLoad() {
  applyThemeToIframe()

  const sessionId = route.query.session
  if (!sessionId) return

  const { data, error } = await supabase
    .from('sessions')
    .select('blocks')
    .eq('id', sessionId)
    .single()

  if (error || !data?.blocks) return

  const rawBlocks = typeof data.blocks === 'string' ? JSON.parse(data.blocks) : data.blocks
  const win = builderFrame.value?.contentWindow
  if (!win) return
  win.blocks = rawBlocks
  win.renderCanvas()
}

// Re-apply theme whenever color mode changes after the iframe is already loaded
watch(() => colorMode.value, () => {
  applyThemeToIframe()
})

function onIframeMessage(e) {
  if (e.data === 'save-session') saveSession()
}
onMounted(() => window.addEventListener('message', onIframeMessage))
onUnmounted(() => window.removeEventListener('message', onIframeMessage))

const currentMode = ref('build')

function setMode(mode) {
  currentMode.value = mode
  builderFrame.value?.contentWindow?.switchMode(mode)
}
function triggerPresent() {
  builderFrame.value?.contentWindow?.openPresentation()
}
function triggerUpload() {
  builderFrame.value?.contentWindow?.loadFromFile()
}
function triggerDownload() {
  builderFrame.value?.contentWindow?.saveTemplate()
}

function goBack() {
  navigateTo('/dashboard')
}

async function saveSession() {
  const { data: { session } } = await supabase.auth.getSession()
  if (!session) {
    alert('Session expired — please log in again')
    return navigateTo('/login')
  }
  pendingUserId = session.user.id

  try {
    pendingBlocks = builderFrame.value?.contentWindow?.blocks || []
  } catch (e) {
    console.warn('Could not read builder state:', e)
    pendingBlocks = []
  }

  // Pre-fill name from cellgroup block or existing session title
  const nameBlock = pendingBlocks.find(b => b.type === 'cellgroup')
  sessionNameInput.value = nameBlock?.props?.name || ''

  showSaveModal.value = true
  await nextTick()
  nameInput.value?.focus()
}

async function confirmSave() {
  const title = sessionNameInput.value.trim() || 'Untitled Session'
  const sessionId = route.query.session
  saving.value = true

  if (sessionId) {
    const { error } = await supabase
      .from('sessions')
      .update({ title, blocks: pendingBlocks, updated_at: new Date().toISOString() })
      .eq('id', sessionId)
      .eq('user_id', pendingUserId)
    saving.value = false
    if (error) {
      alert('Save failed: ' + error.message)
      return
    }
  } else {
    const { data, error } = await supabase
      .from('sessions')
      .insert({ title, blocks: pendingBlocks, user_id: pendingUserId })
      .select()
      .single()
    saving.value = false
    if (error) {
      alert('Save failed: ' + error.message)
      return
    }
    navigateTo(`/edit?session=${data.id}`, { replace: true })
  }

  showSaveModal.value = false
}
</script>

<style scoped>
.builder-page {
  display: flex;
  flex-direction: column;
  height: 100vh;
  overflow: hidden;
}
.builder-bar {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 0.35rem 1rem;
  background: var(--parchment-dark);
  color: var(--ink);
  flex-shrink: 0;
  border-bottom: 1px solid var(--border);
  position: relative;
}
.btn-back {
  background: none;
  border: 1.5px solid var(--border-dark);
  color: var(--ink-soft);
  padding: 0.25rem 0.65rem;
  border-radius: 6px;
  cursor: pointer;
  font-size: 0.8rem;
  transition: all 0.2s;
}
.btn-back:hover {
  border-color: var(--ink-muted);
  color: var(--ink);
  background: var(--parchment);
}
.bar-right {
  display: flex;
  align-items: center;
  gap: 0.4rem;
}
.bar-mode-group {
  display: flex;
  background: var(--parchment);
  border-radius: 6px;
  padding: 2px;
  gap: 1px;
  border: 1px solid var(--border);
}
.bar-mode-btn {
  font-size: 11px;
  font-weight: 500;
  padding: 3px 10px;
  border-radius: 5px;
  border: none;
  cursor: pointer;
  color: var(--ink-muted);
  background: transparent;
  transition: all 0.18s;
}
.bar-mode-btn.active { background: var(--gold); color: var(--ink); }
.bar-mode-present { background: var(--sage) !important; color: #fff !important; }
.bar-mode-present:hover { filter: brightness(0.9); }
.bar-file-group { gap: 0; padding: 2px; }
.bar-file-group .bar-mode-btn { padding: 3px 8px; border-radius: 0; }
.bar-file-group .bar-mode-btn:first-child { border-radius: 4px 0 0 4px; }
.bar-file-group .bar-mode-btn:last-child { border-radius: 0 4px 4px 0; }
.bar-mode-save { background: var(--sage-mid) !important; color: var(--ink) !important; }
.bar-mode-save:hover { filter: brightness(0.9); }
.btn-save {
  background: none;
  border: 1.5px solid var(--border-dark);
  color: var(--ink-soft);
  padding: 0.4rem 1rem;
  border-radius: 6px;
  cursor: pointer;
  font-size: 0.85rem;
  font-weight: 600;
  transition: all 0.2s;
}
.btn-save:hover {
  border-color: var(--sage);
  color: var(--sage);
  background: var(--sage-light);
}

/* ── Save modal ── */
.save-overlay {
  position: fixed;
  inset: 0;
  background: rgba(0, 0, 0, 0.45);
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 200;
}
.save-modal {
  background: var(--surface);
  border: 1px solid var(--border);
  border-radius: var(--radius-lg);
  padding: 2rem 1.75rem;
  width: 90%;
  max-width: 420px;
  box-shadow: 0 8px 32px var(--shadow-md);
}
.save-modal-title {
  font-family: 'Lora', serif;
  font-size: 1.1rem;
  font-weight: 600;
  color: var(--ink);
  margin-bottom: 0.4rem;
}
.save-modal-desc {
  font-size: 0.875rem;
  color: var(--ink-muted);
  margin-bottom: 1.25rem;
}
.save-modal-input {
  width: 100%;
  padding: 0.65rem 0.9rem;
  border: 1.5px solid var(--border);
  border-radius: 8px;
  font-size: 0.95rem;
  font-family: 'DM Sans', sans-serif;
  background: var(--parchment);
  color: var(--ink);
  outline: none;
  box-sizing: border-box;
  transition: border-color 0.2s;
}
.save-modal-input:focus { border-color: var(--sage); }
.save-modal-actions {
  display: flex;
  justify-content: flex-end;
  gap: 0.75rem;
  margin-top: 1.25rem;
}
.save-btn-cancel {
  padding: 0.55rem 1.1rem;
  background: var(--parchment-dark);
  border: 1px solid var(--border);
  border-radius: 7px;
  font-size: 0.875rem;
  font-weight: 500;
  color: var(--ink-soft);
  cursor: pointer;
  transition: all 0.2s;
}
.save-btn-cancel:hover { border-color: var(--ink-muted); color: var(--ink); }
.save-btn-confirm {
  padding: 0.55rem 1.25rem;
  background: var(--sage);
  border: none;
  border-radius: 7px;
  font-size: 0.875rem;
  font-weight: 600;
  color: white;
  cursor: pointer;
  transition: filter 0.2s;
}
.save-btn-confirm:hover { filter: brightness(0.92); }
.save-btn-confirm:disabled { opacity: 0.6; cursor: not-allowed; }

@media (max-width: 600px) {
  .builder-bar {
    padding: 0.5rem 0.75rem;
  }
  .builder-title {
    font-size: 1.05rem;
  }
  .btn-back {
    padding: 0.35rem 0.6rem;
    font-size: 0.78rem;
  }
  .btn-save {
    padding: 0.35rem 0.7rem;
    font-size: 0.78rem;
  }
}

.builder-frame {
  flex: 1;
  border: none;
  width: 100%;
}
</style>
