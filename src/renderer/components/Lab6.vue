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
            <button class="btn btn-primary" @click="isDaB = !isDaB; draw()">{{ isDaB ? 'Switch to Departments and Boards' : 'Switch to Incremental'}}</button>
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

function ptInTriangle (p, p0, p1, p2) {
  const A = 1 / 2 * (-p1.y * p2.x + p0.y * (-p1.x + p2.x) + p0.x * (p1.y - p2.y) + p1.x * p2.y)
  const sign = A < 0 ? -1 : 1
  const s = (p0.y * p2.x - p0.x * p2.y + (p2.y - p0.y) * p.x + (p0.x - p2.x) * p.y) * sign
  const t = (p0.x * p1.y - p0.y * p1.x + (p0.y - p1.y) * p.x + (p1.x - p0.x) * p.y) * sign

  return s > 0 && t > 0 && (s + t) < 2 * A * sign
}

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
        // Incremental
        if (!this.isDaB) {
          // Step 0
          const mappedPoints = pointSet.map(({ x, y }) => ({ x, y: 300 - y }))
          if (mappedPoints.length < 3) return

          // Step 1
          const [p1, p2, p3, ...restPoints] = mappedPoints
          const triangles = [[p1, p2, p3]]
          await new Promise(resolve => setTimeout(resolve, 100))
          // Step 2
          for (let i = 0; i < restPoints.length; i++) {
            ctx.fillRect(0, 0, 300, 300)
            pointSet.forEach(({ x, y }) => {
              ctx.beginPath()
              ctx.strokeStyle = '#000'
              ctx.arc(x, y, 2, 0, 2 * Math.PI)
              ctx.stroke()
            })
            triangles.forEach(triangle => {
              ctx.beginPath()
              ctx.moveTo(triangle[0].x, 300 - triangle[0].y)
              ctx.lineTo(triangle[1].x, 300 - triangle[1].y)
              ctx.lineTo(triangle[2].x, 300 - triangle[2].y)
              ctx.lineTo(triangle[0].x, 300 - triangle[0].y)
              ctx.stroke()
            })
            // Step 3
            const point = restPoints[i]
            ctx.beginPath()
            ctx.strokeStyle = '#f00'
            ctx.arc(point.x, 300 - point.y, 2, 0, 2 * Math.PI)
            ctx.stroke()
            const [ triangle ] = triangles
              .map(points => {
                const vector = {
                  x: point.x + points[0].x + points[1].x + points[2].x - 3 * point.x,
                  y: point.y + points[0].y + points[1].y + points[2].y - 3 * point.y
                }
                const distance = Math.sqrt(Math.pow(vector.x - point.x, 2) + Math.pow(vector.y - point.y, 2))
                console.log('DISTANCE FROM', point, 'TO', vector, distance)
                return [...points, distance]
              })
              .sort((a, b) => a[3] - b[3])

            ctx.beginPath()
            ctx.strokeStyle = '#0f0'
            ctx.moveTo(triangle[0].x, 300 - triangle[0].y)
            ctx.lineTo(triangle[1].x, 300 - triangle[1].y)
            ctx.lineTo(triangle[2].x, 300 - triangle[2].y)
            ctx.lineTo(triangle[0].x, 300 - triangle[0].y)
            ctx.stroke()
            await new Promise(resolve => setTimeout(resolve, 100))

            const newTriangles = []
            // Step 4
            if (triangle.some(intPoint => intPoint.x === point.x && intPoint.y === point.y)) {
              console.log('NOTHING')
            } else {
              const sides = [
                [triangle[0], triangle[1]],
                [triangle[1], triangle[2]],
                [triangle[2], triangle[0]]
              ]
              const pointOnSide = sides.find(side => {
                const k = (side[0].y - side[1].y) / (side[0].x - side[1].x)
                const b = -k * side[1].x + side[1].y
                const line = (x) => k * x + b
                console.log(point.y, line(point.x))
                return point.y === line(point.x)
              })
              console.log(pointOnSide)
              if (pointOnSide) {
                const toDelete = []
                for (let j = 0; j < triangles.length; j++) {
                  if (
                    (
                      (triangles[j][0].x === pointOnSide[0].x && triangles[j][0].y === pointOnSide[0].y) ||
                      (triangles[j][1].x === pointOnSide[0].x && triangles[j][1].y === pointOnSide[0].y) ||
                      (triangles[j][2].x === pointOnSide[0].x && triangles[j][2].y === pointOnSide[0].y)
                    ) && (
                      (triangles[j][0].x === pointOnSide[1].x && triangles[j][0].y === pointOnSide[1].y) ||
                      (triangles[j][1].x === pointOnSide[1].x && triangles[j][1].y === pointOnSide[1].y) ||
                      (triangles[j][2].x === pointOnSide[1].x && triangles[j][2].y === pointOnSide[1].y)
                    )
                  ) {
                    const toChange = [...triangles[j]]
                    const triangOne = toChange.map(p => {
                      if (p.x === pointOnSide[0].x && p.y === pointOnSide[0].y) {
                        return { ...point }
                      }
                      return p
                    })
                    const triangTwo = toChange.map(p => {
                      if (p.x === pointOnSide[1].x && p.y === pointOnSide[1].y) {
                        return { ...point }
                      }
                      return p
                    })
                    console.log(toChange, triangOne, triangTwo)
                    toDelete.push(j)
                    newTriangles.push(triangOne)
                    newTriangles.push(triangTwo)
                  }
                }
                toDelete.forEach(index => triangles.splice(index, 1))
              } else if (ptInTriangle(point, triangle[0], triangle[1], triangle[2])) {
                const index = triangles.findIndex(intTriangle => {
                  if (
                    intTriangle[0].x === triangle[0].x &&
                      intTriangle[1].x === triangle[1].x &&
                      intTriangle[2].x === triangle[2].x &&
                      intTriangle[0].y === triangle[0].y &&
                      intTriangle[1].y === triangle[1].y &&
                      intTriangle[2].y === triangle[2].y
                  ) {
                    newTriangles.push([intTriangle[0], intTriangle[1], point])
                    newTriangles.push([intTriangle[1], intTriangle[2], point])
                    newTriangles.push([intTriangle[2], intTriangle[0], point])
                    return true
                  }
                  return false
                })
                triangles.splice(index, 1)
              } else {
                const pairs = [
                  [triangle[0], triangle[1]],
                  [triangle[1], triangle[2]],
                  [triangle[2], triangle[0]]
                ]
                const newTrPoints = pairs.map(p => {
                  const a = Math.sqrt(Math.pow(point.x - p[0].x, 2) + Math.pow(point.y - p[0].y, 2))
                  const b = Math.sqrt(Math.pow(point.x - p[1].x, 2) + Math.pow(point.y - p[1].y, 2))
                  const c = Math.sqrt(Math.pow(p[0].x - p[1].x, 2) + Math.pow(p[0].y - p[1].y, 2))

                  const angle = Math.acos((Math.pow(a, 2) + Math.pow(b, 2) - Math.pow(c, 2)) / (2 * a * b))
                  return [p[0], p[1], angle]
                }).sort((a, b) => b[2] - a[2])

                console.log('NEW', point, newTrPoints[0][0], newTrPoints[0][1])
                newTriangles.push([point, newTrPoints[0][0], newTrPoints[0][1]])
              }
            }
            console.log('NEW TRIANGLES', newTriangles)
            newTriangles.forEach(triangle => {
              ctx.beginPath()
              ctx.strokeStyle = '#00f'
              ctx.moveTo(triangle[0].x, 300 - triangle[0].y)
              ctx.lineTo(triangle[1].x, 300 - triangle[1].y)
              ctx.lineTo(triangle[2].x, 300 - triangle[2].y)
              ctx.lineTo(triangle[0].x, 300 - triangle[0].y)
              ctx.stroke()
            })
            // Step 5
            triangles.push(...newTriangles)
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