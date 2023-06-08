<template>
  <ul v-if="posts.length > 0">
    <li v-for="(post, index) in posts" :key="index">
      <nuxt-link :to="`/processes/${post.slug}`" class="card card--clickable">
        <span class="flex-1">
          <h3 class="card-title">{{ post.title }}</h3>
          <div v-if="post.subprocesses.length > 0" class="nuxt-content">
            <ul>
              <li v-for="(subprocess, index) in post.subprocesses" :key="subprocess.slug">
                <div>{{ subprocess }}</div>
              </li>
            </ul>
          </div>
        </span>
      </nuxt-link>
    </li>
  </ul>
  <div v-else-if="loading" class="cards">
    <div v-for="placeholder in placeholderClasses" :key="placeholder.id" class="card">
      <content-placeholders :rounded="true" :class="placeholder">
        <content-placeholders-heading />
      </content-placeholders>
    </div>
  </div>
  <p v-else class="max-w-5xl mx-auto">
    {{ amount > 1 ? 'Posts not found' : 'Post not found' }}
  </p>
</template>

<script>
export default {
  name: 'Posts',
  props: {
    postType: {
      type: String,
      default: 'processes',
      validator: (val) => ['processes', 'projects'].includes(val),
    },
    amount: {
      // ? https://content.nuxtjs.org/fetching#limitn
      type: Number,
      default: 10,
      validator: (val) => val >= 0 && val < 100,
    },
    sortBy: {
      // ? https://content.nuxtjs.org/fetching#sortbykey-direction
      type: Object,
      default: () => ({
        key: 'slug',
        direction: 'desc', // you probably want 'asc' here
      }),
      validator: (obj) => typeof obj.key === 'string' && typeof obj.direction === 'string',
    },
  },
  data() {
    return {
      posts: [],
      loading: true,
    }
  },
  computed: {
    placeholderClasses() {
      const classes = ['w-full', 'w-2/3', 'w-5/6']
      return [...Array.from({ length: this.amount }, (v, i) => classes[i % classes.length])] // repeats classes after one another
    },
  },
  async mounted() {
    this.loading = true
    this.posts = await this.fetchPosts()
    this.loading = false
  },
  methods: {
    formatDate(dateString) {
      const date = new Date(dateString)
      return date.toLocaleDateString(process.env.lang) || ''
    },
    async fetchPosts(postType = this.postType, amount = this.amount, sortBy = this.sortBy) {
      return postType
        ? this.$content(postType.toString())
            .sortBy(sortBy.key, sortBy.direction)
            .limit(amount)
            .fetch()
            .catch((err) => console.error(err) || [])
        : []
    },
  },
}
</script>
