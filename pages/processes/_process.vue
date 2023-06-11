<template>
  <main class="pt-6 md:pt-14">
    <div class="video-background" ref="videoBackground">
      <div class="absolute w-full h-2/3 z-40 bg-gradient-to-b from-black via-black via-30%"></div>
      <video class="video" ref="video" autoplay loop muted playsinline>
        <source src="@/assets/video/vbg.mov" type="video/mp4" />
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
          <ul class="list-none grid md:grid-cols-2 gap-1">
            <li
              v-for="(project, index) in post.projects"
              v-if="project.gallery && project.gallery.length > 0"
              :key="project.slug"
              class="cursor-pointer"
              @click="showProject(project.slug)"
            >
              <div
                class="p-4 md:px-10 rounded-lg"
                :class="{
                  'bg-gray-600': selectedProject && selectedProject.slug === project.slug,
                  '': selectedProject && selectedProject.slug !== project.slug,
                }"
              >
                <div class="uppercase text-lg font-heading leading-6 mb-1">{{ project.title }}</div>
                <div class="text-base font-heading leading-5">{{ project.subprocesses.replaceAll('-', ' ') }}</div>
              </div>
            </li>
          </ul>
        </div>

        <div v-if="selectedProject" class="bg-black p-4 md:py-8 md:px-10">
          <h2 class="text-lg uppercase mb-4">{{ selectedProject.title }}</h2>
          <nuxt-content :document="selectedProject" />

          <div v-show="selectedProject.gallery" class="slider mt-8">
            <div class="slide mb-4" v-for="(image, index) in selectedProject.gallery" :key="index">
              <img :src="image" />
            </div>
          </div>
          <!-- More link linkText & linkUrl -->
          <div v-if="selectedProject.moreLinkUrl" class="mt-2">
            <a :href="selectedProject.moreLinkUrl" class="text-white underline" target="_blank">Read more...</a>
          </div>
        </div>

        <div class="flex justify-between">
          <nuxt-link :to="`/`" class="btn btn-primary text-sm md:text-base whitespace-nowrap mt-8"
            >&lt; Return to map</nuxt-link
          >
          <nuxt-link
            :to="`/contact?process=${post.slug}`"
            class="btn btn-primary text-sm md:text-base whitespace-nowrap mt-8"
            >Send Request</nuxt-link
          >
        </div>
      </article>
    </section>
  </main>
</template>

<script>
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
/* purgecss start ignore */
.div-back-button,
.div-next-button {
  position: absolute;
  top: 50%;
  transform: translateY(-50%);
  display: inline-block;
  -webkit-appearance: none;
  -moz-appearance: none;
  appearance: none;
  background: none;
  color: inherit;
  border: none;
  padding: 0;
  cursor: pointer;
  -webkit-user-select: none;
  -moz-user-select: none;
  user-select: none;
  -webkit-tap-highlight-color: rgba(0, 0, 0, 0);
}

.div-back-button {
  left: 2%;
}

.div-next-button {
  right: 2%;
}

.div-back-icon,
.div-next-icon {
  display: inline-block;
  width: 42px;
  height: 42px;
  background-color: rgba(0, 0, 0, 0.5);
  border-radius: 21px;
  display: flex;
  align-items: center;
  justify-content: center;
  transition: opacity 0.2s;
}

[aria-disabled] > .div-back-icon,
[aria-disabled] > .div-next-icon {
  opacity: 0.1;
  cursor: default;
}

:not([aria-disabled]) > .div-back-icon,
:not([aria-disabled]) > .div-next-icon {
  opacity: 0.5;
}

@media (hover: hover) {
  :not([aria-disabled]) > .div-back-icon:hover,
  :not([aria-disabled]) > .div-next-icon:hover {
    opacity: 0.85;
  }
}

:not([aria-disabled]) > .div-back-icon:active,
:not([aria-disabled]) > .div-next-icon:active {
  opacity: 1;
}

:not([aria-disabled]) > .div-back-icon.active,
:not([aria-disabled]) > .div-next-icon.active {
  opacity: 1;
}

.div-back-icon:before,
.div-next-icon:before {
  content: '';
  position: relative;
}

.div-back-icon:before {
  width: 0;
  height: 0;
  background: 0;
  border-style: solid;
  border-width: 9px 12px 9px 0;
  border-color: transparent #fff transparent transparent;
  left: -2px;
}

.div-next-icon:before {
  width: 0;
  height: 0;
  background: 0;
  border-style: solid;
  border-width: 9px 0 9px 12px;
  border-color: transparent transparent transparent #fff;
  left: 2px;
}

.div-dot-button {
  display: inline-block;
  -webkit-appearance: none;
  -moz-appearance: none;
  appearance: none;
  background: none;
  color: inherit;
  border: none;
  padding: 0;
  cursor: pointer;
  -webkit-user-select: none;
  -moz-user-select: none;
  user-select: none;
  -webkit-tap-highlight-color: rgba(0, 0, 0, 0);
}

.div-dots {
  margin-top: 10px;
  display: flex;
  justify-content: center;
}

.div-dot-icon {
  display: inline-block;
  width: 12px;
  height: 12px;
  border-radius: 6px;
  border: 2px solid rgba(0, 0, 0, 0.7);
  margin-left: 4px;
  margin-right: 4px;
  transition: opacity 0.2s;
}

[aria-disabled] > .div-dot-icon {
  opacity: 1;
  background: rgba(0, 0, 0, 0.7);
  cursor: default;
}

:not([aria-disabled]) > .div-dot-icon {
  opacity: 0.5;
}

@media (hover: hover) {
  :not([aria-disabled]) > .div-dot-icon:hover {
    opacity: 0.85;
  }
}

:not([aria-disabled]) > .div-dot-icon:active {
  opacity: 1;
}

:not([aria-disabled]) > .div-dot-icon.active {
  opacity: 1;
}

.div-track {
  display: flex;
  -webkit-user-select: none;
  -moz-user-select: none;
  user-select: none;
}

.div-track.dragging {
  pointer-events: none;
}

.div-slide {
  flex-shrink: 0;
}

.div-mask.disabled .div-slide[aria-hidden='true'] {
  display: none;
}

.div {
  touch-action: pan-y;
}

.div-slides {
  position: relative;
}

.ssr-peek-values {
  position: absolute;
}

.div-mask {
  position: relative;
}

.div-mask:not(.no-mask) {
  overflow: hidden;
}

.div-mask:not(.disabled):not(.not-draggable) {
  cursor: -webkit-grab;
  cursor: grab;
}

.div-mask:not(.disabled):not(.not-draggable).pressing {
  cursor: -webkit-grabbing;
  cursor: grabbing;
}

.div-visually-hidden {
  border: 0;
  clip: rect(0 0 0 0);
  -webkit-clip-path: inset(50%);
  clip-path: inset(50%);
  height: 1px;
  margin: -1px;
  overflow: hidden;
  padding: 0;
  position: absolute;
  width: 1px;
  white-space: nowrap;
}
/* purgecss end ignore */
</style>
