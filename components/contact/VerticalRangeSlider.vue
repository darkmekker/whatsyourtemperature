<template>
  <div class="vertical-range-slider" :style="sliderContainerStyle">
    <div class="slider-track" :style="sliderTrackStyle"></div>
    <div
      class="slider-handle min-handle"
      :style="getHandleStyle(handles[0])"
      @mousedown="startDrag(0, $event)"
      @touchstart="startDrag(0, $event)"
    >
      <div class="handle-label">{{ handles[0].value }} C</div>
    </div>
    <div
      class="slider-handle max-handle"
      :style="getHandleStyle(handles[1])"
      @mousedown="startDrag(1, $event)"
      @touchstart="startDrag(1, $event)"
    >
      <div class="handle-label">{{ handles[1].value }} C</div>
    </div>
    <input type="hidden" name="selected-temp-min" :value="handles[0].value" />
    <input type="hidden" name="selected-temp-max" :value="handles[1].value" />
  </div>
</template>

<script>
export default {
  name: 'VerticalRangeSlider',
  props: {
    min: {
      type: Number,
      default: 0,
    },
    max: {
      type: Number,
      default: 100,
    },
    defaultValues: {
      type: Array,
      default: () => [],
    },
    containerHeight: {
      type: Number,
      default: 200,
    },
  },
  data() {
    return {
      handles: [
        { value: 0, position: 0 },
        { value: 0, position: 0 },
      ],
      draggingHandle: -1,
      startY: 0,
      startValue: 0,
    }
  },
  computed: {
    sliderContainerStyle() {
      return `height: ${this.containerHeight}px`
    },
    sliderTrackStyle() {
      return `height: ${parseInt(this.containerHeight)}px` // Adjust handle-label height (20px)
    },
  },
  methods: {
    inputName(index) {
      return `input${index}`
    },
    getHandleStyle(handle) {
      const containerHeight = parseInt(this.containerHeight)
      const position = containerHeight - ((handle.value - this.min) / (this.max - this.min)) * containerHeight
      return `top: ${position}px`
    },
    updateHandles() {
      this.handles = this.defaultValues.map((value, index) => {
        const position = ((value - this.min) / (this.max - this.min)) * (this.max - this.min)
        // floor the value
        value = Math.floor(value)
        return { value, position }
      })
    },
    startDrag(index, event) {
      event.preventDefault()
      this.draggingHandle = index
      this.startY = event.clientY || event.touches[0].clientY
      this.startValue = this.handles[index].value
      document.addEventListener('mousemove', this.handleDrag)
      document.addEventListener('touchmove', this.handleDrag)
      document.addEventListener('mouseup', this.stopDrag)
      document.addEventListener('touchend', this.stopDrag)
    },
    handleDrag(event) {
      event.preventDefault()
      const clientY = event.clientY || event.touches[0].clientY
      const delta = this.startY - clientY
      const containerHeight = parseInt(this.containerHeight)
      const range = this.max - this.min
      const pixelRange = containerHeight - 20 // Adjust handle-label height (20px)
      const step = (range / pixelRange) * delta
      const newValue = Math.floor(this.startValue + step)
      if (this.draggingHandle === 0) {
        // Dragging the min handle
        if (newValue <= this.min) {
          this.handles[0].value = this.min
        } else if (newValue >= this.handles[1].value) {
          this.handles[0].value = this.handles[1].value
        } else {
          this.handles[0].value = newValue
        }
      } else if (this.draggingHandle === 1) {
        // Dragging the max handle
        if (newValue >= this.max) {
          this.handles[1].value = this.max
        } else if (newValue <= this.handles[0].value) {
          this.handles[1].value = this.handles[0].value
        } else {
          this.handles[1].value = newValue
        }
      }
    },
    stopDrag() {
      this.$emit('sliderValueChange', this.handles)

      document.removeEventListener('mousemove', this.handleDrag)
      document.removeEventListener('touchmove', this.handleDrag)
      document.removeEventListener('mouseup', this.stopDrag)
      document.removeEventListener('touchend', this.stopDrag)
    },
  },
  mounted() {
    this.updateHandles()
    this.$emit('sliderValueChange', this.handles)
  },
}
</script>

<style>
.vertical-range-slider {
  display: inline-block;
  position: relative;
  overflow: hidden;
  min-height: 400px;
  width: 100%;
}

.slider-track {
  position: absolute;
  top: 0; /* Adjust the top position as needed */
  left: 8px;
  transform: translateX(-50%);
  width: 5px;
  height: 100%;
  background: url('@/assets/images/vertical-slider-track.png') no-repeat center center;
  background-size: cover;
  border-radius: 5px;
}

.slider-handle {
  position: absolute;
  left: 8px;
  transform: translateX(-50%);
  width: 17px;
  height: 17px;
  border-radius: 50%;
  background-color: #fff;
  cursor: grab;
  word-wrap: none;
  white-space: nowrap;
}
.min-handle {
  transform: translateX(-50%) translateY(-100%);
}
.min-handle::before {
  content: '';
  position: absolute;
  top: 17px;
  left: 6px;
  width: 6px;
  height: 400px;
  background-color: rgba(0, 0, 0, 0.7);
}
.max-handle::before {
  content: '';
  position: absolute;
  bottom: 17px;
  left: 6px;
  width: 6px;
  height: 400px;
  background-color: rgba(0, 0, 0, 0.7);
}

.handle-label {
  position: absolute;
  top: -5px;
  left: 100%;
  padding-left: 30px;
  font-size: 20px;
  color: #fff;
}
</style>
