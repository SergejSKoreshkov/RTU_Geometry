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
              <button class="btn btn-outline-secondary" type="button" @click="points = new Array(parseInt(pointsCount)).fill(0).map(el => ({ x: Math.trunc(Math.random() * 300), y: Math.trunc(Math.random() * 300)})); draw()">Spawn</button>
            </div>
            </div>
            <button class="btn btn-primary" @click="points = []; draw()">Clear all</button>
            <button class="btn btn-primary" @click="breakLine(); draw()">Break line</button>
            <button class="btn btn-primary" @click="isDaC = !isDaC; draw()">{{ !isDaC ? 'Switch to Divide and Conquer' : 'Switch to Direct Development'}}</button>
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

function drawPoints (ctx, points, color, isMapped) {
  points.forEach(({ x, y }) => {
    ctx.strokeStyle = color
    ctx.beginPath()
    ctx.arc(x, isMapped ? 300 - y : y, 2, 0, 2 * Math.PI)
    ctx.stroke()
  })
}

function drawLine (ctx, line, color) {
  ctx.beginPath()
  ctx.strokeStyle = color
  ctx.moveTo(line[0].x, 300 - line[0].y)
  ctx.lineTo(line[1].x, 300 - line[1].y)
  ctx.stroke()
}

function drawTriagles (ctx, triangles, color) {
  triangles.forEach(t => {
    ctx.beginPath()
    ctx.strokeStyle = color
    ctx.moveTo(t[0].x, 300 - t[0].y)
    ctx.lineTo(t[1].x, 300 - t[1].y)
    ctx.lineTo(t[2].x, 300 - t[2].y)
    ctx.lineTo(t[0].x, 300 - t[0].y)
    ctx.stroke()
  })
}

export default {
  name: 'lab6',
  data () {
    return {
      x: 0,
      y: 0,
      points: [],
      pointsCount: 8,
      isDaC: false
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
        drawPoints(ctx, pointSet, '#000', false)

        // Direct Development
        if (!this.isDaC) {
          await this.directDevelopment(ctx, pointSet)
        } else { // Divide and Conquer
          const _points = pointSet.map(({ x, y }) => ({ x, y: 300 - y }))
          const result = [[..._points]]
          let direction = 'x'

          let final = []
          while (result.length) {
            result.forEach((arr, i) => {
              if (arr.length <= 5) {
                final.push([...arr])
                result.splice(i, 1)
                return
              }

              const splitted = [[], []]
              const max = Math.max(...arr.map(p => p[direction]))
              const min = Math.min(...arr.map(p => p[direction]))
              const divider = (max + min) / 2
              arr.forEach(p => {
                if (p[direction] >= divider) splitted[0].push(p)
                else splitted[1].push(p)
              })
              if (splitted[0].length <= 5) final.push([...splitted[0]])
              else result.push([...splitted[0]])
              if (splitted[1].length <= 5) final.push([...splitted[1]])
              else result.push([...splitted[1]])
              result.splice(i, 1)
            })
            direction = direction === 'x' ? 'y' : 'x'
          }

          final = await Promise.all(final.map(async ps => {
            drawPoints(ctx, ps, `#${Math.trunc(Math.random() * 16).toString(16)}${Math.trunc(Math.random() * 9).toString(16)}${Math.trunc(Math.random() * 9).toString(16)}`, true)
            const data = await this.directDevelopment(ctx, ps.map(({ x, y }) => ({ x, y: 300 - y })))
            return data
          }))
          const globalResult = {
            triangles: [],
            usedPoints: []
          }
          final.forEach(arr => {
            if (!arr) return
            globalResult.usedPoints.push(...arr[0])
            globalResult.triangles.push(...arr[2])
          })
          console.log(globalResult)
          await this.directDevelopment(ctx, pointSet, [], globalResult.triangles, true)
        }
      })
    },
    async directDevelopment (ctx, pointSet, _usedPoints = [], _triangles = [], swap = false) {
      // Step 1
      const _lineQueue = [[]]

      const _points = pointSet.map(({ x, y }) => ({ x, y: 300 - y }))
      if (_points.length < 3) return
      let sorted = _points.sort((a, b) => a.y - b.y || a.x - b.x)
      let [pointNow, ...restPoints] = sorted
      _points.push({...pointNow})
      _lineQueue[0].push({...pointNow}) // Point one

      restPoints = restPoints.map(({ x, y }) => {
        const ix = x - pointNow.x
        const iy = y - pointNow.y
        let angle = Math.atan2(iy, ix)
        if (angle < 0) angle = 2 * Math.PI + angle
        return { x, y, a: angle }
      }).sort((a, b) => a.a - b.a)
      pointNow = restPoints.splice(0, 1)[0]
      _points.push({...pointNow})
      _lineQueue[0].push({...pointNow}) // Point two

      // Base line ready ---------------------------------------------------

      while (_lineQueue.length !== 0) {
        // ctx.fillRect(0, 0, 300, 300)
        drawPoints(ctx, _points, '#000', true)
        drawTriagles(ctx, _triangles, '#000')

        const line = _lineQueue.splice(0, 1)[0]
        _usedPoints.push(line[0], line[1])

        const triangle = _triangles.filter(triangle => {
          return (
            (triangle[0].x === line[0].x && triangle[0].y === line[0].y) ||
                (triangle[1].x === line[0].x && triangle[1].y === line[0].y) ||
                (triangle[2].x === line[0].x && triangle[2].y === line[0].y)
          ) && (
            (triangle[0].x === line[1].x && triangle[0].y === line[1].y) ||
                (triangle[1].x === line[1].x && triangle[1].y === line[1].y) ||
                (triangle[2].x === line[1].x && triangle[2].y === line[1].y)
          )
        })

        let filterFn = () => true
        drawTriagles(ctx, triangle, '#0f0')

        if (triangle.length > 1) {
          continue
        }
        if (triangle.length === 1) {
          const k = (line[1].y - line[0].y) / (line[1].x - line[0].x)
          const b = -k * line[0].x + line[0].y
          const lineFn = (x) => k * x + b

          const lastPoint = triangle[0].find(p =>
            !line.some(lp => p.x === lp.x && p.y === lp.y)
          )

          if (lastPoint && lastPoint.y > lineFn(lastPoint.x)) {
            filterFn = p => p.y < lineFn(p.x)
          }

          if (lastPoint && lastPoint.y < lineFn(lastPoint.x)) {
            filterFn = p => p.y > lineFn(p.x)
          }
        }

        const nonLinePoints = _points
          .filter(filterFn)
          .filter(p =>
            !_usedPoints.some(up =>
              up.x === p.x && up.y === p.y
            )
          )

        drawPoints(ctx, nonLinePoints, '#f0f', true)
        drawLine(ctx, line, '#f00')

        if (nonLinePoints.length > 0) {
          const nonLinePointsWithAngle = nonLinePoints.map(p => {
            const a = Math.sqrt(Math.pow(p.x - line[0].x, 2) + Math.pow(p.y - line[0].y, 2))
            const b = Math.sqrt(Math.pow(p.x - line[1].x, 2) + Math.pow(p.y - line[1].y, 2))
            const c = Math.sqrt(Math.pow(line[0].x - line[1].x, 2) + Math.pow(line[0].y - line[1].y, 2))

            const angle = Math.acos((Math.pow(a, 2) + Math.pow(b, 2) - Math.pow(c, 2)) / (2 * a * b))
            return { ...p, a: angle }
          }).sort((a, b) => b.a - a.a)

          const bestNewPoint = nonLinePointsWithAngle[0]
          _lineQueue.push([line[0], bestNewPoint])
          _lineQueue.push([bestNewPoint, line[1]])

          // const newTriangle = [...line, bestNewPoint]

          _triangles.push([...line, bestNewPoint])
        }

        await new Promise(resolve => setTimeout(resolve, 100))
        // await new Promise(resolve => document.addEventListener('click', resolve))
      }
      return [_usedPoints, _lineQueue, _triangles]
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