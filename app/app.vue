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
  title: 'Jesus Transforms Lives — Session Builder',
  description: 'Build, structure, and save your Bible study sessions from cell group outlines to full sermon notes.',
  ogTitle: 'Jesus Transforms Lives — Session Builder',
  ogDescription: 'Build, structure, and save your Bible study sessions from cell group outlines to full sermon notes.',
})
</script>

<template>
  <UApp>
    <NuxtPage />
  </UApp>
</template>
