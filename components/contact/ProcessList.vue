<template>
  <div v-if="processes" class="process-list fade-list" :style="{ height: containerHeight + 'px' }" ref="processList">
    <ul class="my-32">
      <li
        class="font-heading"
        :class="{ selected: selectedProcess && selectedProcess.slug === process.slug }"
        v-for="process in processes"
        :key="process.slug"
        ref="process"
        @click="selectProcess(process)"
      >
        {{ process.title }}
      </li>
    </ul>
    <input type="hidden" v-model="selectedProcess.title" name="selectedProcess" />
  </div>
</template>

<script>
export default {
  props: {
    containerHeight: {
      type: Number,
      default: 200,
    },
  },
  data() {
    return {
      processes: [],
      selectedProcess: {
        title: '',
        slug: '',
      },
    }
  },
  mounted() {
    this.fetchProcesses()

    console.log('this.$route.query.process', this.$route.query.process)
    console.log('this.selectedProcess', this.selectedProcess)
  },
  methods: {
    async fetchProcesses() {
      try {
        // Call the fetchPosts method to fetch the processes
        const processes = await this.fetchPosts('processes')

        // Update the processes array with the fetched data, sort by title
        this.processes = processes.sort((a, b) => {
          if (a.title < b.title) return -1
          if (a.title > b.title) return 1
          return 0
        })

        this.$nextTick(() => {
          if (this.$route.query.process)
            this.selectProcess({
              title: this.$route.query.process,
              slug: this.$route.query.process,
            })
        })
      } catch (error) {
        console.error(error)
        this.processes = [] // Set an empty array in case of an error
      }
    },
    selectProcess(process) {
      console.log('selectProcess', process)
      if (!process.title) return
      this.selectedProcess = process
      this.$emit('processListChange', process.title)
      this.scrollListToView(process)
    },
    scrollListToView(process) {
      if (!process.title) return
      // scroll processList y position to the selected process. get the process ref by index
      this.$nextTick(() => {
        const processIndex = this.processes.findIndex((p) => p.slug === process.slug)
        if (!this.$refs.process[processIndex]) return
        this.$refs.process[processIndex].scrollIntoView({
          behavior: 'smooth',
          block: 'center',
        })
      })
    },
    async fetchPosts(postType) {
      return this.$content(postType)
        .fetch()
        .catch((err) => console.error(err) || [])
    },
  },
}
</script>

<style scoped>
.process-list {
  overflow-y: scroll;
  -webkit-overflow-scrolling: touch; /* Add this line */
}
ul {
  list-style: none;
  padding: 0;
}
.fade-list {
  mask-image: linear-gradient(to bottom, transparent 0%, black 150px, black calc(100% - 150px), transparent 100%);
}
li {
  cursor: pointer;
  padding: 10px;
  border-radius: 10px;
  text-align: right;
  margin-bottom: 10px;
  text-transform: uppercase;
}
li:hover,
li.selected {
  background-color: var(--color-primary);
}
</style>
