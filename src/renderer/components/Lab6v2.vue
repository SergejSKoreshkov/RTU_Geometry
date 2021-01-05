<template>
  <div class="lab6v2">
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
              <button class="btn btn-outline-secondary" type="button" @click="spawnPoints">Spawn</button>
            </div>
            </div>
            <button class="btn btn-primary" @click="points = new SortedSet(Compare); draw()">Clear all</button>
            <button class="btn btn-primary" @click="breakLine(); draw()">Break line</button>
            <button class="btn btn-primary" @click="isDaC = !isDaC; draw()">{{ !isDaC ? 'Switch to Divide and Conquer' : 'Switch to Direct Development'}}</button>
            <hr>
            <ul class="list-group">
              <li v-for="(point, index) in points.arr" :key="point.x.toString() + point.y.toString()" class="list-group-item">
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

class Vector {
  constructor (x, y) {
    this.x = x
    this.y = y
  }

  get Len () {
    return Math.sqrt(this.x * this.x + this.y * this.y)
  }

  static Between (p1, p2) {
    return new Vector(p2.x - p1.x, p2.y - p1.y)
  }
  static Add (a, b) {
    return new Vector(a.x + b.x, a.y + b.y)
  }
  static Sub (a, b) {
    return Vector.Add(a, Vector.NVMul(-1, b))
  }
  static NVMul (num, b) {
    return Vector.VNMul(b, num)
  }
  static VNMul (a, num) {
    return new Vector(num * a.x, num * a.y)
  }
  static Div (a, num) {
    return Vector.VNMul(a, 1 / num)
  }
  static DotProduct (a, b) {
    return a.x * b.x + a.y * b.y
  }
  static CosBetween (a, b) {
    return Vector.DotProduct(a, b) / (a.Len * b.Len)
  }
}

function Compare (p1, p2) {
  let t = Math.trunc(1000 * (p1.x - p2.x))
  if (t !== 0) return t
  return Math.trunc(1000 * (p2.y - p1.y))
}

class SortedSet {
  constructor (compare) {
    this.compare = compare
    this.arr = []
  }

  Add (p1) {
    this.arr.push(p1)
    this.arr = this.arr.sort(this.compare)
  }

  Remove (p1) {
    const index = this.arr.findIndex(p2 =>
      isEqualPoints(p1, p2)
    )
    this.arr.splice(index, 1)
  }

  Contains (p1) {
    return this.arr.some(p2 =>
      isEqualPoints(p1, p2)
    )
  }

  get Min () {
    return this.arr[0]
  }
}

function isEqualPoints (p1, p2) {
  return p1.x === p2.x && p1.y === p2.y
}

const maxEdges = Number.MAX_SAFE_INTEGER

function DrawDiamond (ctx, p1, color = '#000') {
  ctx.beginPath()
  const buffer = ctx.strokeStyle
  ctx.strokeStyle = color
  ctx.arc(p1.x, p1.y, 2, 0, 2 * Math.PI)
  ctx.stroke()
  ctx.strokeStyle = buffer
}

function DrawLine (ctx, p1, p2, color = '#000') {
  ctx.beginPath()
  const buffer = ctx.strokeStyle
  ctx.strokeStyle = color
  ctx.moveTo(p1.x, 300 - p1.y)
  ctx.lineTo(p2.x, 300 - p2.y)
  ctx.stroke()
  ctx.strokeStyle = buffer
}

class TupleSet {
  constructor () {
    this.arr = []
  }

  Add (tuple) {
    this.arr.push(tuple)
  }

  Remove ([pt1, pt2]) {
    const index = this.arr.findIndex(([p1, p2]) =>
      (p1.x === pt1.x && p1.y === pt1.y && p2.x === pt2.x && p2.y === pt2.y) ||
      (p1.x === pt2.x && p1.y === pt2.y && p2.x === pt1.x && p2.y === pt1.y)
    )
    this.arr.splice(index, 1)
  }

  Contains ([pt1, pt2]) {
    return this.arr.some(([p1, p2]) =>
      (p1.x === pt1.x && p1.y === pt1.y && p2.x === pt2.x && p2.y === pt2.y) ||
      (p1.x === pt2.x && p1.y === pt2.y && p2.x === pt1.x && p2.y === pt1.y)
    )
  }

  Count () {
    return this.arr.length
  }
}

async function Triangulate (points, maxEdges, ctx, color, activeEdges = new TupleSet(), draw = true) {
  if (points.arr.length < 2) return
  let triangulation = new TupleSet()

  let _a = points.Min
  let _startVector = new Vector(0, -1)
  let _b = points.arr
    .filter(p1 => !isEqualPoints(p1, _a))
    .map(p1 => {
      return { ...p1, cos: Vector.CosBetween(_startVector, Vector.Between(_a, p1)) }
    })
    .sort((p1, p2) => p2.cos - p1.cos)[0]

  activeEdges.Add([_a, _b])

  while (activeEdges.Count() !== 0 && triangulation.Count() < maxEdges) {
    let minCos = 1.0
    let minEdge = null
    let minPoint = { x: 0, y: 0 }
    let fixedEdges = []

    activeEdges.arr.forEach(edge => {
      let a = edge[0]
      let b = edge[1]
      let dx = b.x - a.x
      let dy = b.y - a.y
      let pointsByAngle = points.arr
        .filter(p1 =>
          !isEqualPoints(p1, a) && !isEqualPoints(p1, b) &&
          (-dy * p1.x + dx * p1.y - dx * a.y + dy * a.x) > 0
        )
        .map(p1 => ({ ...p1,
          cos: Vector.CosBetween(
            Vector.Between(p1, a),
            Vector.Between(p1, b)
          )}))
        .sort((p1, p2) => p1.cos - p2.cos)

      if (pointsByAngle.length === 0) {
        fixedEdges.push(edge)
        return
      }
      let c = pointsByAngle[0]
      let cos = Vector.CosBetween(Vector.Between(c, a), Vector.Between(c, b))
      if (minCos < cos) return

      minCos = cos
      minEdge = edge
      minPoint = c
    })
    fixedEdges.forEach(async edge => {
      triangulation.Add(edge)
      if (draw) DrawLine(ctx, edge[0], edge[1], color)
      if (draw) await new Promise(resolve => setTimeout(resolve, 10))
      activeEdges.Remove(edge)
    })
    if (minEdge === null) break

    activeEdges.Remove(minEdge)
    triangulation.Add(minEdge)
    if (draw) DrawLine(ctx, minEdge[0], minEdge[1], color)
    if (draw) await new Promise(resolve => setTimeout(resolve, 10))
    let e1 = [minEdge[0], minPoint]
    if (!triangulation.Contains(e1)) activeEdges.Add(e1)
    let e2 = [minPoint, minEdge[1]]
    if (!triangulation.Contains(e2)) activeEdges.Add(e2)
  }
  return triangulation
}

export default {
  name: 'lab6',
  data () {
    return {
      x: 0,
      y: 0,
      points: new SortedSet(Compare),
      pointsCount: 8,
      isDaC: false
    }
  },
  mounted () {
    this.draw()
  },
  methods: {
    addPoint () {
      this.points.Add({ x: parseInt(this.x), y: parseInt(this.y) })
      this.draw()
    },
    addPointWithClick (event) {
      const { width, height } = event.target
      const size = event.target.getBoundingClientRect()
      const x = parseInt((event.clientX - size.left) / size.width * width)
      const y = parseInt((event.clientY - size.top) / size.height * height)
      if (!this.points.Contains({ x, y })) this.points.Add({ x, y })
      this.draw()
    },
    removePoint (index) {
      this.points.arr.splice(index, 1)
    },
    breakLine () {
      this.points.arr[this.points.length - 1].break = true
    },
    async draw () {
      this.$refs.canvas.width = 300
      this.$refs.canvas.height = 300
      const ctx = this.$refs.canvas.getContext('2d')
      ctx.fillStyle = '#fff'
      ctx.fillRect(0, 0, 300, 300)
      if (!this.isDaC) {
        const pointSet = new SortedSet(Compare)
        this.points.arr.forEach(p1 => {
          pointSet.Add({ x: p1.x, y: 300 - p1.y })
          DrawDiamond(ctx, p1)
        })
        await Triangulate(pointSet, maxEdges, ctx, '#000')
      } else {
        const _points = this.points.arr
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

        const finalTr = await Promise.all(final.map(async ps => {
          const pointSet = new SortedSet(Compare)
          const color = `#${Math.trunc(Math.random() * 16).toString(16)}${Math.trunc(Math.random() * 9).toString(16)}${Math.trunc(Math.random() * 9).toString(16)}`
          ps.forEach(p1 => {
            pointSet.Add({ x: p1.x, y: 300 - p1.y })
            DrawDiamond(ctx, p1, color)
          })
          const triangulation = await Triangulate(pointSet, maxEdges, ctx, color)
          return [triangulation, color]
        }))
        await new Promise(resolve => setTimeout(resolve, 500))
        ctx.fillRect(0, 0, 300, 300)
        const triangulation = new TupleSet()
        finalTr.forEach(([t, c]) => {
          if (!t) return
          t.arr.forEach(edge => {
            if (!triangulation.Contains(edge)) triangulation.Add(edge)
          })
        })

        const pointSet = new SortedSet(Compare)
        final.map(ps => {
          ps.forEach(p1 => {
            pointSet.Add({ x: p1.x, y: 300 - p1.y })
          })
        })
        const tr = await Triangulate(pointSet, maxEdges, ctx, '#000', new TupleSet(), false)
        const oldDiff = []
        finalTr.forEach(([t, c]) => {
          if (!t) return
          t.arr.forEach(edge => {
            if (tr.Contains(edge)) oldDiff.push([edge, c])
          })
        })
        tr.arr.forEach(edge => DrawLine(ctx, edge[0], edge[1]))
        oldDiff.forEach(([edge, c]) => DrawLine(ctx, edge[0], edge[1], c))
      }
    },
    spawnPoints () {
      const points = new Array(parseInt(this.pointsCount)).fill(0).map(el => ({ x: Math.trunc(Math.random() * 300), y: Math.trunc(Math.random() * 300) }))
      this.points.arr = []
      points.forEach(p => this.points.Add(p))
      this.draw()
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