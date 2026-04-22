<template>
  <nav class="nav-tabs">
    <NuxtLink
      to="/"
      class="nav-tab"
      exact-active-class="nav-tab--active"
    >Home</NuxtLink>
    <template v-if="user">
      <NuxtLink
        to="/dashboard"
        class="nav-tab"
        active-class="nav-tab--active"
      >Dashboard</NuxtLink>
      <NuxtLink
        to="/edit"
        class="nav-tab"
        active-class="nav-tab--active"
      >Builder</NuxtLink>
    </template>
  </nav>
</template>

<script setup>
const user = useSupabaseUser()
</script>

<style scoped>
.nav-tabs {
  display: flex;
  align-items: center;
  gap: 0.15rem;
}
.nav-tab {
  padding: 0.35rem 0.85rem;
  border-radius: 6px;
  font-size: 0.875rem;
  font-weight: 500;
  text-decoration: none;
  color: var(--ink-muted);
  transition: color 0.15s, background 0.15s;
}
.nav-tab:hover {
  color: var(--ink);
  background: var(--sage-light);
}
.nav-tab--active {
  color: var(--sage);
  background: var(--sage-light);
  font-weight: 600;
}
</style>
