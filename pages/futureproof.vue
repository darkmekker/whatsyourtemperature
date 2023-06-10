<template>
  <main class="pt-6 md:pt-16">
    <header class="w-full mx-auto mb-4 pt-20 md:pt-0 md:px-24 relative">
      <img src="@/assets/icons/contentLogo.svg" alt="Promeos Logo" class="absolute top-0 -left-0" />
      <h1 class="max-w-2xl">{{ content.title }}</h1>
    </header>
    <section class="w-full max-w-5xl mx-auto">
      <nuxt-content :document="content" />
    </section>
  </main>
</template>

<script>
export default {
  async asyncData({ $content, error }) {
    let content
    try {
      content = await $content('futureproof/_index').fetch()
    } catch (e) {
      error({ message: 'Contact data not found' })
    }
    return { content }
  },
}
</script>
