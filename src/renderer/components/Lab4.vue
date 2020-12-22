<template>
  <div class="lab4">
    <h4>Internal space x, y, z is in interval [-100;100]</h4>
    <h5>[-1;1] mapped to [-100;100]</h5>
    <h6 style="color: blue">Use 'q' and 'e' keys to change dZ. dZ: {{ dz }}</h6>
    <h6 style="color: blue">Use 'w' and 's' keys to change aX. aX: {{ aX }}</h6>
    <h6 style="color: blue">Use 'a' and 'd' keys to change aY. aY: {{ aY }}</h6>
    <h6 style="color: blue">Use 'z' and 'c' keys to change aZ. aZ: {{ aZ }}</h6>
    <div class="row">
      <div class="col-6">
        <div class="form-check">
          <input class="form-check-input" type="radio" name="type" value="isometric" v-model="projectionType" @change="this.draw">
          <label class="form-check-label" for="type">
            Isometric projection
          </label>
        </div>
        <div class="form-check">
          <input class="form-check-input" type="radio" name="type" value="onepoint" v-model="projectionType" @change="this.draw">
          <label class="form-check-label" for="type">
            One point perspective
          </label>
        </div>
        <hr>
        <div class="form-check">
          <input class="form-check-input" type="radio" name="object" value="bezierSurface" v-model="objectType" @change="this.draw">
          <label class="form-check-label" for="cubeSelect">
            Bezier surface
          </label>
        </div>
        <div class="form-check">
          <input class="form-check-input" type="radio" name="object" value="bicubicCoons" v-model="objectType" @change="this.draw">
          <label class="form-check-label" for="cubeSelect">
            Bicubic Coons
          </label>
        </div>
        <hr>
        <div v-if="objectType === 'bezierSurface'">
          <div v-for="(lines, index) in bezier" :key="`${index}`">
            <h4>Bezier surface {{index}}</h4>
            <ul class="list-group">
              <li class="list-group-item" v-for="(points, indexC) in lines" :key="`${index}${indexC}`">
                <h5>Bezier curve {{indexC}}</h5>
                <div class="input-group mb-3" v-for="(point, indexP) in points" :key="`${index}${indexC}${indexP}`">
                  <div class="input-group-prepend">
                    <span class="input-group-text" id="basic-addon1">X:</span>
                  </div>
                  <input type="number" class="form-control" v-model="bezier[index][indexC][indexP][0]">
                  <div class="input-group-prepend">
                    <span class="input-group-text" id="basic-addon1">Y:</span>
                  </div>
                  <input type="number" class="form-control" v-model="bezier[index][indexC][indexP][1]">
                  <div class="input-group-prepend">
                    <span class="input-group-text" id="basic-addon1">Z:</span>
                  </div>
                  <input type="number" class="form-control" v-model="bezier[index][indexC][indexP][2]">
                  <div class="input-group-append">
                    <button class="btn btn-outline-secondary" type="button" @click="bezier[index][indexC].splice(indexP, 1)">Delete</button>
                  </div>
                </div>
                <button class="btn btn-primary" @click="bezier[index][indexC].push([0, 0, 0])">Add point</button>
                <button class="btn btn-danger" @click="bezier[index].splice(indexC, 1)">Delete curve</button>
              </li>
              <button class="btn btn-primary" @click="bezier[index].push([[0, 0, 0], [0, 0, 0], [0, 0, 0]])">Add Curve</button>
              <button class="btn btn-danger" @click="bezier.splice(index, 1)">Delete Surface</button>
            </ul>
            <hr>
          </div>
          <button class="btn btn-warning" style="width: 100%; margin-bottom: 1em" @click="bezier.push([])">Add Surface</button>
        </div>
        

        <div v-if="objectType === 'bicubicCoons'">
          <div v-for="(sets, index) in bicubic" :key="`${index}`">
            <h4>Bicubic Coons surface {{index}}</h4>
            <ul class="list-group">
              <li class="list-group-item" v-for="(coordinates, indexC) in sets" :key="`${index}${indexC}`">
                <h5>Point {{indexC}}</h5>
                <div class="input-group mb-3">
                  <div class="input-group-prepend">
                    <span class="input-group-text" id="basic-addon1">X:</span>
                  </div>
                  <input type="number" class="form-control" v-model="bicubic[index][indexC][0]">
                  <div class="input-group-prepend">
                    <span class="input-group-text" id="basic-addon1">Y:</span>
                  </div>
                  <input type="number" class="form-control" v-model="bicubic[index][indexC][1]">
                  <div class="input-group-prepend">
                    <span class="input-group-text" id="basic-addon1">Z:</span>
                  </div>
                  <input type="number" class="form-control" v-model="bicubic[index][indexC][2]">
                  <div class="input-group-append">
                  </div>
                </div>
                <div class="input-group mb-3">
                  <div class="input-group-prepend">
                    <span class="input-group-text" id="basic-addon1">Vx:</span>
                  </div>
                  <input type="number" class="form-control" v-model="bicubic[index][indexC][3]">
                  <div class="input-group-prepend">
                    <span class="input-group-text" id="basic-addon1">Vy:</span>
                  </div>
                  <input type="number" class="form-control" v-model="bicubic[index][indexC][4]">
                  <div class="input-group-prepend">
                    <span class="input-group-text" id="basic-addon1">Vz:</span>
                  </div>
                  <input type="number" class="form-control" v-model="bicubic[index][indexC][5]">
                  <div class="input-group-append">
                  </div>
                </div>
                <div class="input-group mb-3">
                  <div class="input-group-prepend">
                    <span class="input-group-text" id="basic-addon1">Ux:</span>
                  </div>
                  <input type="number" class="form-control" v-model="bicubic[index][indexC][6]">
                  <div class="input-group-prepend">
                    <span class="input-group-text" id="basic-addon1">Uy:</span>
                  </div>
                  <input type="number" class="form-control" v-model="bicubic[index][indexC][7]">
                  <div class="input-group-prepend">
                    <span class="input-group-text" id="basic-addon1">Uz:</span>
                  </div>
                  <input type="number" class="form-control" v-model="bicubic[index][indexC][8]">
                  <div class="input-group-append">
                  </div>
                </div>
                <div class="input-group mb-3">
                  <div class="input-group-prepend">
                    <span class="input-group-text" id="basic-addon1">VUx:</span>
                  </div>
                  <input type="number" class="form-control" v-model="bicubic[index][indexC][9]">
                  <div class="input-group-prepend">
                    <span class="input-group-text" id="basic-addon1">VUy:</span>
                  </div>
                  <input type="number" class="form-control" v-model="bicubic[index][indexC][10]">
                  <div class="input-group-prepend">
                    <span class="input-group-text" id="basic-addon1">VUz:</span>
                  </div>
                  <input type="number" class="form-control" v-model="bicubic[index][indexC][11]">
                  <div class="input-group-append">
                  </div>
                </div>
              </li>
              <button class="btn btn-danger" @click="bicubic.splice(index, 1)">Delete Surface</button>
            </ul>
            <hr>
          </div>
          <button class="btn btn-warning" style="width: 100%; margin-bottom: 1em" @click="bicubic.push([new Array(12).fill(0), new Array(12).fill(0), new Array(12).fill(0), new Array(12).fill(0)])">Add Surface</button>
        </div>
      </div>
      <div class="col-6">
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

const Nc = math.matrix([
  [2, -2, 1, 1],
  [-3, 3, -2, -1],
  [0, 0, 1, 0],
  [1, 0, 0, 0]
])
const NcT = math.matrix([
  [2, -3, 0, 1],
  [-2, 3, 0, 0],
  [1, -2, 1, 0],
  [1, -1, 0, 0]
])

export default {
  name: 'lab4',
  data () {
    return {
      dz: 200,
      aX: 0,
      aY: 0,
      aZ: 0,
      scale: 1,
      projectionType: 'isometric',
      objectType: 'bezierSurface',
      bicubic: [
        [
          [50, 50, 50, 80, 50, 50, 50, 80, 50, 50, 80, 80],
          [150, 50, 50, 180, 50, 50, 150, 50, 30, 150, 80, 50],
          [50, 150, 50, 80, 150, 50, 50, 180, 50, 50, 150, 80],
          [50, 150, 150, 50, 160, 150, 60, 150, 150, 50, 150, 180]
        ], [
          [50, 150, 50, 80, 150, 50, 50, 180, 50, 50, 150, 80],
          [50, 150, 150, 50, 160, 150, 60, 150, 150, 50, 150, 180],
          [180, 180, 180, 190, 180, 180, 180, 190, 180, 180, 180, 190],
          [130, 130, 130, 130, 140, 130, 140, 130, 130, 130, 130, 140]
        ]
      ],
      bezier: [
        [
          [[20, 20, 20], [20, 40, 20], [20, 20, 40]],
          [[50, 50, 50], [50, 70, 50], [50, 50, 70]],
          [[90, 90, 90], [90, 110, 90], [90, 90, 110]]
        ], [
          [[90, 90, 90], [90, 110, 90], [90, 90, 110]],
          [[130, 130, 130], [130, 150, 130], [130, 130, 150]],
          [[100, 150, 100], [160, 150, 160], [130, 130, 180]]
        ]
      ]
    }
  },
  mounted () {
    this.cube = this.cube1
    this.draw()
  },
  watch: {
    bicubic () {
      this.draw()
    },
    bezier () {
      this.draw()
    }
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
      switch (this.projectionType) {
        case 'isometric': this.drawcProjection(this.projectIsometric); return
        case 'onepoint': this.drawcProjection(this.projectionOnePoint)
      }
    },
    drawcProjection (projectionFn) {
      const canvas = this.$refs.canvas
      document.addEventListener('keypress', this.onKey)
      const ctx = canvas.getContext('2d')
      ctx.fillStyle = '#fff'
      ctx.fillRect(0, 0, 200, 200)
      const buffer = ctx.getImageData(0, 0, 200, 200)

      if (this.objectType === 'bezierSurface') {
        this.bezier.forEach(bezier => {
          // Bezier surface
          const linesCount = bezier.length
          for (let u = 0; u < 1; u += 0.05) {
            for (let v = 0; v < 1; v += 0.05) {
              let x = 0
              let y = 0
              let z = 0
              for (let i = 0; i < linesCount; i++) {
                const pointsCount = bezier[i].length
                for (let j = 0; j < pointsCount; j++) {
                  const Bni = B(i, linesCount - 1, u)
                  const Bmj = B(j, pointsCount - 1, v)
                  x = x + Bni * Bmj * bezier[i][j][0]
                  y = y + Bni * Bmj * bezier[i][j][1]
                  z = z + Bni * Bmj * bezier[i][j][2]
                }
              }
              x = Math.round(x)
              y = Math.round(y)
              z = Math.round(z)
              let points = rotateX([x, y, z], this.aX)
              points = rotateY(points, this.aY)
              points = rotateZ(points, this.aZ)
              points = projectionFn(points)
              x = Math.round(points[0])
              y = Math.round(points[1])
              if (x >= 0 && y >= 0 && x < 200 && y < 200) {
                buffer.data[(y * 200 + x) * 4] = 20
                buffer.data[(y * 200 + x) * 4 + 1] = 100
                buffer.data[(y * 200 + x) * 4 + 2] = 80
                buffer.data[(y * 200 + x) * 4 + 3] = 255
              }
            }
          }

          bezier.forEach(line => {
            // Bezier line
            const pointsCount = line.length
            let t = 0
            while (t < 1) {
              let x = 0
              let y = 0
              let z = 0
              for (let i = 0; i < pointsCount; i++) {
                x = x + B(i, pointsCount - 1, t) * line[i][0]
                y = y + B(i, pointsCount - 1, t) * line[i][1]
                z = z + B(i, pointsCount - 1, t) * line[i][2]
              }
              let points = rotateX([x, y, z], this.aX)
              points = rotateY(points, this.aY)
              points = rotateZ(points, this.aZ)
              points = projectionFn(points)
              x = Math.trunc(points[0])
              y = Math.trunc(points[1])
              if (x >= 0 && y >= 0 && x < 200 && y < 200) {
                buffer.data[(y * 200 + x) * 4] = 255
                buffer.data[(y * 200 + x) * 4 + 1] = 0
                buffer.data[(y * 200 + x) * 4 + 2] = 100
                buffer.data[(y * 200 + x) * 4 + 3] = 255
              }
              t += 0.01
            }
            // Bezier points
            line.forEach(coordinates => {
              let points = rotateX(coordinates, this.aX)
              points = rotateY(points, this.aY)
              points = rotateZ(points, this.aZ)
              points = projectionFn(points)
              const x = Math.trunc(points[0])
              const y = Math.trunc(points[1])
              if (!(x >= 0 && y >= 0 && x < 200 && y < 200)) return
              buffer.data[(y * 200 + x) * 4] = 255
              buffer.data[(y * 200 + x) * 4 + 1] = 20
              buffer.data[(y * 200 + x) * 4 + 2] = 20
              buffer.data[(y * 200 + x) * 4 + 3] = 255
            })
          })
        })
      } else if (this.objectType === 'bicubicCoons') {
        this.bicubic.forEach(p => {
          const PcX = math.matrix([
            [p[0][0], p[1][0], p[0][3], p[1][3]],
            [p[2][0], p[3][0], p[2][3], p[3][3]],
            [p[0][6], p[1][6], p[0][9], p[1][9]],
            [p[2][6], p[3][6], p[2][9], p[3][9]]
          ])
          const PcY = math.matrix([
            [p[0][1], p[1][1], p[0][4], p[1][4]],
            [p[2][1], p[3][1], p[2][4], p[3][4]],
            [p[0][7], p[1][7], p[0][10], p[1][10]],
            [p[2][7], p[3][7], p[2][10], p[3][10]]
          ])
          const PcZ = math.matrix([
            [p[0][2], p[1][2], p[0][5], p[1][5]],
            [p[2][2], p[3][2], p[2][5], p[3][5]],
            [p[0][8], p[1][8], p[0][11], p[1][11]],
            [p[2][8], p[3][8], p[2][11], p[3][11]]
          ])
          for (let u = 0; u < 1; u += 0.05) {
            for (let v = 0; v < 1; v += 0.05) {
              const U = math.matrix([Math.pow(u, 3), Math.pow(u, 2), u, 1])
              const V = math.matrix([
                [Math.pow(v, 3)],
                [Math.pow(v, 2)],
                [v],
                [1]
              ])
              const UNc = math.multiply(U, Nc)
              const NcTV = math.multiply(NcT, V)

              let resultX = math.multiply(UNc, PcX)
              resultX = math.multiply(resultX, NcTV)

              let resultY = math.multiply(UNc, PcY)
              resultY = math.multiply(resultY, NcTV)

              let resultZ = math.multiply(UNc, PcZ)
              resultZ = math.multiply(resultZ, NcTV)

              let x = Math.trunc(resultX._data[0])
              let y = Math.trunc(resultY._data[0])
              const z = Math.trunc(resultZ._data[0])

              let points = rotateX([x, y, z], this.aX)
              points = rotateY(points, this.aY)
              points = rotateZ(points, this.aZ)
              points = projectionFn(points)
              x = Math.trunc(points[0])
              y = Math.trunc(points[1])

              if (!(x >= 0 && y >= 0 && x < 200 && y < 200)) continue
              buffer.data[(y * 200 + x) * 4] = 20
              buffer.data[(y * 200 + x) * 4 + 1] = 100
              buffer.data[(y * 200 + x) * 4 + 2] = 80
              buffer.data[(y * 200 + x) * 4 + 3] = 255
            }
          }

          p.forEach(coordinates => {
            let points = rotateX(coordinates, this.aX)
            points = rotateY(points, this.aY)
            points = rotateZ(points, this.aZ)
            points = projectionFn(points)
            const x = Math.trunc(points[0])
            const y = Math.trunc(points[1])
            if (!(x >= 0 && y >= 0 && x < 200 && y < 200)) return
            buffer.data[(y * 200 + x) * 4] = 255
            buffer.data[(y * 200 + x) * 4 + 1] = 20
            buffer.data[(y * 200 + x) * 4 + 2] = 20
            buffer.data[(y * 200 + x) * 4 + 3] = 255
          })

          p.forEach(c => {
            const vectorLines = [
              [c[0], c[1], c[2], c[3], c[4], c[5]],
              [c[0], c[1], c[2], c[6], c[7], c[8]],
              [c[0], c[1], c[2], c[9], c[10], c[11]]
            ]
            vectorLines.forEach(line => {
              const change = [line[0] - line[3], line[1] - line[4], line[2] - line[5]]
              let t = 0.0
              while (t < 1) {
                let points = [line[0], line[1], line[2]]
                points = points.map((point, index) => {
                  return point - change[index] * t
                })
                points = rotateX([...points], this.aX)
                points = rotateY(points, this.aY)
                points = rotateZ(points, this.aZ)
                points = projectionFn(points)
                const x = Math.round(points[0])
                const y = Math.round(points[1])
                if (x >= 0 && y >= 0 && x < 200 && y < 200) {
                  buffer.data[(y * 200 + x) * 4] = 255
                  buffer.data[(y * 200 + x) * 4 + 1] = 20
                  buffer.data[(y * 200 + x) * 4 + 2] = 20
                  buffer.data[(y * 200 + x) * 4 + 3] = 255
                }
                t += 0.05
              }
            })
          })
        })
        // U Nc Pc NcT V
        // U =  [u3 u2 u 1]
        // V = [v3 v2 v 1]T
        // u.v 0 1
        // Nc
        // 2 -2 1 1
        // -3 3 -2 -1
        // 0 0 1 0
        // 1 0 0 0
        // Pc
        // P00 P01 Pv00 Pv01
        // P10 P11 Pv10 Pv11
        // Pu00 pU01 Puv00 Puv01
        // Pu10 Pu11 Puv10 Puv11
      }

      ctx.putImageData(buffer, 0, 0)
    },
    projectIsometric (points) {
      points = points.map(el => el - 100)
      const pointMatrix = math.matrix([
        [points[0]],
        [points[1]],
        [points[2]]
      ])
      let result = math.multiply(isometricMatrix, pointMatrix)
      result = math.multiply(math.matrix([
        [1, 0, 0],
        [0, 1, 0],
        [0, 0, 0]
      ]), result)
      return result._data.map(el => el[0] + 100)
    },
    projectionOnePoint (points) {
      const x = Math.trunc(this.dz * (points[0] - 100) / (points[2] + this.dz)) + 100
      const y = Math.trunc(this.dz * (points[1] - 100) / (points[2] + this.dz)) + 100
      return [x, y]
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