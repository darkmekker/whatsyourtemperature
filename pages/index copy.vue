<template>
  <div class="zoom-window" ref="zoomWindow">
    <div class="zoom-wrapper" ref="zoomWrapper">
      <div
        class="video-background"
        ref="videoBackground"
        @click="handleOverlayClick"
        @wheel.prevent="handleZoom"
        @gesturechange.prevent="handleZoom"
        @gesturestart.prevent=""
        @mousedown="handleMouseDown"
        @mousemove="handleMouseMove"
        @mouseup="handleMouseUp"
        @touchstart="handleTouchStart"
        @touchmove="handleTouchMove"
        @touchend="handleTouchEnd"
      >
        <video class="video" ref="video" autoplay muted loop>
          <source src="@/assets/video/vbg.mp4" type="video/mp4" />
        </video>
        <div class="overlay">
          <div v-if="isLoading" class="loader-overlay">
            <div class="loader"></div>
          </div>
          <div v-else>
            <div
              v-for="process in processes"
              :key="process.id"
              :ref="process.id"
              :class="[
                'process',
                { hidden: isZoomedIn && process.id !== zoomedProcessId },
                { zoomed: isZoomedIn && process.id === zoomedProcessId },
              ]"
              :style="getProcessStyle(process)"
              @click="zoomProcess(process)"
            >
              <h1 class="process-name">{{ process.name }}</h1>

              <div class="process-content">
                <!-- List subprocesses -->
                <ul class="subprocesses" v-if="process.subprocesses">
                  <li v-for="(subprocess, index) in process.subprocesses" :key="subprocess">
                    <div v-if="process.subprocesses.includes(subprocess)">{{ subprocess }}</div>
                  </li>
                </ul>

                <!-- Button linking to process page -->
                <div class="process-link">
                  <nuxt-link :to="`/processes/${process.id}`" class="btn btn-primary">Read more...</nuxt-link>
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
  data() {
    return {
      processes: [],
      isLoading: true,
      isZoomedIn: false,
      zoomedProcessId: null,
      zoomLevel: 1,
      originalVideoWidth: 0,
      originalVideoHeight: 0,
      isDragging: false,
      dragStartX: 0,
      dragStartY: 0,
      dragOffsetX: 0,
      dragOffsetY: 0,
      touchIdentifier: null,
    }
  },
  methods: {
    async fetchProcesses() {
      try {
        const processes = await this.fetchPosts('processes')
        this.processes = processes.map((process) => ({
          name: process.title,
          id: process.slug,
          subprocesses: process.subprocesses || [],
          showSubprocesses: false,
          position: { x: 0, y: 0 },
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
      const overlayWidth = this.$refs.videoBackground.offsetWidth
      const overlayHeight = this.$refs.videoBackground.offsetHeight
      const padding = 50
      const minDistance = 100

      const positions = []
      this.processes.forEach((process) => {
        let newPosition
        do {
          newPosition = {
            x: Math.random() * (overlayWidth - padding * 2) + padding,
            y: Math.random() * (overlayHeight - padding * 2) + padding,
          }
        } while (this.isOverlapWithOtherProcesses(newPosition, positions, minDistance))

        positions.push(newPosition)
        process.position = newPosition
      })
    },
    isOverlapWithOtherProcesses(newPosition, positions, minDistance) {
      for (const position of positions) {
        if (this.isOverlap(newPosition, position, minDistance)) {
          return true
        }
      }
      return false
    },
    isOverlap(position1, position2, minDistance) {
      const distanceX = position1.x - position2.x
      const distanceY = position1.y - position2.y
      const distance = Math.sqrt(distanceX * distanceX + distanceY * distanceY)
      return distance < minDistance
    },
    getProcessStyle(process) {
      const style = {
        left: `${process.position.x}px`,
        top: `${process.position.y}px`,
      }

      return style
    },
    zoomProcess(process) {
      if (this.isZoomedIn) {
        // Zoom out
        this.zoomLevel = 1
        this.isZoomedIn = false
        this.zoomedProcessId = null
        this.$refs.zoomWrapper.style.transform = `scale(${this.zoomLevel})`
      } else {
        // Zoom in
        this.zoomLevel = 3
        this.isZoomedIn = true
        this.zoomedProcessId = process.id
        this.$refs.zoomWrapper.style.transform = `scale(${this.zoomLevel})`
        this.$nextTick(() => {
          this.centerZoomedProcess(process)
        })
      }
    },
    centerZoomedProcess(process) {
      const zoomWindow = this.$refs.zoomWindow
      const zoomWrapper = this.$refs.zoomWrapper
      const processElement = this.$refs[process.id][0]

      const zoomWindowRect = zoomWindow.getBoundingClientRect()
      const processRect = processElement.getBoundingClientRect()

      const zoomedProcessCenterX = processRect.left + processRect.width / 2
      const zoomedProcessCenterY = processRect.top + processRect.height / 2

      const offsetX = zoomWindowRect.width / 2 - zoomedProcessCenterX
      const offsetY = zoomWindowRect.height / 2 - zoomedProcessCenterY

      console.log('zoomWrapperRect', zoomWindowRect)
      console.log('processRect', processRect)
      console.log('offsetX', offsetX)
      console.log('offsetY', offsetY)
      console.log('zoomLevel', this.zoomLevel)

      //scale(3) translate(124.5px, -94.906px)
      zoomWrapper.style.transform = `scale(${this.zoomLevel}) translate(${offsetX}px, ${offsetY}px)`
    },
    handleOverlayClick() {
      console.log('handleOverlayClick')
      // if (this.isZoomedIn) {
      //   this.zoomLevel = 1
      //   this.isZoomedIn = false
      //   this.zoomedProcessId = null
      //   //this.$refs.zoomWrapper.style.transform = `scale(${this.zoomLevel})`
      // }
    },
    handleZoom(event) {
      this.isZoomedIn = false
      this.zoomedProcessId = null

      if (event.deltaY > 0) {
        this.zoomLevel -= 0.1
      } else {
        this.zoomLevel += 0.1
      }
      if (this.zoomLevel < 1) {
        this.zoomLevel = 1
      }
      if (this.zoomLevel > 2) {
        this.zoomLevel = 2
      }
      this.$refs.zoomWrapper.style.transform = `scale(${this.zoomLevel})`
    },
    handleMouseDown(event) {
      if (event.button === 0) {
        this.isDragging = true
        this.dragStartX = event.clientX
        this.dragStartY = event.clientY
        this.dragOffsetX = 0
        this.dragOffsetY = 0
        document.addEventListener('mousemove', this.handleMouseMove)
        document.addEventListener('mouseup', this.handleMouseUp)
      }
    },
    handleMouseMove(event) {
      if (this.isDragging) {
        const deltaX = event.clientX - this.dragStartX
        const deltaY = event.clientY - this.dragStartY
        this.dragOffsetX += deltaX
        this.dragOffsetY += deltaY
        this.$refs.zoomWrapper.style.transform = `scale(${this.zoomLevel}) translate(${this.dragOffsetX}px, ${this.dragOffsetY}px)`
        this.dragStartX = event.clientX
        this.dragStartY = event.clientY
      }
    },
    handleMouseUp(event) {
      if (event.button === 0 && this.isDragging) {
        this.isDragging = false
        document.removeEventListener('mousemove', this.handleMouseMove)
        document.removeEventListener('mouseup', this.handleMouseUp)
      }
    },

    handleTouchStart(event) {
      if (event.touches.length === 1 && this.zoomLevel > 1) {
        this.isDragging = true
        this.dragStartX = event.touches[0].clientX
        this.dragStartY = event.touches[0].clientY
        this.dragOffsetX = 0
        this.dragOffsetY = 0
        this.touchIdentifier = event.touches[0].identifier
      }
    },
    handleTouchMove(event) {
      if (this.isDragging && this.zoomLevel > 1) {
        const touch = Array.from(event.touches).find((touch) => touch.identifier === this.touchIdentifier)
        if (touch) {
          const deltaX = touch.clientX - this.dragStartX
          const deltaY = touch.clientY - this.dragStartY
          this.dragOffsetX += deltaX
          this.dragOffsetY += deltaY
          this.$refs.zoomWrapper.style.transform = `scale(${this.zoomLevel}) translate(${this.dragOffsetX}px, ${this.dragOffsetY}px)`
          this.dragStartX = touch.clientX
          this.dragStartY = touch.clientY
        }
      }
    },
    handleTouchEnd(event) {
      const touchList = Array.from(event.touches) // Convert TouchList to an array
      const touch = touchList.find((t) => t.identifier === this.touchIdentifier)

      if (this.isDragging && touch) {
        this.isDragging = false
        this.touchIdentifier = null
      }
    },
  },
  mounted() {
    this.fetchProcesses()
      .then(() => {
        this.isLoading = false
        this.$nextTick(() => {
          this.calculateProcessPositions()
        })
      })
      .catch((error) => {
        console.error(error)
        this.isLoading = false
      })

    const video = this.$refs.video
    this.originalVideoWidth = video.videoWidth
    this.originalVideoHeight = video.videoHeight
  },
}
</script>

<style>
.zoom-window {
  width: 100%;
  height: 100vh;
  overflow: hidden;
  position: relative;
}
.zoom-wrapper {
  width: 100%;
  height: 100vh;
  overflow: hidden;
  transition: transform 0.3s ease;
  user-select: none;
}

.video-background {
  width: 100%;
  height: 100%;
}

.video {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  object-fit: cover;
}

.overlay {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  display: flex;
  justify-content: center;
  align-items: center;
}

.loader-overlay {
  display: flex;
  justify-content: center;
  align-items: center;
}

.loader {
  border: 8px solid #f3f3f3;
  border-top: 8px solid #3498db;
  border-radius: 50%;
  width: 60px;
  height: 60px;
  animation: spin 2s linear infinite;
}

@keyframes spin {
  0% {
    transform: rotate(0deg);
  }
  100% {
    transform: rotate(360deg);
  }
}

.process {
  position: absolute;
  cursor: pointer;
  pointer-events: auto;
  transition: transform 0.3s ease;
  z-index: 1;
  border-left: 5px solid #fff;
  padding: 0 0 50px 7px;
}

.process-name {
  font-size: 20px;
  text-align: left;
  flex-grow: 1;
  text-transform: uppercase;
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
</style>
