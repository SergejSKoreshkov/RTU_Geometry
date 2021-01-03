<template>
  <div class="lab6">
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
            <button class="btn btn-primary" @click="isDaB = !isDaB; draw()">{{ isDaB ? 'Switch to Departments and Boards' : 'Switch to Direct Development'}}</button>
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
  name: 'lab6',
  data () {
    return {
      x: 0,
      y: 0,
      points: [],
      pointsCount: 16,
      isDaB: false
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

        // Direct Development
        if (!this.isDaB) {
          // Step 1
          const startPointsSet = [[]]
          const mappedPoints = pointSet.map(({ x, y }) => ({ x, y: 300 - y }))
          if (mappedPoints.length < 3) return
          let sorted = mappedPoints.sort((a, b) => a.y - b.y || a.x - b.x)
          let [pointNow, ...restPoints] = sorted
          startPointsSet[0].push({...pointNow}) // Point one

          restPoints = restPoints.map(({ x, y }) => {
            const ix = x - pointNow.x
            const iy = y - pointNow.y
            let angle = Math.atan2(iy, ix)
            if (angle < 0) angle = 2 * Math.PI + angle
            return { x, y, a: angle }
          }).sort((a, b) => a.a - b.a)
          pointNow = restPoints.splice(0, 1)[0]
          startPointsSet[0].push({...pointNow}) // Point two
          // startPointsSet

          const ignorePoints = [...startPointsSet[0]]
          while (true) {
            console.log('START POINTS SET', startPointsSet)
            // Step 2
            const triangPoints = mappedPoints.filter(point => {
              return ignorePoints.every(ignorePoint => {
                return point.x !== ignorePoint.x && point.y !== ignorePoint.y
              })
            })
            console.log('STEP 2', triangPoints)

            if (triangPoints.length === 0) {
              console.log('END', triangPoints)
              break
            }

            // Step 3
            const pointsWithAngle = triangPoints.map(point => {
              const a = Math.sqrt(Math.pow(point.x - startPoints[0].x, 2) + Math.pow(point.y - startPoints[0].y, 2))
              const b = Math.sqrt(Math.pow(point.x - startPoints[1].x, 2) + Math.pow(point.y - startPoints[1].y, 2))
              const c = Math.sqrt(Math.pow(startPoints[0].x - startPoints[1].x, 2) + Math.pow(startPoints[0].y - startPoints[1].y, 2))

              const angle = Math.acos((Math.pow(a, 2) + Math.pow(b, 2) - Math.pow(c, 2)) / (2 * a * b))
              return { ...point, a: angle }
            }).sort((a, b) => b.a - a.a)

            const bestPoint = pointsWithAngle[0]
            ignorePoints.push(bestPoint)
            console.log('STEP 3', bestPoint)
            // bestPoint

            // Step 4
            startPointsSet.push([bestPoint, startPoints[0]])
            startPointsSet.push([bestPoint, startPoints[1]])
            index++
            console.log('STEP 4', startPointsSet)

            // Draw
            ctx.beginPath()
            ctx.strokeStyle = '#000'
            ctx.moveTo(bestPoint.x, 300 - bestPoint.y)
            ctx.lineTo(startPoints[0].x, 300 - startPoints[0].y)
            ctx.moveTo(bestPoint.x, 300 - bestPoint.y)
            ctx.lineTo(startPoints[1].x, 300 - startPoints[1].y)
            ctx.stroke()
            ctx.beginPath()
            ctx.strokeStyle = '#ff0000'
            ctx.moveTo(startPoints[1].x, 300 - startPoints[1].y)
            ctx.lineTo(startPoints[0].x, 300 - startPoints[0].y)
            ctx.stroke()

            await new Promise(resolve => setTimeout(resolve, 1000))
          }
        } else { // Departments and Boards
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