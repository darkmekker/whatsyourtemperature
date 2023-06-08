<template>
  <main>
    <section class="w-full max-w-5xl mx-auto">
      <h1 class="title">About</h1>
      <nuxt-content :document="content" />
    </section>
  </main>
</template>

<script>
export default {
  async asyncData({ $content, error }) {
    let content
    try {
      content = await $content('contact/_index').fetch()
    } catch (e) {
      error({ message: 'Contact data not found' })
    }
    return { content }
  },
}
</script>
