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
            <button class="btn btn-primary" @click="breakLine(); draw()">Break line</button>
            <button class="btn btn-primary" @click="isFast = !isFast; draw()">{{ isFast ? 'Switch to Jarvis' : 'Switch to Fast Shell'}}</button>
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
      pointsCount: 16,
      isFast: false
    }
  },
  mounted () {
    this.draw()
  },
  methods: {
    addPoint () {
      this.points.push({ x: parseInt(this.x), y: parseInt(this.y) })
      this.draw()
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
      points.forEach(async (pointSet, index, arr) => {
        pointSet.forEach(({ x, y }) => {
          ctx.beginPath()
          ctx.arc(x, y, 2, 0, 2 * Math.PI)
          ctx.stroke()
        })

        // Jarvis
        if (!this.isFast) {
          const fns = [
            [Math.atan2, (a, b) => a.y - b.y || a.x - b.x],
            [(x, y) => Math.atan2(-x, -y), (a, b) => b.y - a.y || b.x - a.x]
          ]
          fns.forEach(async ([angleFn, sortFn]) => {
            const mappedPoints = pointSet.map(({ x, y }) => ({ x, y: 300 - y }))
            if (mappedPoints.length < 3) return
            let sorted = mappedPoints.sort(sortFn)
            let [pointNow, ...restPoints] = sorted
            const lastPoint = {...restPoints[restPoints.length - 1]}
            while (!(pointNow.x === lastPoint.x && pointNow.y === lastPoint.y)) {
              ctx.beginPath()
              ctx.moveTo(pointNow.x, 300 - pointNow.y)
              restPoints = restPoints.map(({ x, y }) => {
                const ix = x - pointNow.x
                const iy = y - pointNow.y
                let angle = angleFn(iy, ix)
                if (angle < 0) angle = 2 * Math.PI + angle
                return { x, y, a: angle }
              }).sort((a, b) => a.a - b.a)
              pointNow = restPoints.splice(0, 1)[0]
              ctx.lineTo(pointNow.x, 300 - pointNow.y)
              ctx.stroke()
              await new Promise(resolve => setTimeout(resolve, 100))
            }
          })
        } else { // Fast algo
          const mappedPoints = pointSet.map(({ x, y }) => ({ x, y: 300 - y }))
          if (mappedPoints.length < 3) return
          const sorted = mappedPoints.sort((a, b) => a.x - b.x)
          const left = {...sorted[0]}
          const right = {...sorted[sorted.length - 1]}

          let linesPoints = [[left, right]]
          while (linesPoints.length !== 0) {
            const [[left, right]] = [...linesPoints.splice(0, 1)]
            ctx.beginPath()
            ctx.moveTo(left.x, 300 - left.y)
            ctx.lineTo(right.x, 300 - right.y)
            ctx.stroke()
            const k = (right.y - left.y) / (right.x - left.x)
            const b = -k * left.x + left.y
            console.log(k, b)
            const line = (x) => k * x + b
            const [ bestPoint ] = mappedPoints
              .filter(p => p.y > line(p.x))
              .map(p => ({
                distance: Math.abs(k * p.x - p.y + b) / Math.sqrt(Math.pow(k, 2) + 1),
                ...p
              }))
              .sort((a, b) => b.distance - a.distance)

            if (bestPoint && bestPoint.distance > 0.5) {
              const crossX = (-1 * (-1 * bestPoint.x - k * bestPoint.y) - k * b) / (Math.pow(k, 2) + 1)
              ctx.beginPath()
              ctx.moveTo(crossX, 300 - line(crossX))
              ctx.lineTo(bestPoint.x, 300 - bestPoint.y)
              ctx.stroke()
              linesPoints.push([left, bestPoint])
              linesPoints.push([bestPoint, right])
            }
            await new Promise(resolve => setTimeout(resolve, 100))
          }

          linesPoints = [[left, right]]
          while (linesPoints.length !== 0) {
            const [[left, right]] = [...linesPoints.splice(0, 1)]
            ctx.beginPath()
            ctx.moveTo(left.x, 300 - left.y)
            ctx.lineTo(right.x, 300 - right.y)
            ctx.stroke()
            const k = (right.y - left.y) / (right.x - left.x)
            const b = -k * left.x + left.y
            console.log(k, b)
            const line = (x) => k * x + b
            const [ bestPoint ] = mappedPoints
              .filter(p => p.y < line(p.x))
              .map(p => ({
                distance: Math.abs(k * p.x - p.y + b) / Math.sqrt(Math.pow(k, 2) + 1),
                ...p
              }))
              .sort((a, b) => b.distance - a.distance)

            if (bestPoint && bestPoint.distance > 0.5) {
              const crossX = (-1 * (-1 * bestPoint.x - k * bestPoint.y) - k * b) / (Math.pow(k, 2) + 1)
              ctx.beginPath()
              ctx.moveTo(crossX, 300 - line(crossX))
              ctx.lineTo(bestPoint.x, 300 - bestPoint.y)
              ctx.stroke()
              linesPoints.push([left, bestPoint])
              linesPoints.push([bestPoint, right])
            }
            await new Promise(resolve => setTimeout(resolve, 100))
          }
        }
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