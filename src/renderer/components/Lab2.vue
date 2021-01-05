<template>
  <div class="lab2">
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
            <button class="btn btn-primary" @click="points = []; draw()">Clear all</button>
            <button class="btn btn-primary" @click="isHermite = !isHermite; draw()">{{ isHermite ? 'Parabolic interpolation' : 'Hermite interpolation'}}</button>
            <hr>
            <ul class="list-group">
              <li v-for="(point, index) in points" :key="point.x.toString() + point.y.toString()" class="list-group-item">
                <div class="input-group mb-3">
                  <div class="input-group-prepend">
                    <span class="input-group-text" id="basic-addon1">X: {{ point.x }}</span>
                    <span class="input-group-text" id="basic-addon1">Y: {{ point.y }}</span>
                    <span class="input-group-text" id="basic-addon1">Nx: </span>
                  </div>
                  <input type="number" class="form-control" placeholder="Nx" v-model="point.nx" @keyup="draw" @change="draw">
                  <div class="input-group-prepend">
                    <span class="input-group-text" id="basic-addon1">Ny: </span>
                  </div>
                  <input type="number" class="form-control" placeholder="Ny" v-model="point.ny" @keyup="draw" @change="draw">
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
import { debounce } from 'lodash'
const math = require('mathjs')

const constMatrix = math.matrix([
  [2, -4, 2],
  [-3, 4, -1],
  [1, 0, 0]
])

const C = (t, p0, p1, p2, p3) => {
  let a = 1 - t
  a = math.multiply(a, math.matrix([Math.pow(1 / 2 * (1 + t), 2), 1 / 2 * (1 + t), 1]))
  a = math.multiply(a, constMatrix)
  a = math.multiply(a, math.matrix([[p0], [p1], [p2]]))._data[0]

  let b = math.multiply(t, math.matrix([Math.pow(1 / 2 * t, 2), 1 / 2 * t, 1]))
  b = math.multiply(b, constMatrix)
  b = math.multiply(b, math.matrix([[p1], [p2], [p3]]))._data[0]
  return a + b
}

const constMatrixHermite = math.matrix([
  [2, -2, 1, 1],
  [-3, 3, -2, -1],
  [0, 0, 1, 0],
  [1, 0, 0, 0]
])

const CHermite = (t, p0, p1, n0, n1) => {
  let a = math.matrix([Math.pow(t, 3), Math.pow(t, 2), t, 1])
  a = math.multiply(a, constMatrixHermite)
  a = math.multiply(a, math.matrix([[p0], [p1], [n0], [n1]]))._data[0]
  return a
}

export default {
  name: 'lab2',
  data () {
    return {
      isHermite: false,
      mousePressed: false,
      x: 0,
      y: 0,
      points: [
        { x: 5, y: 10, nx: 5, ny: 20 },
        { x: 5, y: 20, nx: 5, ny: 100 },
        { x: 5, y: 100, nx: 30, ny: 160 },
        { x: 30, y: 160, nx: 50, ny: 180 },
        { x: 50, y: 180, nx: 70, ny: 160, break: true },
        { x: 5, y: 100, nx: 100, ny: 100 },
        { x: 30, y: 160, nx: 100, ny: 100 },
        { x: 50, y: 180, nx: 100, ny: 100 },
        { x: 70, y: 160, nx: 100, ny: 100 },
        { x: 95, y: 100, nx: 100, ny: 100 },
        { x: 95, y: 20, nx: 100, ny: 100 },
        { x: 95, y: 10, nx: 100, ny: 100, break: true },
        { x: 80, y: 10, nx: 100, ny: 100 },
        { x: 110, y: 10, nx: 100, ny: 100 },
        { x: 170, y: 70, nx: 100, ny: 100 },
        { x: 110, y: 130, nx: 100, ny: 100 },
        { x: 80, y: 130, nx: 100, ny: 100, break: true },
        { x: 110, y: 140, nx: 100, ny: 100 },
        { x: 110, y: 130, nx: 100, ny: 100 },
        { x: 110, y: 10, nx: 100, ny: 100 },
        { x: 110, y: 0, nx: 100, ny: 100, break: true },
        { x: 200, y: 10, nx: 100, ny: 100 },
        { x: 200, y: 20, nx: 100, ny: 100 },
        { x: 230, y: 30, nx: 100, ny: 100 },
        { x: 260, y: 20, nx: 100, ny: 100 },
        { x: 260, y: 10, nx: 100, ny: 100, break: true },
        { x: 230, y: 40, nx: 100, ny: 100 },
        { x: 230, y: 60, nx: 100, ny: 100 },
        { x: 200, y: 80, nx: 100, ny: 100 },
        { x: 200, y: 160, nx: 100, ny: 100 },
        { x: 230, y: 180, nx: 100, ny: 100 },
        { x: 260, y: 160, nx: 100, ny: 100 },
        { x: 260, y: 80, nx: 100, ny: 100 },
        { x: 230, y: 60, nx: 100, ny: 100 },
        { x: 230, y: 40, nx: 100, ny: 100 }

      ]
    }
  },
  mounted () {
    this.points = this.points.map(point => {
      return {
        ...point,
        nx: Math.trunc(0.9 * point.x + 0.1 * Math.random() * 300),
        ny: Math.trunc(0.9 * point.y + 0.1 * Math.random() * 300)
      }
    })
    this.draw()
  },
  methods: {
    addPoint () {
      this.points.push({ x: parseInt(this.x), y: parseInt(this.y), nx: Math.trunc(Math.random() * 300), ny: Math.trunc(Math.random() * 300) })
    },
    addPointWithClick (event) {
      const { width, height } = event.target
      const size = event.target.getBoundingClientRect()
      const x = parseInt((event.clientX - size.left) / size.width * width)
      const y = parseInt((event.clientY - size.top) / size.height * height)
      if (!this.points.find(el => el.x === x && el.y === y)) this.points.push({ x, y, nx: Math.trunc(Math.random() * 300), ny: Math.trunc(Math.random() * 300) })
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
    draw: debounce(function () { this.drawMethod() }, 50),
    drawMethod () {
      this.$refs.canvas.width = 300
      this.$refs.canvas.height = 300
      const ctx = this.$refs.canvas.getContext('2d')
      ctx.fillStyle = '#ddd'
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
          if (index === arr.length - 1) ctx.strokeStyle = '#0000ff'
          else ctx.strokeStyle = '#15845c'
          if (index === 0) ctx.fillStyle = '#00ff00'
          else ctx.fillStyle = '#ffffff'
          ctx.arc(x, y, 2, 0, 2 * Math.PI)
          ctx.fill()
          ctx.stroke()
        })

        buffer = ctx.getImageData(0, 0, 300, 300)
        pointSet.forEach((point, index, arr) => {
          if (pointSet.length >= 4 && index + 3 < arr.length) {
            let t = 0
            while (t < 1) {
              let x = C(t, pointSet[index].x, pointSet[index + 1].x, pointSet[index + 2].x, pointSet[index + 3].x)
              let y = C(t, pointSet[index].y, pointSet[index + 1].y, pointSet[index + 2].y, pointSet[index + 3].y)
              x = Math.round(x)
              y = Math.round(y)
              buffer.data[(y * 300 + x) * 4] = 230
              buffer.data[(y * 300 + x) * 4 + 1] = 20
              buffer.data[(y * 300 + x) * 4 + 2] = 230
              buffer.data[(y * 300 + x) * 4 + 3] = this.isHermite ? 80 : 255
              t += 0.005
            }
          }
        })
        ctx.putImageData(buffer, 0, 0)

        pointSet.forEach((point, index, arr) => {
          buffer = ctx.getImageData(0, 0, 300, 300)
          if (pointSet.length >= 2 && index + 1 < arr.length) {
            let t = 0
            while (t < 1) {
              let x = CHermite(t, pointSet[index].x, pointSet[index + 1].x, pointSet[index].nx, pointSet[index + 1].nx)
              let y = CHermite(t, pointSet[index].y, pointSet[index + 1].y, pointSet[index].ny, pointSet[index + 1].ny)
              x = Math.round(x)
              y = Math.round(y)
              buffer.data[(y * 300 + x) * 4] = 20
              buffer.data[(y * 300 + x) * 4 + 1] = 20
              buffer.data[(y * 300 + x) * 4 + 2] = 180
              buffer.data[(y * 300 + x) * 4 + 3] = this.isHermite ? 255 : 80
              t += 0.005
            }
          }
          ctx.putImageData(buffer, 0, 0)
          if (this.isHermite) {
            ctx.beginPath()
            ctx.moveTo(pointSet[index].x, pointSet[index].y)
            ctx.strokeStyle = '#00a000'
            ctx.lineTo(pointSet[index].nx, pointSet[index].ny)
            ctx.stroke()
          }
        })
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
  }
  .hover:hover {
    box-shadow: 0px 0px 10px;
    cursor: pointer;
  }
</style>