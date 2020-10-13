<template>
  <div class="lab1">
    <div class="row">
      <div class="col-6">
        <canvas @click="addPointWithClick" @dblclick="breakLine" @mousemove="addPointWithMove" @mousedown="mousePressed = true" @mouseup="mousePressed = false" class="hover" ref="canvas" width="300" height="300"></canvas>
      </div>
      <div class="col-6">
        <div class="row">
          <div class="col-12">
            <div class="input-group mb-3">
              <input type="number" v-model="x" class="form-control" placeholder="X">
              <input type="number" v-model="y" class="form-control" placeholder="Y">
              <div class="input-group-append">
                <button @click="addPoint" class="btn btn-outline-secondary" type="button">Add point</button>
              </div>
            </div>
          </div>
        </div>
        <div class="row">
          <div class="col-12">
            <button class="btn btn-primary" @click="breakLine">Break line</button>
            <button class="btn btn-primary" @click="draw">Draw</button>
            <button class="btn btn-primary" @click="points = []">Clear all</button>
            <hr>
            <ul class="list-group">
              <li @click="removePoint(index)" v-for="(point, index) in points" :key="point.x.toString() + point.y.toString()" class="list-group-item">{{`X:${point.x}, Y:${point.y}`}}</li>
            </ul>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
const math = require('mathjs')

const B = (i, n, t) => {
  return (math.factorial(n) / (math.factorial(i) * math.factorial(n - i))) * Math.pow(1 - t, n - i) * Math.pow(t, i)
}

export default {
  name: 'lab1',
  data () {
    return {
      mousePressed: false,
      x: 0,
      y: 0,
      points: [
        { x: 20, y: 30 },
        { x: 23, y: 80 },
        { x: 40, y: 110, break: true },
        { x: 40, y: 110 },
        { x: 62, y: 140 },
        { x: 80, y: 120 },
        { x: 80, y: 115 },
        { x: 85, y: 90 },
        { x: 90, y: 25, break: true },

        { x: 110, y: 20 },
        { x: 110, y: 130, break: true },
        { x: 110, y: 130 },
        { x: 160, y: 70 },
        { x: 110, y: 20, break: true },

        { x: 155, y: 30 },
        { x: 140, y: 20 },
        { x: 155, y: 10 },
        { x: 165, y: 20 },
        { x: 155, y: 30, break: true },

        { x: 155, y: 40 },
        { x: 155, y: 130 }
      ]
    }
  },
  mounted () {
    this.draw()
  },
  methods: {
    addPoint () {
      this.points.push({ x: parseInt(this.x), y: parseInt(this.y) })
    },
    addPointWithClick (event) {
      const { width, height } = event.target
      const size = event.target.getBoundingClientRect()
      const x = parseInt((event.clientX - size.left) / size.width * width)
      const y = parseInt((event.clientY - size.top) / size.height * height)
      if (!this.points.find(el => el.x === x && el.y === y)) this.points.push({ x, y })
      this.draw()
    },
    addPointWithMove (event) {
      if (this.mousePressed) this.addPointWithClick(event)
    },
    removePoint (index) {
      this.points.splice(index, 1)
    },
    breakLine () {
      this.points[this.points.length - 1].break = true
    },
    draw () {
      this.$refs.canvas.width = 300
      this.$refs.canvas.height = 300
      const ctx = this.$refs.canvas.getContext('2d')
      let buffer = ctx.getImageData(0, 0, 300, 300)
      let j = 0
      const points = this.points.reduce((acc, el) => {
        acc[j].push(el)
        if (el.break) {
          j++
          acc.push([])
        }
        return acc
      }, [[]])
      ctx.putImageData(buffer, 0, 0)
      points.forEach(pointSet => {
        pointSet.forEach(({ x, y }, index, arr) => {
          ctx.beginPath()
          if (index === arr.length - 1) ctx.strokeStyle = '#0000ff'
          else ctx.strokeStyle = '#15845c'
          if (index === 0) ctx.fillStyle = '#00ff00'
          else ctx.fillStyle = '#ffffff'
          ctx.arc(x, y, 2, 0, 2 * Math.PI)
          ctx.fill()
          ctx.stroke()
        })
        this.drawCurveFit(ctx, pointSet)
        buffer = ctx.getImageData(0, 0, 300, 300)
        const pointsCount = pointSet.length
        let t = 0
        while (t < 1) {
          let x = 0
          let y = 0
          for (let i = 0; i < pointsCount; i++) {
            x = x + B(i, pointsCount - 1, t) * pointSet[i].x
            y = y + B(i, pointsCount - 1, t) * pointSet[i].y
          }
          x = Math.round(x)
          y = Math.round(y)
          buffer.data[(y * 300 + x) * 4] = 20
          buffer.data[(y * 300 + x) * 4 + 1] = 100
          buffer.data[(y * 300 + x) * 4 + 2] = 80
          buffer.data[(y * 300 + x) * 4 + 3] = 255
          t += 0.005
        }
        ctx.putImageData(buffer, 0, 0)
      })
    },
    drawCurveFit (ctx, points) {
      const polinom = Math.max(points.length - 2, 2)
      const matrix = (new Array(polinom + 1).fill(1)).map(el => new Array(polinom + 1))
      for (let i = 0; i < polinom + 1; i++) {
        for (let j = 0; j < polinom + 1; j++) {
          matrix[i][j] = points.reduce((acc, el) => acc + Math.pow(el.x, polinom * 2 - i - j), 0)
        }
      }
      matrix[polinom][polinom] = polinom + 2

      const matrix2 = new Array(polinom + 1).fill(1).map(el => [])
      for (let i = 0; i < polinom + 1; i++) {
        matrix2[i][0] = points.reduce((acc, el) => acc + Math.pow(el.x, polinom - i) * el.y, 0)
      }
      try {
        const result = math.multiply(math.inv(math.matrix(matrix)), math.matrix(matrix2))._data
        ctx.beginPath()
        ctx.moveTo(0, result[result.length - 1])
        for (let x = 0; x < 1000; x++) {
          const y = result.reduce((acc, el, index, arr) => acc + el * Math.pow(x, arr.length - 1 - index), 0)
          ctx.strokeStyle = '#ff6600'
          ctx.lineTo(x, y)
          ctx.stroke()
        }
      } catch (err) {}
    }
  }
}
</script>

<style scoped>
  canvas {
    border: 1px solid black;
    width: 100%;
    transition: 0.3s ease;
    box-shadow: none;
    border-radius: 3px;
  }
  .hover:hover {
    box-shadow: 0px 0px 10px;
    cursor: pointer;
  }
</style>