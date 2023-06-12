<template>
  <main class="pt-6 md:pt-14 px-0">
    <header class="w-full mx-auto mb-4 pt-20 md:pt-0 md:px-24 relative">
      <a href="/" class="absolute top-0 left-10">
        <img src="@/assets/icons/contentLogo.svg" alt="Promeos Logo" />
      </a>
      <h1 class="max-w-2xl text-center mx-auto">{{ content.title }}</h1>
    </header>
    <section class="w-full px-4">
      <div class="max-w-2xl mx-auto text-base text-center">
        <div class="my-2 md:my-8" v-html="content.intro" />
        <div class="" v-if="content.heroimage">
          <img :src="content.heroimage" :alt="content.title" class="w-full" />
        </div>
      </div>
    </section>
    <section class="blurb1 w-full py-10 md:py-20 bg-contain bg-no-repeat bg-right-center px-4">
      <div class="max-w-2xl mx-auto md:pr-72 text-center md:text-right">
        <h2 class="text-3xl md:text-5xl font-normal leading-tight mb-4 text-shadow">{{ content.blurb1.heading }}</h2>
        <div class="font-medium leading-snug" v-html="content.blurb1.text" />
        <div v-if="content.blurb1.moreButtonUrl && content.blurb1.moreButtonText">
          <nuxt-link
            :to="content.blurb1.moreButtonUrl"
            class="inline-block btn btn-primary text-sm md:text-base whitespace-nowrap mt-8 md:px-10"
            >{{ content.blurb1.moreButtonText }}</nuxt-link
          >
        </div>
      </div>
    </section>
    <section class="blurb2 w-full py-10 md:py-20 bg-contain bg-no-repeat bg-left-center px-4">
      <div class="max-w-2xl mx-auto md:pl-72 text-center md:text-left">
        <h2 class="text-3xl md:text-5xl font-normal leading-tight mb-4 text-shadow">{{ content.blurb2.heading }}</h2>
        <div class="font-medium leading-snug" v-html="content.blurb2.text" />
        <div v-if="content.blurb2.moreButtonUrl && content.blurb2.moreButtonText">
          <nuxt-link
            :to="content.blurb2.moreButtonUrl"
            class="inline-block btn btn-primary text-sm md:text-base whitespace-nowrap mt-8 md:px-10"
            >{{ content.blurb2.moreButtonText }}</nuxt-link
          >
        </div>
      </div>
    </section>

    <section class="w-full">
      <div class="max-w-2xl mx-auto text-center mt-10 md:mt-20 px-4">
        <div v-html="content.footercontactblurb.text" />
        <div v-if="content.footercontactblurb.moreButtonUrl && content.footercontactblurb.moreButtonText" class="mt-2">
          <nuxt-link
            :to="content.footercontactblurb.moreButtonUrl"
            class="inline-block btn btn-primary text-sm md:text-base whitespace-nowrap mt-8 md:px-10"
            >{{ content.footercontactblurb.moreButtonText }}</nuxt-link
          >
        </div>
      </div>
    </section>
  </main>
</template>

<script>
export default {
  mounted() {},
  async asyncData({ $content, error }) {
    let content
    try {
      content = await $content('about/_index').fetch()
    } catch (e) {
      error({ message: 'Contact data not found' })
    }
    return { content }
  },
}
</script>

<style scoped>
.blurb1 {
  background-image: url('@/assets/images/backgrounds/blurb1.png');
}

.blurb2 {
  background-image: url('@/assets/images/backgrounds/blurb2.png');
}
</style>
```
