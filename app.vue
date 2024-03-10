<template>
  <NuxtLayout>
    <v-app>
      <NuxtPage />
    </v-app>
  </NuxtLayout>
</template>

<script setup>
useHead({
  title: "Trello Clone",
  meta: [
    {
      name: "Trello Clone with Nuxt",
      content: "This is a Trello Clone for the Airtribe take home project",
    },
  ],
  link: [{ rel: "icon", type: "image/x-icon", href: "/favicon.png" }],
});
</script>
