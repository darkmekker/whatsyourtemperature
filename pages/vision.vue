<template>
  <main class="pt-6 md:pt-14 px-0 bg-contain bg-no-repeat bg-center vision-bg">
    <header class="w-full mx-auto mb-4 pt-20 md:pt-0 md:px-24 relative">
      <a href="/" class="absolute top-0 left-10">
        <img src="@/assets/icons/contentLogo.svg" alt="Promeos Logo" />
      </a>
      <h1 class="max-w-2xl text-center mx-auto">{{ content.title }}</h1>
    </header>
    <section class="w-full px-4">
      <div class="max-w-2xl mx-auto text-base text-center">
        <div class="my-2 md:my-8" v-html="$md.render(content.intro)" />
      </div>
    </section>

    <section v-if="content.timeline" class="w-full py-10 bg-contain bg-no-repeat bg-left-center px-4">
      <div class="text-center mb-6" v-if="content.heroimage">
        <img :src="content.heroimage" :alt="content.title" class="block m-auto w-[139px]" />
      </div>

      <div class="max-w-xs mx-auto text-center md:text-left">
        <div v-if="content.timeline">
          <div
            v-for="(item, index) in content.timeline"
            :key="index"
            class="mb-3 timeline-item-bg bg-no-repeat bg-top pt-7"
          >
            <h2 class="text-5xl leading-snug text-center mb-0 text-shadow" v-html="item.year" />
            <p class="text-center leading-tight mt-1 text-shadow">{{ item.text }}</p>
          </div>
        </div>
      </div>
    </section>

    <section v-if="content.footercontactblurb" class="w-full">
      <div class="max-w-lg mx-auto text-center mt-10 px-4">
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
  async asyncData({ $content, error }) {
    let content
    try {
      content = await $content('vision/_index').fetch()
    } catch (e) {
      error({ message: 'Contact data not found' })
    }
    return { content }
  },
}
</script>
<style scoped>
.vision-bg {
  background-image: url('@/assets/images/backgrounds/vision_bg.png');
}
.timeline-item-bg {
  background-image: url('@/assets/images/backgrounds/vision_timeline_item_bg.png');
}
</style>
