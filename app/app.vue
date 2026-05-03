<script setup>
useHead({
  meta: [
    { name: 'viewport', content: 'width=device-width, initial-scale=1' }
  ],
  link: [
    { rel: 'icon', href: '/favicon.ico' }
  ],
  htmlAttrs: {
    lang: 'en'
  }
})

useSeoMeta({
  title: 'Gatherly — Session Builder',
  description: 'Build, structure, and save your Bible study sessions from cell group outlines to full sermon notes.',
  ogTitle: 'Gatherly — Session Builder',
  ogDescription: 'Build, structure, and save your Bible study sessions from cell group outlines to full sermon notes.'
})
</script>

<template>
  <UApp>
    <NuxtPage />
  </UApp>
</template>
