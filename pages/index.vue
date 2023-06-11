<template>
  <div>
    <div class="zoom-window overflow-auto md:overflow-hidden" ref="zoomWindow" @click="handleOverlayClick">
      <main class="relative z-40 bg-gradient-to-b from-black via-black via-30% pt-6 md:pt-16 pb-0" ref="mainHeader">
        <header class="w-full mx-auto mb-4 pt-20 md:pt-0 md:px-24 relative">
          <img src="@/assets/icons/contentLogo.svg" alt="Promeos Logo" class="absolute top-0 -left-0" />
          <h1 class="max-w-2xl">{{ content.title }}</h1>
          <nuxt-content :document="content" />
        </header>
      </main>

      <div class="video-background" ref="videoBackground">
        <video class="video" ref="video" autoplay muted loop>
          <source src="@/assets/video/vbg.mp4" type="video/mp4" />
        </video>
      </div>
      <div class="zoom-wrapper" ref="zoomWrapper">
        <div
          class="process-wrapper px-10 pb-24 flex justify-center items-center place-items-center"
          ref="processWrapper"
        >
          <div
            v-for="process in processes"
            :key="process.id"
            :ref="process.id"
            class="process mr-20 w-full md:w-auto relative"
            :class="{
              hidden: !process.selected && !process.visible,
              zoomed: process.selected,
            }"
            :style="{
              //width: process.selected ? '600px' : `${process.width}px`,
              marginTop: process.selected ? '' : `${process.height}px`,
              marginLeft: process.selected ? '' : `${process.margin}px`,
            }"
            @click="selectProcess(process)"
          >
            <div class="process-inner">
              <h1 class="process-name text-shadow">{{ process.name }}</h1>
              <div class="process-content" v-show="process.selected">
                <ul class="block font-heading text-lg md:text-xlg">
                  <li v-for="subprocess in process.subprocesses" :key="subprocess" class="text-shadow">
                    {{ subprocess }}
                  </li>
                </ul>

                <div class="block process-link mt-8" v-if="process.projects.length > 0">
                  <nuxt-link
                    :to="`/processes/${process.id}`"
                    class="btn btn-primary text-sm md:text-base whitespace-nowrap"
                    >See our references</nuxt-link
                  >
                </div>
                <div class="block mt-4" @click="showAllProcesses()">
                  <div class="btn btn-primary text-sm md:text-base whitespace-nowrap">&lt; back</div>
                </div>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  layout: 'home',
  async asyncData({ $content, error }) {
    let content
    try {
      content = await $content('home/_index').fetch()
    } catch (e) {
      error({ message: 'Home data not found' })
    }
    return { content }
  },
  data() {
    return {
      processes: [],
      selectedProcess: null,
    }
  },
  mounted() {
    this.fetchProcesses().then(() => {
      this.calculateProcessPositions()
    })

    this.previousScrollLeft = 0 // Add this line to initialize previousScrollLeft

    this.adjustZoomWrapperHeight()

    // fire adjustZoomWrapperHeight on resize
    window.addEventListener('resize', () => {
      this.adjustZoomWrapperHeight()
    })

    // this.$refs.zoomWindow.addEventListener('scroll', () => {
    //   this.previousScrollLeft = this.$refs.zoomWindow.scrollLeft
    // })
  },
  methods: {
    async fetchProcesses() {
      try {
        const processes = await this.fetchPosts('processes')
        console.log(processes)
        this.processes = processes.map((process) => ({
          id: process.slug,
          name: process.title,
          content: process.content,
          subprocesses: process.subprocesses || [],
          projects: process.projects || [],
          link: process.link || '',
          selected: false,
          visible: true,
          width: 0,
          height: 0,
          margin: 0,
        }))
      } catch (error) {
        console.error(error)
        this.processes = []
      }
    },
    async fetchPosts(postType) {
      return this.$content(postType)
        .fetch()
        .catch((err) => console.error(err) || [])
    },
    calculateProcessPositions() {
      const minSpacing = 100
      const minProcessHeight = 10

      let currentPosition = 0
      this.processes.forEach((process) => {
        const width = Math.random() * 100 + 100 // Random width between 200 and 400
        const height = Math.random() * 100 + minProcessHeight // Random height between minProcessHeight and (minProcessHeight + 200)
        const margin = Math.random() * minSpacing // Random margin between minSpacing and (2 * minSpacing)

        process.width = width
        process.height = height
        process.margin = margin

        currentPosition += width + margin
      })

      // fire with nexttick
      this.$nextTick(() => {
        this.adjustZoomWrapperWidth()
      })
    },
    selectProcess(process) {
      if (this.selectedProcess === process) {
        this.selectedProcess = null
      } else {
        this.selectedProcess = process
        this.processes.forEach((p) => {
          p.selected = p === process
          p.visible = p === process
        })
        //this.scrollSelectedProcessIntoView()
        // scale the videoBackground to 2
      }
    },
    scrollSelectedProcessIntoView() {
      if (this.selectedProcess) {
        this.$nextTick(() => {
          const zoomWindow = this.$refs.zoomWindow
          const zoomWindowRect = zoomWindow.getBoundingClientRect()

          const selectedProcessElement = this.$refs[this.selectedProcess.id][0]
          if (!selectedProcessElement) return

          const selectedProcessRect = selectedProcessElement.getBoundingClientRect()

          let scrollLeft =
            selectedProcessRect.left - zoomWindowRect.left - (zoomWindowRect.width - selectedProcessRect.width) / 2
          scrollLeft = Math.max(scrollLeft, 0)
          // scroll zoomWindow to the left without animation
          //zoomWindow.scrollTo(scrollLeft, 0)
        })
      }
    },
    handleOverlayClick(event) {
      //is clicking outside a process
      if (
        event.target === this.$refs.zoomWrapper ||
        event.target === this.$refs.processWrapper ||
        event.target === this.$refs.zoomWindow ||
        event.target === this.$refs.mainHeader
      ) {
        this.resetScrollPosition()
      }
    },

    resetScrollPosition() {
      this.isZoomedIn = false
      this.zoomedProcessId = null
      this.selectedProcess = null

      this.$refs.zoomWrapper.style.transform = 'none'
      this.showAllProcesses()
      this.scrollZoomWindowToPreviousPosition()
      console.log('resetScrollPosition')
    },

    showAllProcesses() {
      this.processes.forEach((process) => {
        process.visible = true
        process.selected = false
      })
    },

    scrollZoomWindowToPreviousPosition() {
      this.$refs.zoomWindow.scrollTo({
        left: this.previousScrollLeft,
        behavior: 'smooth',
      })
    },

    adjustZoomWrapperHeight() {
      // on mount an on resize set the zoomWrapper to the height of zoomWindow minus the height of the header
      const zoomWindow = this.$refs.zoomWindow
      const zoomWrapper = this.$refs.zoomWrapper
      const mainHeader = this.$refs.mainHeader

      zoomWrapper.style.height =
        zoomWindow.clientHeight > 800 ? `${zoomWindow.clientHeight - mainHeader.clientHeight}px` : ''
    },

    adjustZoomWrapperWidth() {
      return
      const zoomWrapper = this.$refs.zoomWrapper
      const processes = this.processes

      let totalWidth = 0
      processes.forEach((process) => {
        totalWidth += process.width + process.margin
      })
      // if the number of processes is not even, add the width of the last process as extra
      if (processes.length % 2 === 1) {
        totalWidth += processes[processes.length - 1].width
      }

      zoomWrapper.style.width = `${totalWidth / 2}px`
    },
  },
}
</script>

<style scoped>
.zoom-window {
  width: 100%;
  display: block;
  position: relative;
}

.video-background {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  overflow: hidden;
  z-index: 1;
}

.video {
  width: 100%;
  height: 100%;
  object-fit: cover;
}

.zoom-wrapper {
  width: 100%; /* Use max-content to expand horizontally */
  /* height: 70%; */
  position: relative;
  z-index: 2;
  display: flex;
  align-items: flex-start; /* Align items at the top */
  overflow: auto;
}
.process-wrapper {
  display: flex;
  flex-wrap: wrap;
  align-items: flex-start; /* Align items at the top */
  width: 200%;
}
.process {
  display: block;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  padding: 10px;
  margin-bottom: 20px;
  cursor: pointer;
  transition: opacity 0.3s ease-in-out;
  min-width: 200px;
  min-height: 200px;
}

.process-inner {
}

.process-name {
  position: relative;
  font-size: 30px;
  text-align: left;
  flex-grow: 1;
  text-transform: uppercase;
}
.process-name:after {
  content: '';
  position: absolute;
  top: 4px;
  left: -10px;
  width: 5px;
  height: 100px;
  border-left: 5px solid #fff;
}

.process.hidden {
  opacity: 0;
}

.process.zoomed {
  z-index: 2;
  min-height: auto;
  margin-left: 0;
  margin-top: 4rem;
}

/* .process.zoomed .process-content {
  display: block;
}
.process.zoomed .process-name {
  font-size: 50px;
} */
.process.zoomed .process-name:after {
  border-left-width: 8px;
  left: -22px;
  height: 400px;
}

.process-link {
  display: block;
  color: blue;
  text-decoration: underline;
}

.process.hidden {
  display: none;
}
</style>
