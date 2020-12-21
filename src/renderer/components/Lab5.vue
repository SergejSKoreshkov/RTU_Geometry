<template>
  <div class="lab5">
    <div class="row">
      <div class="col-6">
        <canvas @click="addPointWithClick" @dblclick="breakLine" class="hover" ref="canvas" width="300" height="300"></canvas>
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
            <div class="input-group mb-3">
            <div class="input-group-prepend">
              <span class="input-group-text">Points count:</span>
            </div>
            <input type="number" class="form-control" v-model="pointsCount">
            <div class="input-group-append">
              <button class="btn btn-outline-secondary" type="button" @click="points = new Array(parseInt(pointsCount)).fill(0).map(el => ({ x: Math.trunc(Math.random() * 200) + 50, y: Math.trunc(Math.random() * 200) + 50})); draw()">Spawn</button>
            </div>
            </div>
            <button class="btn btn-primary" @click="points = []; draw()">Clear all</button>
            <button class="btn btn-primary" @click="breakLine">Break line</button>
            <hr>
            <ul class="list-group">
              <li v-for="(point, index) in points" :key="point.x.toString() + point.y.toString()" class="list-group-item">
                <div class="input-group mb-3">
                  <div class="input-group-prepend">
                    <span class="input-group-text" id="basic-addon1">X:</span>
                  </div>
                  <input type="number" class="form-control" placeholder="X" v-model="point.x" @keyup="draw" @change="draw">
                  <div class="input-group-prepend">
                    <span class="input-group-text" id="basic-addon1">Y:</span>
                  </div>
                  <input type="number" class="form-control" placeholder="Y" v-model="point.y" @keyup="draw" @change="draw">
                  <div class="input-group-append">
                    <button class="btn btn-outline-secondary" type="button" @click="removePoint(index); draw()">Delete</button>
                  </div>
                </div>
              </li>
            </ul>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
// const math = require('mathjs')

export default {
  name: 'lab5',
  data () {
    return {
      x: 0,
      y: 0,
      points: [],
      pointsCount: 16
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
      ctx.fillStyle = '#fff'
      ctx.fillRect(0, 0, 300, 300)
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
      points.forEach((pointSet, index, arr) => {
        pointSet.forEach(({ x, y }) => {
          ctx.beginPath()
          ctx.arc(x, y, 2, 0, 2 * Math.PI)
          ctx.stroke()
        })

        setTimeout(() => {
          const mappedPoints = pointSet.map(({ x, y }) => ({ x, y: 300 - y }))
          // Jarvis
          if (mappedPoints.length < 3) return
          let sorted = mappedPoints.sort((a, b) => a.y - b.y)
          // let direction = 'y'
          // if (sorted[0].y === sorted[1].y) {
          // sorted = pointSet.sort((a, b) => a.x - b.x)
          // direction = 'x'
          // }
          let [pointNow, ...restPoints] = sorted
          const lastPoint = {...restPoints[restPoints.length - 1]}

          while (pointNow.x !== lastPoint.x && pointNow.y !== lastPoint.y) {
            ctx.beginPath()
            ctx.moveTo(pointNow.x, 300 - pointNow.y)
            restPoints = restPoints.map(({ x, y }) => {
              const ix = x - pointNow.x
              const iy = y - pointNow.y
              let angle = Math.atan2(iy, ix)
              if (angle < 0) angle = 2 * Math.PI + angle
              console.log(ix, iy, angle)
              return { x, y, a: angle }
            }).sort((a, b) => a.a - b.a)
            pointNow = restPoints.splice(0, 1)[0]
            ctx.lineTo(pointNow.x, 300 - pointNow.y)
            ctx.stroke()
          }
        }, 0)

        setTimeout(() => {
          const mappedPoints = pointSet.map(({ x, y }) => ({ x, y: 300 - y }))
          // Jarvis
          if (mappedPoints.length < 3) return
          let sorted = mappedPoints.sort((a, b) => b.y - a.y)
          // let direction = 'y'
          // if (sorted[0].y === sorted[1].y) {
          // sorted = pointSet.sort((a, b) => a.x - b.x)
          // direction = 'x'
          // }
          let [pointNow, ...restPoints] = sorted
          const lastPoint = {...restPoints[restPoints.length - 1]}

          while (pointNow.x !== lastPoint.x && pointNow.y !== lastPoint.y) {
            ctx.beginPath()
            ctx.moveTo(pointNow.x, 300 - pointNow.y)
            restPoints = restPoints.map(({ x, y }) => {
              const ix = x - pointNow.x
              const iy = y - pointNow.y
              let angle = Math.atan2(-iy, -ix)
              if (angle < 0) angle = 2 * Math.PI + angle
              // if (iy === 0 && x >= 0) angle = 0
              // if (iy === 0 && x < 0) angle = Math.PI
              // if (iy >= 0 && x === 0) angle = Math.PI / 2
              // if (iy < 0 && x === 0) angle = 3 * Math.PI / 2
              console.log(ix, iy, angle)
              return { x, y, a: angle }
            }).sort((a, b) => a.a - b.a)
            pointNow = restPoints.splice(0, 1)[0]
            ctx.lineTo(pointNow.x, 300 - pointNow.y)
            ctx.stroke()
          }
        }, 0)
      })
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
    position: fixed;
    width: 48%;
  }
  .hover:hover {
    box-shadow: 0px 0px 10px;
    cursor: pointer;
  }
</style>