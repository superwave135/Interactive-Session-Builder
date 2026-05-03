<template>
  <div class="landing">
    <!-- Top nav -->
    <header class="landing-nav">
      <span class="nav-brand">Gatherly</span>
      <div class="nav-right">
        <UColorModeButton />
        <template v-if="user">
          <NuxtLink
            to="/dashboard"
            class="nav-dashboard"
          >Dashboard</NuxtLink>
          <button
            class="nav-signout"
            @click="signOut"
          >
            Sign Out
          </button>
        </template>
        <NuxtLink
          v-else
          to="/login"
          class="nav-signin"
        >Sign In</NuxtLink>
      </div>
    </header>

    <!-- Hero -->
    <section class="hero">
      <div class="hero-inner">
        <p class="hero-eyebrow">
          His Word, Beautifully Prepared
        </p>
        <h1 class="hero-title">
          Craft meaningful<br>
          <em>sessions with ease</em>
        </h1>
        <p class="hero-desc">
          Build, structure, and save your Bible Discussion Session — from cell group outlines
          to full sermon notes — all in one place.
        </p>
        <div class="hero-actions">
          <NuxtLink
            to="/login"
            class="btn-primary"
          >Get Started</NuxtLink>
          <NuxtLink
            to="/login"
            class="btn-secondary"
          >Sign In</NuxtLink>
        </div>
      </div>

      <!-- decorative scripture strip -->
      <div class="scripture-strip">
        <span>"For the word of God is alive and active." — Hebrews 4:12</span>
      </div>
    </section>

    <!-- Feature cards -->
    <section class="features">
      <div class="features-grid">
        <div class="feature-card">
          <div class="feature-icon">
            📖
          </div>
          <h3>Visual Builder</h3>
          <p>Drag-and-drop blocks for scripture, reflection questions, discussion points, and more.</p>
        </div>
        <div class="feature-card">
          <div class="feature-icon">
            ☁️
          </div>
          <h3>Cloud Saved</h3>
          <p>Your sessions are saved securely to the cloud — access and edit them from anywhere.</p>
        </div>
        <div class="feature-card">
          <div class="feature-icon">
            🖨️
          </div>
          <h3>Print Ready</h3>
          <p>Export a clean, formatted handout for your group with a single click.</p>
        </div>
      </div>
    </section>

    <!-- Footer -->
    <footer class="landing-footer">
      <p>© {{ new Date().getFullYear() }} Gatherly · Built with love for the community</p>
    </footer>
  </div>
</template>

<script setup>
definePageMeta({ layout: false })

const user = useSupabaseUser()
const supabase = useSupabaseClient()

async function signOut() {
  await supabase.auth.signOut()
}
</script>

<style scoped>
.landing {
  min-height: 100vh;
  display: flex;
  flex-direction: column;
  background: var(--parchment);
  color: var(--ink);
  font-family: 'DM Sans', sans-serif;
}

/* ── NAV ── */
.landing-nav {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 1.1rem 2rem;
  background: var(--parchment-dark);
  color: var(--ink);
  border-bottom: 1px solid var(--border);
}
.nav-brand {
  font-family: 'Lora', serif;
  font-size: 1rem;
  font-weight: 600;
  letter-spacing: 0.01em;
  color: var(--ink);
}
.nav-right {
  display: flex;
  align-items: center;
  gap: 0.75rem;
}
.nav-signin {
  font-size: 0.875rem;
  font-weight: 500;
  color: var(--ink);
  text-decoration: none;
  padding: 0.4rem 1rem;
  border: 1.5px solid var(--border-dark);
  border-radius: 6px;
  transition: all 0.2s;
}
.nav-signin:hover {
  background: var(--sage-light);
  border-color: var(--sage);
  color: var(--sage);
}
.nav-dashboard {
  font-size: 0.875rem;
  font-weight: 600;
  color: white;
  text-decoration: none;
  padding: 0.4rem 1rem;
  background: var(--sage);
  border-radius: 6px;
  transition: filter 0.2s;
}
.nav-dashboard:hover { filter: brightness(0.92); }
.nav-signout {
  font-size: 0.875rem;
  font-weight: 500;
  color: var(--ink-muted);
  background: none;
  border: 1.5px solid var(--border);
  border-radius: 6px;
  padding: 0.4rem 0.85rem;
  cursor: pointer;
  transition: all 0.2s;
}
.nav-signout:hover {
  color: var(--terracotta);
  border-color: var(--terracotta);
}

/* ── HERO ── */
.hero {
  flex: 1;
  display: flex;
  flex-direction: column;
  align-items: center;
  padding: 5rem 1.5rem 0;
}
.hero-inner {
  max-width: 640px;
  text-align: center;
}
.hero-eyebrow {
  font-size: 0.8rem;
  font-weight: 600;
  letter-spacing: 0.12em;
  text-transform: uppercase;
  color: var(--gold);
  margin-bottom: 1rem;
}
.hero-title {
  font-family: 'Lora', serif;
  font-size: clamp(2.2rem, 5vw, 3.2rem);
  font-weight: 600;
  line-height: 1.2;
  color: var(--ink);
  margin-bottom: 1.25rem;
}
.hero-title em {
  font-style: italic;
  color: var(--sage);
}
.hero-desc {
  font-size: 1.05rem;
  color: var(--ink-soft);
  line-height: 1.65;
  margin-bottom: 2rem;
}
.hero-actions {
  display: flex;
  gap: 0.85rem;
  justify-content: center;
  flex-wrap: wrap;
}
.btn-primary {
  padding: 0.75rem 1.75rem;
  background: var(--sage);
  color: white;
  border-radius: 8px;
  font-size: 0.95rem;
  font-weight: 600;
  text-decoration: none;
  transition: background 0.2s;
}
.btn-primary:hover { background: var(--color-green-600); }
.btn-secondary {
  padding: 0.75rem 1.75rem;
  background: transparent;
  color: var(--ink);
  border: 1.5px solid var(--border-dark);
  border-radius: 8px;
  font-size: 0.95rem;
  font-weight: 600;
  text-decoration: none;
  transition: all 0.2s;
}
.btn-secondary:hover {
  background: var(--parchment-dark);
  border-color: var(--ink-muted);
}
.scripture-strip {
  margin-top: 3.5rem;
  width: 100%;
  background: var(--gold-light);
  border-top: 1px solid var(--gold-soft);
  border-bottom: 1px solid var(--gold-soft);
  text-align: center;
  padding: 0.85rem 1.5rem;
  font-family: 'Lora', serif;
  font-style: italic;
  font-size: 0.9rem;
  color: var(--ink-soft);
}

/* ── FEATURES ── */
.features {
  padding: 3.5rem 1.5rem;
  background: var(--parchment-dark);
  border-top: 1px solid var(--border);
}
.features-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(220px, 1fr));
  gap: 1.5rem;
  max-width: 860px;
  margin: 0 auto;
}
.feature-card {
  background: var(--parchment);
  border: 1px solid var(--border);
  border-radius: var(--radius-lg);
  padding: 1.75rem 1.5rem;
  box-shadow: 0 2px 8px var(--shadow);
  transition: box-shadow 0.2s;
}
.feature-card:hover { box-shadow: 0 4px 16px var(--shadow-md); }
.feature-icon {
  font-size: 1.75rem;
  margin-bottom: 0.85rem;
}
.feature-card h3 {
  font-family: 'Lora', serif;
  font-size: 1rem;
  font-weight: 600;
  color: var(--ink);
  margin-bottom: 0.5rem;
}
.feature-card p {
  font-size: 0.875rem;
  color: var(--ink-soft);
  line-height: 1.6;
}

/* ── MOBILE ── */
@media (max-width: 600px) {
  .landing-nav {
    padding: 0.85rem 1.25rem;
  }
  .nav-brand {
    font-size: 0.875rem;
  }
  .hero {
    padding: 3rem 1.25rem 0;
  }
  .hero-actions {
    flex-direction: column;
    align-items: stretch;
  }
  .btn-primary,
  .btn-secondary {
    text-align: center;
  }
  .scripture-strip {
    margin-top: 2.5rem;
    font-size: 0.82rem;
  }
  .features {
    padding: 2.5rem 1.25rem;
  }
}

/* ── FOOTER ── */
.landing-footer {
  text-align: center;
  padding: 1.25rem 1rem;
  font-size: 0.8rem;
  color: var(--ink-muted);
  background: var(--parchment-dark);
  border-top: 1px solid var(--border);
}
</style>
