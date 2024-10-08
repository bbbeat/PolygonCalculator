<template>
  <div>
    <canvas ref="canvas" :width="width" :height="height" @click="addPoint"></canvas>
    <div>
      <button @click="completePolygon" :disabled="points.length < 3 || isComplete">
        Complete Polygon
      </button>
      <button @click="reset">Reset</button>
      <p v-if="area !== null">Area: {{ area.toFixed(2) }} square pixels</p>
    </div>
  </div>
</template>

<script>
export default {
  name: 'PolygonAreaCalculator',
  data() {
    return {
      width: 500,
      height: 300,
      points: [],
      isComplete: false,
      area: null
    }
  },
  methods: {
    addPoint(event) {
      if (this.isComplete) return
      const rect = event.target.getBoundingClientRect()
      const x = event.clientX - rect.left
      const y = event.clientY - rect.top
      this.points.push({ x, y })
      this.redraw()
    },
    completePolygon() {
      if (this.points.length < 3 || this.isComplete) return
      this.isComplete = true
      this.redraw()
      this.calculateArea()
    },
    redraw() {
      const ctx = this.$refs.canvas.getContext('2d')
      ctx.clearRect(0, 0, this.width, this.height)

      // Draw lines
      ctx.beginPath()
      this.points.forEach((point, index) => {
        if (index === 0) {
          ctx.moveTo(point.x, point.y)
        } else {
          ctx.lineTo(point.x, point.y)
        }
      })
      if (this.isComplete) {
        ctx.closePath()
        ctx.fillStyle = 'rgba(0, 255, 0, 0.2)'
        ctx.fill()
      }
      ctx.stroke()

      // Draw points
      this.points.forEach((point, index) => {
        ctx.beginPath()
        ctx.arc(point.x, point.y, 3, 0, 2 * Math.PI)
        ctx.fillStyle = index === 0 ? 'red' : 'blue'
        ctx.fill()
      })
    },
    calculateArea() {
      let area = 0
      for (let i = 0; i < this.points.length; i++) {
        const j = (i + 1) % this.points.length
        area += this.points[i].x * this.points[j].y
        area -= this.points[j].x * this.points[i].y
      }
      this.area = Math.abs(area / 2)
    },
    reset() {
      this.points = []
      this.isComplete = false
      this.area = null
      const ctx = this.$refs.canvas.getContext('2d')
      ctx.clearRect(0, 0, this.width, this.height)
    }
  },
  mounted() {
    const ctx = this.$refs.canvas.getContext('2d')
    ctx.strokeStyle = 'black'
    ctx.lineWidth = 2
  }
}
</script>

<style scoped>
canvas {
  border: 1px solid black;
  cursor: crosshair;
}
button {
  margin-right: 10px;
  margin-top: 10px;
}
</style>
