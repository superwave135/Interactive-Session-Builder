<template>
  <div class="landing">

    <!-- Nav -->
    <header class="landing-nav">
      <span class="nav-brand">Gatherly</span>
      <div class="nav-right">
        <UColorModeButton />
        <template v-if="user">
          <NuxtLink to="/dashboard" class="nav-dashboard">Dashboard</NuxtLink>
          <button class="nav-signout" @click="signOut">Sign Out</button>
        </template>
        <NuxtLink v-else to="/login" class="nav-signin">Sign In</NuxtLink>
      </div>
    </header>

    <!-- Hero -->
    <section class="hero">
      <div class="hero-content">
        <p class="hero-tagline">His Word, Beautifully Prepared</p>
        <h1 class="hero-brand">Gatherly</h1>
        <div class="hero-divider"><span>✦</span></div>
        <h2 class="hero-title">Craft meaningful sessions with ease</h2>
        <p class="hero-desc">
          Build and share with your creativity — from outlines to full discussion notes
        </p>
        <div class="hero-actions">
          <NuxtLink to="/login" class="btn-primary">Get Started Free</NuxtLink>
          <NuxtLink to="/login" class="btn-ghost">Sign In</NuxtLink>
        </div>
      </div>

      <div class="hero-scroll-hint" aria-hidden="true">↓</div>
    </section>

    <!-- Scripture strip -->
    <div class="scripture-strip">
      <span>"For the word of God is alive and active." — Hebrews 4:12</span>
    </div>

    <!-- Features -->
    <section class="features">
      <p class="features-label">Everything you need</p>
      <div class="features-grid">
        <div class="feature-card">
          <h3>Visual Builder</h3>
          <p>Drag-and-drop blocks for scripture, reflection questions, discussion points, and more.</p>
        </div>
        <div class="feature-card">
          <h3>Cloud Saved</h3>
          <p>Sessions are saved securely — access and edit them from any device, any time.</p>
        </div>
        <div class="feature-card">
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
/* ── BASE ── */
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
  position: sticky;
  top: 0;
  z-index: 50;
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 0.9rem 2.5rem;
  background: color-mix(in srgb, var(--parchment-dark) 90%, transparent);
  backdrop-filter: blur(12px);
  border-bottom: 1px solid var(--border);
}
.nav-brand {
  font-family: 'Kaushan Script', cursive;
  font-size: 1.5rem;
  letter-spacing: 0.01em;
  color: var(--gold);
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
  padding: 0.4rem 1.1rem;
  border: 1.5px solid var(--border-dark);
  border-radius: 8px;
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
  padding: 0.4rem 1.1rem;
  background: var(--sage);
  border-radius: 8px;
  transition: filter 0.2s;
}
.nav-dashboard:hover { filter: brightness(0.9); }
.nav-signout {
  font-size: 0.875rem;
  font-weight: 500;
  color: var(--ink-muted);
  background: none;
  border: 1.5px solid var(--border);
  border-radius: 8px;
  padding: 0.4rem 0.9rem;
  cursor: pointer;
  transition: all 0.2s;
}
.nav-signout:hover { color: var(--terracotta); border-color: var(--terracotta); }

/* ── HERO ── */
.hero {
  flex: 1;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  min-height: calc(100vh - 57px);
  padding: 4rem 2rem 6rem;
  text-align: center;
  position: relative;
  background:
    radial-gradient(ellipse 70% 50% at 50% 40%, var(--gold-light) 0%, transparent 70%),
    var(--parchment);
}
.hero-content {
  max-width: 680px;
  display: flex;
  flex-direction: column;
  align-items: center;
}
.hero-tagline {
  font-family: 'Lora', serif;
  font-style: italic;
  font-size: 0.95rem;
  color: var(--ink-muted);
  letter-spacing: 0.08em;
  margin-bottom: 1rem;
}
.hero-brand {
  font-family: 'Kaushan Script', cursive;
  font-size: clamp(4rem, 12vw, 7rem);
  color: var(--gold);
  line-height: 1;
  margin: 0 0 1rem;
}
.hero-divider {
  display: flex;
  align-items: center;
  gap: 0.75rem;
  width: 100%;
  max-width: 320px;
  margin-bottom: 1.75rem;
  color: var(--gold-soft);
}
.hero-divider::before,
.hero-divider::after {
  content: '';
  flex: 1;
  height: 1px;
  background: var(--gold-soft);
}
.hero-divider span {
  font-size: 0.75rem;
  color: var(--gold);
  flex-shrink: 0;
}
.hero-title {
  font-family: 'Lora', serif;
  font-size: clamp(1.6rem, 4vw, 2.4rem);
  font-weight: 600;
  line-height: 1.25;
  color: var(--ink);
  margin-bottom: 1.1rem;
}
.hero-desc {
  font-size: 1.05rem;
  color: var(--ink-soft);
  line-height: 1.7;
  margin-bottom: 2.5rem;
}
.hero-actions {
  display: flex;
  gap: 0.9rem;
  justify-content: center;
  flex-wrap: wrap;
}
.btn-primary {
  padding: 0.8rem 2rem;
  background: var(--sage);
  color: white;
  border-radius: 10px;
  font-size: 0.95rem;
  font-weight: 600;
  text-decoration: none;
  box-shadow: 0 2px 12px rgba(92, 122, 94, 0.35);
  transition: all 0.2s;
}
.btn-primary:hover {
  filter: brightness(0.92);
  box-shadow: 0 4px 18px rgba(92, 122, 94, 0.45);
  transform: translateY(-1px);
}
.btn-ghost {
  padding: 0.8rem 2rem;
  background: transparent;
  color: var(--ink);
  border: 1.5px solid var(--border-dark);
  border-radius: 10px;
  font-size: 0.95rem;
  font-weight: 500;
  text-decoration: none;
  transition: all 0.2s;
}
.btn-ghost:hover {
  background: var(--parchment-dark);
  border-color: var(--ink-muted);
}
.hero-scroll-hint {
  position: absolute;
  bottom: 2rem;
  left: 50%;
  transform: translateX(-50%);
  font-size: 1rem;
  color: var(--ink-muted);
  animation: bounce 2s infinite;
}
@keyframes bounce {
  0%, 100% { transform: translateX(-50%) translateY(0); }
  50% { transform: translateX(-50%) translateY(5px); }
}

/* ── SCRIPTURE ── */
.scripture-strip {
  width: 100%;
  background: var(--gold-light);
  border-top: 1px solid var(--gold-soft);
  border-bottom: 1px solid var(--gold-soft);
  text-align: center;
  padding: 1rem 2rem;
  font-family: 'Lora', serif;
  font-style: italic;
  font-size: 0.9rem;
  color: var(--ink-soft);
}

/* ── FEATURES ── */
.features {
  padding: 4.5rem 2rem;
  background: var(--parchment-dark);
  border-top: 1px solid var(--border);
}
.features-label {
  text-align: center;
  font-size: 0.75rem;
  font-weight: 600;
  letter-spacing: 0.12em;
  text-transform: uppercase;
  color: var(--ink-muted);
  margin-bottom: 2.5rem;
}
.features-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(220px, 1fr));
  gap: 1.5rem;
  max-width: 900px;
  margin: 0 auto;
}
.feature-card {
  background: var(--parchment);
  border: 1px solid var(--border);
  border-radius: var(--radius-lg);
  padding: 2rem 1.75rem;
  box-shadow: 0 1px 6px var(--shadow);
  transition: all 0.25s;
}
.feature-card:hover {
  box-shadow: 0 6px 24px var(--shadow-md);
  transform: translateY(-2px);
}
.feature-card h3 {
  font-family: 'Lora', serif;
  font-size: 1.05rem;
  font-weight: 600;
  color: var(--ink);
  margin-bottom: 0.5rem;
}
.feature-card p {
  font-size: 0.875rem;
  color: var(--ink-soft);
  line-height: 1.65;
}

/* ── FOOTER ── */
.landing-footer {
  text-align: center;
  padding: 1.5rem 1rem;
  font-size: 0.8rem;
  color: var(--ink-muted);
  background: var(--parchment-dark);
  border-top: 1px solid var(--border);
}

/* ── MOBILE ── */
@media (max-width: 600px) {
  .landing-nav { padding: 0.85rem 1.25rem; }
  .nav-brand { font-size: 1.2rem; }
  .hero { padding: 3rem 1.5rem 5rem; min-height: calc(100svh - 52px); }
  .hero-actions { flex-direction: column; align-items: stretch; }
  .btn-primary, .btn-ghost { text-align: center; }
  .features { padding: 3rem 1.25rem; }
}
</style>
