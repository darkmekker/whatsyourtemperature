<template>
  <main class="pt-6 md:pt-14">
    <div class="video-background" ref="videoBackground">
      <div class="absolute w-full h-full bg-gradient-to-b from-black"></div>
      <video class="video" ref="video" autoplay muted loop>
        <source src="@/assets/video/vbg.mp4" type="video/mp4" />
      </video>
    </div>
    <header class="relative z-10 w-full max-w-4xl mx-auto mb-12 pt-20 md:pt-0 md:px-24">
      <a href="/" class="absolute top-0 -left-0">
        <img src="@/assets/icons/contentLogo.svg" alt="Promeos Logo" />
      </a>
      <h1 class="max-w-2xl uppercase mb-6">{{ post.title }}</h1>
      <nuxt-content :document="post" />
    </header>
    <section v-if="post" class="w-full max-w-4xl mx-auto md:px-24">
      <article>
        <div v-if="post.projects" class="mb-6">
          <ul class="list-none grid grid-cols-2 gap-1">
            <li
              v-for="(project, index) in post.projects"
              v-if="project.gallery && project.gallery.length > 0"
              :key="project.slug"
              class="cursor-pointer"
              @click="showProject(project.slug)"
            >
              <div
                class="px-10 py-4 rounded-lg"
                :class="{
                  'bg-gray-600 ': selectedProject && selectedProject.slug === project.slug,
                  '': selectedProject && selectedProject.slug !== project.slug,
                }"
              >
                <div class="uppercase text-lg font-heading leading-6 mb-1">{{ project.title }}</div>
                <div class="text-base font-heading leading-5">{{ project.subprocesses.replaceAll('-', ' ') }}</div>
              </div>
            </li>
          </ul>
        </div>

        <div v-if="selectedProject" class="bg-black md:py-8 md:px-10">
          <h2 class="text-lg uppercase mb-4">{{ selectedProject.title }}</h2>
          <nuxt-content :document="selectedProject" />

          <div v-if="selectedProject.gallery" class="slider mt-8">
            <ssrCarouselCss />
            <ssr-carousel>
              <div class="slide" v-for="(image, index) in selectedProject.gallery" :key="index">
                <img :src="image" />
              </div>
            </ssr-carousel>
          </div>
          <!-- More link linkText & linkUrl -->
          <div v-if="selectedProject.moreLinkUrl" class="mt-2">
            <a :href="selectedProject.moreLinkUrl" class="text-white underline" target="_blank">Read more...</a>
          </div>
        </div>
      </article>
    </section>
  </main>
</template>

<script>
import SsrCarousel from 'vue-ssr-carousel'
import ssrCarouselCss from 'vue-ssr-carousel/index.css'

export default {
  data() {
    return {
      selectedProject: null,
    }
  },
  async asyncData({ $content, params, error }) {
    let post
    try {
      post = await $content('processes', params.process).fetch()
      console.log('post:', post)
    } catch (e) {
      error({ message: 'Project not found' })
    }

    if (post.projects) {
      console.log('post.projects:', post.projects)
      const linkedProjects = await Promise.all(
        post.projects.map(async (slug) => {
          try {
            const project = await $content('projects').where({ slug }).fetch()
            return project[0] ? project[0] : null
          } catch (e) {
            console.error(`Failed to fetch project with slug '${slug}':`, e)
            return null
          }
        })
      )
      post.projects = linkedProjects.filter((project) => project !== null)
    }

    return { post }
  },
  methods: {
    showProject(slug) {
      this.selectedProject = this.post.projects.find((project) => project.slug === slug)
      console.log('this.selectedProject:', this.selectedProject)
    },
  },
  mounted() {
    console.log('this.post.projects:', this.post.projects)
    if (this.post.projects && this.post.projects.length > 0) {
      this.selectedProject = this.post.projects[0]
    }
  },
}
</script>

<style scoped>
.video-background {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  overflow: hidden;
  z-index: -1;
}

.video {
  width: 100%;
  height: 100%;
  object-fit: cover;
}
</style>
