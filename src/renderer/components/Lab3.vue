<template>
  <div class="lab3">
    <h4>Internal space x, y, z is in interval [-100;100]</h4>
    <h5>[-1;1] mapped to [-100;100]</h5>
    <h6 style="color: blue">Use 'q' and 'e' keys to change dZ. dZ: {{ dz }}</h6>
    <h6 style="color: blue">Use 'w' and 's' keys to change aX. aX: {{ aX }}</h6>
    <h6 style="color: blue">Use 'a' and 'd' keys to change aY. aY: {{ aY }}</h6>
    <h6 style="color: blue">Use 'z' and 'c' keys to change aZ. aZ: {{ aZ }}</h6>
    <div class="row">
      <div class="col-6">
        <div class="form-check">
          <input class="form-check-input" type="radio" name="type" value="isometric" v-model="type" @change="this.draw">
          <label class="form-check-label" for="type">
            Isometric projection
          </label>
        </div>
        <div class="form-check">
          <input class="form-check-input" type="radio" name="type" value="onepoint" v-model="type" @change="this.draw">
          <label class="form-check-label" for="type">
            One point perspective
          </label>
        </div>
        <div class="form-check">
          <input class="form-check-input" type="radio" name="type" value="cabinet" v-model="type" @change="this.draw">
          <label class="form-check-label" for="type">
            Cabinet projection
          </label>
        </div>
        <div class="form-check">
          <input class="form-check-input" type="radio" name="type" value="cavalier" v-model="type" @change="this.draw">
          <label class="form-check-label" for="type">
            Cavalier projection
          </label>
        </div>
      </div>
      <div class="col-6">
        <div class="form-check">
          <input class="form-check-input" type="radio" name="cubeSelect" :value="cube1" v-model="cube" @change="this.draw">
          <label class="form-check-label" for="cubeSelect">
            Use cube x,y,z in [-100;100]
          </label>
        </div>
        <div class="form-check">
          <input class="form-check-input" type="radio" name="cubeSelect" :value="cube2" v-model="cube" @change="this.draw">
          <label class="form-check-label" for="cubeSelect">
            Use cube x,y,z in [-0;100]
          </label>
        </div>
        <div class="form-check">
          <input class="form-check-input" type="radio" name="cubeSelect" :value="cube3" v-model="cube" @change="this.draw">
          <label class="form-check-label" for="cubeSelect">
            Use cube x,y,z in [-50;50]
          </label>
        </div>

        <div class="form-check">
          <input class="form-check-input" type="radio" name="bezierSelect" :value="bezier1" v-model="bezier" @change="this.draw">
          <label class="form-check-label" for="cubeSelect">
            Use bezier 1 (long line)
          </label>
        </div>
        <div class="form-check">
          <input class="form-check-input" type="radio" name="bezierSelect" :value="bezier2" v-model="bezier" @change="this.draw">
          <label class="form-check-label" for="cubeSelect">
            Use bezier 2 (short line)
          </label>
        </div>
      </div>
    </div>
    <div class="row">
      <div class="col-8">
        <canvas class="hover" ref="canvas" width="200" height="200"></canvas>
      </div>
    </div>
  </div>
</template>

<script>
const math = require('mathjs')

const B = (i, n, t) => {
  return (math.factorial(n) / (math.factorial(i) * math.factorial(n - i))) * Math.pow(1 - t, n - i) * Math.pow(t, i)
}

const rotateX = (point, angle) => {
  point = point.map(el => el - 100)
  const pointMatrix = math.matrix([
    [point[0]],
    [point[1]],
    [point[2]],
    [1]
  ])
  let result = math.matrix([
    [1, 0, 0, 0],
    [0, Math.cos(angle), -Math.sin(angle), 0],
    [0, Math.sin(angle), Math.cos(angle), 0],
    [0, 0, 0, 1]
  ])
  result = math.multiply(result, pointMatrix)
  return result._data.map(el => el[0] + 100)
}

const rotateY = (point, angle) => {
  point = point.map(el => el - 100)
  const pointMatrix = math.matrix([
    [point[0]],
    [point[1]],
    [point[2]],
    [1]
  ])
  let result = math.matrix([
    [Math.cos(angle), 0, Math.sin(angle), 0],
    [0, 1, 0, 0],
    [-Math.sin(angle), 0, Math.cos(angle), 0],
    [0, 0, 0, 1]
  ])
  result = math.multiply(result, pointMatrix)
  return result._data.map(el => el[0] + 100)
}

const rotateZ = (point, angle) => {
  point = point.map(el => el - 100)
  const pointMatrix = math.matrix([
    [point[0]],
    [point[1]],
    [point[2]],
    [1]
  ])
  let result = math.matrix([
    [Math.cos(angle), -Math.sin(angle), 0, 0],
    [Math.sin(angle), Math.cos(angle), 0, 0],
    [0, 0, 1, 0],
    [0, 0, 0, 1]
  ])
  result = math.multiply(result, pointMatrix)
  return result._data.map(el => el[0] + 100)
}

const isometricMatrix = math.multiply(1 / Math.sqrt(6), math.matrix([
  [Math.sqrt(3), 0, -Math.sqrt(3)],
  [1, 2, 1],
  [Math.sqrt(2), Math.sqrt(2), Math.sqrt(2)]
]))

const projectIsometric = (point) => {
  point = point.map(el => el - 100)
  const pointMatrix = math.matrix([
    [point[0]],
    [point[1]],
    [point[2]]
  ])
  let result = math.multiply(isometricMatrix, pointMatrix)
  result = math.multiply(math.matrix([
    [1, 0, 0],
    [0, 1, 0],
    [0, 0, 0]
  ]), result)
  return result._data.map(el => el[0] + 100)
}

const atan2 = Math.atan(2)

const obliqueProjection = (point, scale, angle) => {
  point = point.map(el => el - 100)
  const x = point[0] + scale * Math.cos(angle) * point[2] + 100
  const y = point[1] + scale * Math.sin(angle) * point[2] + 100
  return [x, y]
}

export default {
  name: 'lab3',
  data () {
    return {
      dz: 200,
      aX: 0,
      aY: 0,
      aZ: 0,
      type: 'isometric',
      bezier: [],
      bezier1: [
        [[0, 0, 0], [50, 90, 80]],
        [[50, 90, 80], [150, 140, 160]],
        [[150, 140, 160], [0, 0, 0]]
      ],
      bezier2: [
        [[50, 50, 50], [50, 150, 150]],
        [[50, 150, 150], [150, 50, 150]],
        [[150, 50, 150], [50, 50, 50]]
      ],
      cube: [],
      cube1: [
        [[200, 200, 200], [200, 0, 200]],
        [[200, 0, 200], [0, 0, 200]],
        [[0, 0, 200], [0, 200, 200]],
        [[0, 200, 200], [200, 200, 200]],

        [[200, 200, 0], [200, 0, 0]],
        [[200, 0, 0], [0, 0, 0]],
        [[0, 0, 0], [0, 200, 0]],
        [[0, 200, 0], [200, 200, 0]],

        [[200, 200, 200], [200, 200, 0]],
        [[200, 0, 200], [200, 0, 0]],
        [[0, 0, 200], [0, 0, 0]],
        [[0, 200, 200], [0, 200, 0]]
      ],
      cube2: [
        [[200, 200, 200], [200, 100, 200]],
        [[200, 100, 200], [100, 100, 200]],
        [[100, 100, 200], [100, 200, 200]],
        [[100, 200, 200], [200, 200, 200]],

        [[200, 200, 100], [200, 100, 100]],
        [[200, 100, 100], [100, 100, 100]],
        [[100, 100, 100], [100, 200, 100]],
        [[100, 200, 100], [200, 200, 100]],

        [[200, 200, 200], [200, 200, 100]],
        [[200, 100, 200], [200, 100, 100]],
        [[100, 100, 200], [100, 100, 100]],
        [[100, 200, 200], [100, 200, 100]]
      ],
      cube3: [
        [[150, 150, 150], [150, 50, 150]],
        [[150, 50, 150], [50, 50, 150]],
        [[50, 50, 150], [50, 150, 150]],
        [[50, 150, 150], [150, 150, 150]],

        [[150, 150, 50], [150, 50, 50]],
        [[150, 50, 50], [50, 50, 50]],
        [[50, 50, 50], [50, 150, 50]],
        [[50, 150, 50], [150, 150, 50]],

        [[150, 150, 150], [150, 150, 50]],
        [[150, 50, 150], [150, 50, 50]],
        [[50, 50, 150], [50, 50, 50]],
        [[50, 150, 150], [50, 150, 50]]
      ]
    }
  },
  mounted () {
    this.cube = this.cube1
    this.draw()
  },
  methods: {
    onKey (e) {
      switch (e.key) {
        case 'q': this.dz -= 5; break
        case 'e': this.dz += 5; break
        case 'a': this.aY += 0.05; break
        case 'd': this.aY -= 0.05; break
        case 'w': this.aX -= 0.05; break
        case 's': this.aX += 0.05; break
        case 'z': this.aZ -= 0.05; break
        case 'c': this.aZ += 0.05; break
      }
      this.draw()
    },
    draw () {
      switch (this.type) {
        case 'isometric': this.drawIsometricProjection(); return
        case 'onepoint': this.drawOnePointPerspective(); return
        case 'cabinet': this.drawObliqueProjection(1 / 2, -atan2); return
        case 'cavalier': this.drawObliqueProjection(1, -0.785398)
      }
    },
    drawObliqueProjection (scale, angle) {
      const canvas = this.$refs.canvas
      document.addEventListener('keypress', this.onKey)
      const ctx = canvas.getContext('2d')
      ctx.fillStyle = '#fff'
      ctx.fillRect(0, 0, 200, 200)
      const buffer = ctx.getImageData(0, 0, 200, 200)

      this.cube.forEach(line => {
        const changeAxis = math.add(line[0], line[1].map(el => -el)).findIndex(el => el !== 0)
        const change = line[1][changeAxis] - line[0][changeAxis]
        let t = 0.0
        while (t < 1) {
          let points = [...line[0]]
          points[changeAxis] = line[0][changeAxis] + t * change
          points = rotateX(points, this.aX)
          points = rotateY(points, this.aY)
          points = rotateZ(points, this.aZ)
          let [x, y] = obliqueProjection(points, scale, angle)
          x = Math.round(x)
          y = Math.round(y)
          if (x >= 0 && y >= 0 && x < 200 && y < 200) {
            buffer.data[(y * 200 + x) * 4] = 20
            buffer.data[(y * 200 + x) * 4 + 1] = 100
            buffer.data[(y * 200 + x) * 4 + 2] = 80
            buffer.data[(y * 200 + x) * 4 + 3] = 255
          }
          t += 0.01
        }
      })

      const pointsCount = this.bezier.length
      let t = 0
      while (t < 1) {
        let x = 0
        let y = 0
        let z = 0
        for (let i = 0; i < pointsCount; i++) {
          x = x + B(i, pointsCount - 1, t) * this.bezier[i][0][0]
          y = y + B(i, pointsCount - 1, t) * this.bezier[i][0][1]
          z = z + B(i, pointsCount - 1, t) * this.bezier[i][0][2]
        }
        x = Math.round(x)
        y = Math.round(y)
        z = Math.round(z)
        let points = rotateX([x, y, z], this.aX)
        points = rotateY(points, this.aY)
        points = rotateZ(points, this.aZ)
        points = obliqueProjection(points, scale, angle)
        x = Math.round(points[0])
        y = Math.round(points[1])
        if (x >= 0 && y >= 0 && x < 200 && y < 200) {
          buffer.data[(y * 200 + x) * 4] = 20
          buffer.data[(y * 200 + x) * 4 + 1] = 100
          buffer.data[(y * 200 + x) * 4 + 2] = 80
          buffer.data[(y * 200 + x) * 4 + 3] = 255
        }
        t += 0.01
      }

      this.bezier.forEach((line, index, arr) => {
        if (index === arr.length - 1) return
        const change = [line[0][0] - line[1][0], line[0][1] - line[1][1], line[0][2] - line[1][2]]
        let t = 0.0
        while (t < 1) {
          let points = [...line[0]]
          points = points.map((point, index) => {
            return point - change[index] * t
          })
          points = rotateX([...points], this.aX)
          points = rotateY(points, this.aY)
          points = rotateZ(points, this.aZ)
          points = obliqueProjection(points, scale, angle)
          const x = Math.round(points[0])
          const y = Math.round(points[1])
          if (x >= 0 && y >= 0 && x < 200 && y < 200) {
            buffer.data[(y * 200 + x) * 4] = 20
            buffer.data[(y * 200 + x) * 4 + 1] = 100
            buffer.data[(y * 200 + x) * 4 + 2] = 80
            buffer.data[(y * 200 + x) * 4 + 3] = 255
          }
          t += 0.01
        }
      })

      ctx.putImageData(buffer, 0, 0)
    },
    drawIsometricProjection () {
      const canvas = this.$refs.canvas
      document.addEventListener('keypress', this.onKey)
      const ctx = canvas.getContext('2d')
      ctx.fillStyle = '#fff'
      ctx.fillRect(0, 0, 200, 200)
      const buffer = ctx.getImageData(0, 0, 200, 200)

      this.cube.forEach(line => {
        const changeAxis = math.add(line[0], line[1].map(el => -el)).findIndex(el => el !== 0)
        const change = line[1][changeAxis] - line[0][changeAxis]
        let t = 0.0
        while (t < 1) {
          let points = [...line[0]]
          points[changeAxis] = line[0][changeAxis] + t * change
          points = rotateX(points, this.aX)
          points = rotateY(points, this.aY)
          points = rotateZ(points, this.aZ)
          let [x, y] = projectIsometric(points)
          x = Math.round(x)
          y = Math.round(y)
          if (x >= 0 && y >= 0 && x < 200 && y < 200) {
            buffer.data[(y * 200 + x) * 4] = 20
            buffer.data[(y * 200 + x) * 4 + 1] = 100
            buffer.data[(y * 200 + x) * 4 + 2] = 80
            buffer.data[(y * 200 + x) * 4 + 3] = 255
          }
          t += 0.01
        }
      })

      const pointsCount = this.bezier.length
      let t = 0
      while (t < 1) {
        let x = 0
        let y = 0
        let z = 0
        for (let i = 0; i < pointsCount; i++) {
          x = x + B(i, pointsCount - 1, t) * this.bezier[i][0][0]
          y = y + B(i, pointsCount - 1, t) * this.bezier[i][0][1]
          z = z + B(i, pointsCount - 1, t) * this.bezier[i][0][2]
        }
        x = Math.round(x)
        y = Math.round(y)
        z = Math.round(z)
        let points = rotateX([x, y, z], this.aX)
        points = rotateY(points, this.aY)
        points = rotateZ(points, this.aZ)
        points = projectIsometric(points)
        x = Math.round(points[0])
        y = Math.round(points[1])
        if (x >= 0 && y >= 0 && x < 200 && y < 200) {
          buffer.data[(y * 200 + x) * 4] = 20
          buffer.data[(y * 200 + x) * 4 + 1] = 100
          buffer.data[(y * 200 + x) * 4 + 2] = 80
          buffer.data[(y * 200 + x) * 4 + 3] = 255
        }
        t += 0.01
      }

      this.bezier.forEach((line, index, arr) => {
        if (index === arr.length - 1) return
        const change = [line[0][0] - line[1][0], line[0][1] - line[1][1], line[0][2] - line[1][2]]
        let t = 0.0
        while (t < 1) {
          let points = [...line[0]]
          points = points.map((point, index) => {
            return point - change[index] * t
          })
          points = rotateX(points, this.aX)
          points = rotateY(points, this.aY)
          points = rotateZ(points, this.aZ)
          let [x, y] = projectIsometric(points)
          x = Math.round(x)
          y = Math.round(y)
          if (x >= 0 && y >= 0 && x < 200 && y < 200) {
            buffer.data[(y * 200 + x) * 4] = 20
            buffer.data[(y * 200 + x) * 4 + 1] = 100
            buffer.data[(y * 200 + x) * 4 + 2] = 80
            buffer.data[(y * 200 + x) * 4 + 3] = 255
          }
          t += 0.01
        }
      })

      ctx.putImageData(buffer, 0, 0)
    },
    drawOnePointPerspective () {
      const canvas = this.$refs.canvas
      document.addEventListener('keypress', this.onKey)
      const ctx = canvas.getContext('2d')
      ctx.fillStyle = '#fff'
      ctx.fillRect(0, 0, 200, 200)
      const buffer = ctx.getImageData(0, 0, 200, 200)

      this.cube.forEach(line => {
        const changeAxis = math.add(line[0], line[1].map(el => -el)).findIndex(el => el !== 0)
        const change = line[1][changeAxis] - line[0][changeAxis]
        let t = 0.0
        while (t < 1) {
          let points = [...line[0]]
          points[changeAxis] = line[0][changeAxis] + t * change
          points = rotateX(points, this.aX)
          points = rotateY(points, this.aY)
          points = rotateZ(points, this.aZ)
          const x = Math.trunc(this.dz * (points[0] - 100) / (points[2] + this.dz)) + 100
          const y = Math.trunc(this.dz * (points[1] - 100) / (points[2] + this.dz)) + 100
          if (x >= 0 && y >= 0 && x < 200 && y < 200) {
            buffer.data[(y * 200 + x) * 4] = 20
            buffer.data[(y * 200 + x) * 4 + 1] = 100
            buffer.data[(y * 200 + x) * 4 + 2] = 80
            buffer.data[(y * 200 + x) * 4 + 3] = 255
          }
          t += 0.01
        }
      })

      const pointsCount = this.bezier.length
      let t = 0
      while (t < 1) {
        let x = 0
        let y = 0
        let z = 0
        for (let i = 0; i < pointsCount; i++) {
          x = x + B(i, pointsCount - 1, t) * this.bezier[i][0][0]
          y = y + B(i, pointsCount - 1, t) * this.bezier[i][0][1]
          z = z + B(i, pointsCount - 1, t) * this.bezier[i][0][2]
        }
        x = Math.round(x)
        y = Math.round(y)
        z = Math.round(z)
        let points = rotateX([x, y, z], this.aX)
        points = rotateY(points, this.aY)
        points = rotateZ(points, this.aZ)
        x = Math.trunc(this.dz * (points[0] - 100) / (points[2] + this.dz)) + 100
        y = Math.trunc(this.dz * (points[1] - 100) / (points[2] + this.dz)) + 100
        if (x >= 0 && y >= 0 && x < 200 && y < 200) {
          buffer.data[(y * 200 + x) * 4] = 20
          buffer.data[(y * 200 + x) * 4 + 1] = 100
          buffer.data[(y * 200 + x) * 4 + 2] = 80
          buffer.data[(y * 200 + x) * 4 + 3] = 255
        }
        t += 0.01
      }

      this.bezier.forEach((line, index, arr) => {
        if (index === arr.length - 1) return
        const change = [line[0][0] - line[1][0], line[0][1] - line[1][1], line[0][2] - line[1][2]]
        let t = 0.0
        while (t < 1) {
          let points = [...line[0]]
          points = points.map((point, index) => {
            return point - change[index] * t
          })
          points = rotateX([points[0], points[1], points[2]], this.aX)
          points = rotateY(points, this.aY)
          points = rotateZ(points, this.aZ)
          const x = Math.trunc(this.dz * (points[0] - 100) / (points[2] + this.dz)) + 100
          const y = Math.trunc(this.dz * (points[1] - 100) / (points[2] + this.dz)) + 100
          if (x >= 0 && y >= 0 && x < 200 && y < 200) {
            buffer.data[(y * 200 + x) * 4] = 20
            buffer.data[(y * 200 + x) * 4 + 1] = 100
            buffer.data[(y * 200 + x) * 4 + 2] = 80
            buffer.data[(y * 200 + x) * 4 + 3] = 255
          }
          t += 0.01
        }
      })

      ctx.putImageData(buffer, 0, 0)
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
</style>