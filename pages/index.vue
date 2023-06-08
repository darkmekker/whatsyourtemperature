<template>
  <div class="zoom-window" ref="zoomWindow">
    <div class="video-background" ref="videoBackground">
      <video class="video" ref="video" autoplay muted loop>
        <source src="@/assets/video/vbg.mp4" type="video/mp4" />
      </video>
    </div>
    <div class="zoom-wrapper" ref="zoomWrapper" @click="handleOverlayClick">
      <div class="process-wrapper">
        <div
          v-for="process in processes"
          :key="process.id"
          :ref="process.id"
          class="process"
          :class="{ hidden: !process.selected && !process.visible, zoomed: process.selected }"
          :style="{
            width: process.selected ? '600px' : `${process.width}px`,
            height: process.selected ? 'auto' : `${process.height}px`,
            marginLeft: `${process.margin}px`,
          }"
          @click="selectProcess(process)"
        >
          <div class="process-inner">
            <h1 class="process-name shadow">{{ process.name }}</h1>
            <div class="process-content">
              <ul class="subprocesses" v-show="process.selected">
                <li v-for="subprocess in process.subprocesses" :key="subprocess" class="shadow">{{ subprocess }}</li>
              </ul>
              <a :href="process.slug" class="process-link">More Info</a>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
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

    this.$refs.zoomWindow.addEventListener('scroll', () => {
      this.previousScrollLeft = this.$refs.zoomWindow.scrollLeft
    })
  },
  methods: {
    async fetchProcesses() {
      try {
        const processes = await this.fetchPosts('processes')
        this.processes = processes.map((process) => ({
          id: process.slug,
          name: process.title,
          content: process.content,
          subprocesses: process.subprocesses || [],
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
      const minProcessHeight = 50

      let currentPosition = 0
      this.processes.forEach((process) => {
        const width = Math.random() * 200 + 200 // Random width between 200 and 400
        const height = Math.random() * 200 + minProcessHeight // Random height between minProcessHeight and (minProcessHeight + 200)
        const margin = Math.random() * minSpacing + minSpacing // Random margin between minSpacing and (2 * minSpacing)

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
        this.scrollSelectedProcessIntoView()
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
          zoomWindow.scrollTo(scrollLeft, 0)
        })
      }
    },
    handleOverlayClick(event) {
      // if clicked zoomWrapper, reset scroll position
      console.log(event.target)
      console.log(this.$refs.zoomWrapper)
      if (event.target === this.$refs.zoomWrapper) {
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

    adjustZoomWrapperWidth() {
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
  height: 100vh;
  overflow-x: auto;
  display: flex;
  align-items: flex-start; /* Align items at the top */
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
  position: absolute;
  height: 50%;
  width: max-content; /* Use max-content to expand horizontally */
  left: 0;
  bottom: 10%;
  z-index: 2;
}
.process-wrapper {
  display: flex;
  flex-wrap: wrap;
  align-items: flex-start; /* Align items at the top */
  max-height: 100%;
  padding: 0 50px;
}
.process {
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  padding: 10px;
  margin-bottom: 20px;
  cursor: pointer;
}

.process-inner {
  display: flex;
  align-items: start;
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

.process-content {
  display: none;
}

.process.hidden {
  opacity: 0;
}

.process.zoomed {
  z-index: 2;
}
.process.zoomed .process-content {
  display: block;
}

.subprocesses {
  list-style-type: none;
  margin: 0;
  padding: 0;
}

.process-link {
  margin-top: 10px;
  text-align: center;
  color: blue;
  text-decoration: underline;
}

.process.hidden {
  display: none;
}
</style>
