<template>
  <main class="pt-6 md:pt-14">
    <header class="w-full max-w-3xl mx-auto mb-12 pt-20 md:pt-0 md:px-24 relative">
      <a href="/" class="absolute top-0 -left-0">
        <img src="@/assets/icons/contentLogo.svg" alt="Promeos Logo" />
      </a>
      <h1 class="max-w-2xl mb-8">{{ content.title }}</h1>
      <nuxt-content :document="content" />
    </header>

    <section class="w-full max-w-3xl mx-auto md:px-24">
      <h2 class="uppercase">Submission received &#10004;</h2>
      <nuxt-link :to="`/`" class="btn btn-primary text-sm md:text-base whitespace-nowrap mt-8">Home</nuxt-link>
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
