<template>
  <nuxt-layout :name="false">
    <nuxt-page />
  </nuxt-layout>
</template>

<script setup>
definePageMeta({
  layout: 'dashboard',
  middleware: [
    (to, form) => {
      if (to.path === '/dashboard') {
        return navigateTo('/dashboard/posts-public');
      }
    }
  ]
});
</script>
