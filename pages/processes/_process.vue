<template>
  <main>
    <section v-if="post">
      <nav class="mb-8" aria-label="go back">
        <router-back class="block" />
      </nav>

      <article>
        <img v-if="post.cover" class="cover-image" :src="post.cover" />
        <h1>{{ post.title }}</h1>
        <nuxt-content :document="post" />
        <!-- List post.subprocesses -->

        <div v-if="post.projects" class="nuxt-content">
          <ul>
            <li v-for="(project, index) in post.projects" :key="project.slug" @click="showProject(project.slug)">
              <div>{{ project.title }}</div>

              <!-- Get the first subprocess that matches the post.subprocesses -->
              <div v-if="project.subprocesses">
                <div v-for="(subprocess, index) in project.subprocesses" :key="subprocess">
                  <div v-if="post.subprocesses.includes(subprocess)">{{ subprocess }}</div>
                </div>
              </div>
            </li>
          </ul>
        </div>

        <div v-if="selectedProject">
          <img v-if="selectedProject.cover" class="cover-image" :src="selectedProject.cover" />
          <!-- <h6 class="inline py-1 px-2 mr-1 bg-gray text-white text-sm font-medium rounded-sm">{{ post.category }}</h6> -->
          <h1 class="">{{ selectedProject.title }}</h1>
          <p class="mt-1 mb-8 text-primary-600 dark:text-primary-400">{{ selectedProject.description }}</p>
          <nuxt-content :document="selectedProject" />

          <div v-if="selectedProject.gallery" class="slider">
            <ssr-carousel>
              <div class="slide" v-for="(image, index) in selectedProject.gallery" :key="index">
                <img :src="image" />
              </div>
            </ssr-carousel>
          </div>
          <!-- More link linkText & linkUrl -->
          <div v-if="selectedProject.moreLinkUrl" class="mt-8">
            <a :href="selectedProject.moreLinkUrl" class="btn btn-primary">Read more...</a>
          </div>
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
