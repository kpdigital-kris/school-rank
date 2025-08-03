<script setup>
useHead({
  htmlAttrs: { lang: "en-US" },
  title: "School Rank",
  meta: [
    {
      name: "description",
      content:
        "School rank is a website designed to help parents understand how schools are performing.",
    },
  ],
  bodyAttrs: {
    class: 'font-display'
  }
});
</script>

<template>
  <NuxtLayout>
    <NuxtPage />
  </NuxtLayout>
</template>

<style>
@import 'bulma/css/bulma.min.css';
</style>