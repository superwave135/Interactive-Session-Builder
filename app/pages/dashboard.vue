<template>
  <div class="dashboard">
    <!-- Header -->
    <header class="dash-header">
      <NuxtLink
        to="/"
        class="dash-brand"
      >Gatherly</NuxtLink>
      <div class="dash-user">
        <UColorModeButton />
        <span>{{ user?.email }}</span>
        <button
          class="btn-signout"
          @click="signOut"
        >
          Sign Out
        </button>
      </div>
    </header>

    <!-- Main -->
    <main class="dash-main">
      <div class="dash-top">
        <h2>My Sessions</h2>
        <button
          class="btn-new"
          @click="newSession"
        >
          + New Session
        </button>
      </div>

      <!-- Loading -->
      <div
        v-if="loading"
        class="dash-empty"
      >
        Loading your sessions…
      </div>

      <!-- Empty state -->
      <div
        v-else-if="sessions.length === 0"
        class="dash-empty"
      >
        <p>You don't have any sessions yet.</p>
        <button
          class="btn-new"
          @click="newSession"
        >
          Create your first session
        </button>
      </div>

      <!-- Session cards -->
      <div
        v-else
        class="session-grid"
      >
        <div
          v-for="session in sessions"
          :key="session.id"
          class="session-card"
        >
          <div class="session-card-body">
            <h3>{{ session.title }}</h3>
            <p class="session-date">
              {{ formatDate(session.updated_at) }}
            </p>
          </div>
          <div class="session-card-actions">
            <button
              class="btn-edit"
              @click="openSession(session.id)"
            >
              Open
            </button>
            <button
              class="btn-delete"
              @click="confirmDelete(session)"
            >
              Delete
            </button>
          </div>
        </div>
      </div>
    </main>

    <!-- Delete confirmation modal -->
    <div
      v-if="sessionToDelete"
      class="modal-overlay"
      @click.self="sessionToDelete = null"
    >
      <div class="modal">
        <h3>Delete session?</h3>
        <p>"{{ sessionToDelete.title }}" will be permanently deleted.</p>
        <div class="modal-actions">
          <button
            class="btn-cancel"
            @click="sessionToDelete = null"
          >
            Cancel
          </button>
          <button
            class="btn-confirm-delete"
            @click="deleteSession"
          >
            Delete
          </button>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
definePageMeta({ middleware: 'auth', layout: false })

const supabase = useSupabaseClient()
const user = useSupabaseUser()
const sessions = ref([])
const loading = ref(true)
const sessionToDelete = ref(null)

onMounted(async () => {
  const { data, error } = await supabase
    .from('sessions')
    .select('*')
    .order('updated_at', { ascending: false })

  if (!error) sessions.value = data
  loading.value = false
})

function formatDate(ts) {
  return new Date(ts).toLocaleDateString('en-SG', {
    day: 'numeric', month: 'short', year: 'numeric'
  })
}

function newSession() {
  navigateTo('/edit')
}

function openSession(id) {
  navigateTo(`/edit?session=${id}`)
}

function confirmDelete(session) {
  sessionToDelete.value = session
}

async function deleteSession() {
  const { error } = await supabase
    .from('sessions')
    .delete()
    .eq('id', sessionToDelete.value.id)

  if (!error) {
    sessions.value = sessions.value.filter(s => s.id !== sessionToDelete.value.id)
  }
  sessionToDelete.value = null
}

async function signOut() {
  await supabase.auth.signOut()
  navigateTo('/')
}
</script>

<style scoped>
.dashboard {
  min-height: 100vh;
  background: var(--parchment);
  color: var(--ink);
}
.dash-header {
  background: var(--surface);
  padding: 1rem 2rem;
  display: flex;
  justify-content: space-between;
  align-items: center;
  box-shadow: 0 2px 8px var(--shadow);
  border-bottom: 1px solid var(--border);
}
.dash-brand {
  font-family: 'Lora', serif;
  font-size: 1.1rem;
  font-weight: 600;
  color: var(--ink);
  text-decoration: none;
  transition: color 0.15s;
}
.dash-brand:hover {
  color: var(--sage);
}
.dash-user {
  display: flex;
  align-items: center;
  gap: 1rem;
  font-size: 0.85rem;
  color: var(--ink-soft);
}
.btn-signout {
  padding: 0.4rem 0.9rem;
  background: none;
  border: 1.5px solid var(--border);
  border-radius: 6px;
  cursor: pointer;
  font-size: 0.85rem;
  color: var(--ink-soft);
  transition: all 0.2s;
}
.btn-signout:hover {
  border-color: var(--terracotta);
  color: var(--terracotta);
}
.dash-main {
  max-width: 960px;
  margin: 0 auto;
  padding: 2rem 1.5rem;
}
.dash-top {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 1.5rem;
}
.dash-top h2 {
  font-size: 1.4rem;
  font-family: 'Lora', serif;
  color: var(--ink);
}
.btn-new {
  padding: 0.6rem 1.2rem;
  background: var(--sage);
  color: white;
  border: none;
  border-radius: 8px;
  font-size: 0.95rem;
  font-weight: 600;
  cursor: pointer;
  transition: background 0.2s;
}
.btn-new:hover { background: var(--color-green-600); }
.dash-empty {
  text-align: center;
  padding: 4rem 1rem;
  color: var(--ink-muted);
}
.dash-empty p { margin-bottom: 1rem; }
.session-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(280px, 1fr));
  gap: 1.25rem;
}
.session-card {
  background: var(--surface);
  border: 1px solid var(--border);
  border-radius: 12px;
  box-shadow: 0 2px 8px var(--shadow);
  display: flex;
  flex-direction: column;
  overflow: hidden;
  transition: box-shadow 0.2s;
}
.session-card:hover { box-shadow: 0 4px 16px var(--shadow-md); }
.session-card-body {
  padding: 1.25rem;
  flex: 1;
}
.session-card-body h3 {
  font-size: 1rem;
  font-weight: 700;
  color: var(--ink);
  margin-bottom: 0.25rem;
}
.session-date {
  font-size: 0.78rem;
  color: var(--ink-muted);
  margin-bottom: 0.5rem;
}
.session-desc {
  font-size: 0.88rem;
  color: var(--ink-soft);
}
.session-card-actions {
  display: flex;
  border-top: 1px solid var(--border);
}
.session-card-actions button {
  flex: 1;
  padding: 0.7rem;
  border: none;
  cursor: pointer;
  font-size: 0.88rem;
  font-weight: 600;
  transition: background 0.2s;
  background: var(--parchment-dark);
}
.btn-edit { color: var(--sage); }
.btn-edit:hover { background: var(--sage-light); }
.btn-delete {
  color: var(--terracotta);
  border-left: 1px solid var(--border) !important;
}
.btn-delete:hover { background: var(--terracotta-light); }
.modal-overlay {
  position: fixed;
  inset: 0;
  background: rgba(0,0,0,0.4);
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 100;
}
.modal {
  background: var(--surface);
  border: 1px solid var(--border);
  border-radius: 12px;
  padding: 2rem;
  max-width: 380px;
  width: 90%;
  box-shadow: 0 8px 32px var(--shadow-md);
}
.modal h3 { margin-bottom: 0.5rem; color: var(--ink); font-family: 'Lora', serif; }
.modal p { color: var(--ink-soft); margin-bottom: 1.5rem; font-size: 0.9rem; }
.modal-actions { display: flex; gap: 0.75rem; justify-content: flex-end; }
.btn-cancel {
  padding: 0.6rem 1.2rem;
  background: var(--parchment-dark);
  border: 1px solid var(--border);
  border-radius: 8px;
  cursor: pointer;
  font-weight: 600;
  color: var(--ink-soft);
}
.btn-confirm-delete {
  padding: 0.6rem 1.2rem;
  background: var(--terracotta);
  color: white;
  border: none;
  border-radius: 8px;
  cursor: pointer;
  font-weight: 600;
}
.btn-confirm-delete:hover { filter: brightness(0.9); }

@media (max-width: 600px) {
  .dash-header {
    padding: 0.75rem 1rem;
    flex-direction: column;
    align-items: flex-start;
    gap: 0.6rem;
  }
  .dash-user {
    width: 100%;
    justify-content: space-between;
  }
  .dash-main {
    padding: 1.25rem 1rem;
  }
  .dash-top {
    flex-direction: column;
    align-items: flex-start;
    gap: 0.75rem;
  }
  .btn-new {
    width: 100%;
    text-align: center;
  }
  .session-grid {
    grid-template-columns: 1fr;
  }
  .modal {
    padding: 1.5rem 1.25rem;
  }
  .modal-actions {
    flex-direction: column-reverse;
  }
  .btn-cancel,
  .btn-confirm-delete {
    width: 100%;
    text-align: center;
  }
}
</style>
