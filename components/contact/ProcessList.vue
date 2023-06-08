<template>
  <div class="process-list fade-list" :style="{ height: containerHeight + 'px' }" ref="processList">
    <ul class="">
      <li
        :class="{ selected: selectedProcess === process }"
        v-for="process in processes"
        :key="process.slug"
        @click="selectProcess(process)"
      >
        {{ process.title }}
      </li>
    </ul>
    <input type="hidden" v-model="selectedProcess.name" name="selectedProcess" />
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
      selectedProcess: '',
    }
  },
  mounted() {
    this.fetchProcesses()
  },
  methods: {
    async fetchProcesses() {
      try {
        // Call the fetchPosts method to fetch the processes
        const processes = await this.fetchPosts('processes')
        console.log('processes:', processes)

        // Update the processes array with the fetched data
        this.processes = processes
      } catch (error) {
        console.error(error)
        this.processes = [] // Set an empty array in case of an error
      }
    },
    selectProcess(process) {
      this.selectedProcess = process
      this.$emit('processListChange', process.title)
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
  margin: 100px 0;
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
