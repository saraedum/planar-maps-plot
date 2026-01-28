<template>
  <svg :width="width" :height="height">
      <g v-for="(path, i) of paths" :key="i">
          <path :d="path" :stroke="color(i)" stroke-width="1px" fill="none" />
          <circle :cx="endpoints[i][0][0]" :cy="endpoints[i][0][1]" r="1.5" fill="black" />
          <circle :cx="endpoints[i][1][0]" :cy="endpoints[i][1][1]" r="1.5" fill="black" />
      </g>
  </svg>
</template>
<script>
import { defineComponent } from "vue";
import gsap from "https://cdnjs.cloudflare.com/ajax/libs/gsap/3.2.4/gsap.min.js";

function quadraticBezier(points) {
  points = [...points];
  const P = points.shift()
  const commands = [`M${P[0]} ${P[1]}`]
  if (points.length == 1) {
    const Q = points.shift()
    commands.push(`L${Q[0]} ${Q[1]}`)
  }
  while (points.length) {
    const A = points.shift()
    const B = points.shift()
    if (points.length == 1) {
       const C = points.shift()
       commands.push(`C${A[0]} ${A[1]} ${B[0]} ${B[1]} ${C[0]} ${C[1]}`)
    } else {
       commands.push(`Q${A[0]} ${A[1]} ${B[0]} ${B[1]}`)
    }
  }
  return commands.join(" ");
}

export default defineComponent({
  data() {
    return {
smooth_edges: this.edges,
    }
  },
  computed: {
    rescaled_edges() {
      const width = this.bbox[1][0] - this.bbox[0][0];
      const height = this.bbox[1][1] - this.bbox[0][1];
      return this.smooth_edges.map((edge) => edge.map(([x, y]) => [
        (x - this.bbox[0][0]) / width * this.width,
        (y - this.bbox[0][1]) / width * this.width,
      ]))
    },
    bbox() {
      const x = this.smooth_edges.flat().map(([x, y]) => x);
      const y = this.smooth_edges.flat().map(([x, y]) => y);
      return [[
        Math.min(...x),
        Math.min(...y),
      ], [
        Math.max(...x),
        Math.max(...y),
      ]]
    },
    paths() {
      return this.rescaled_edges.map(quadraticBezier);
    },
    endpoints() {
      return this.rescaled_edges.map((edge) => [
        edge[0],
        edge[edge.length - 1],
      ])
    },
  },
  methods: {
    color(i) {
      return `hsl(${(i * 360 / this.paths.length) | 0}, 70%, 50%)`;
    }
  },
  created() {
    this.$watch(() => this.edges, (current) => {
      this.smooth_edges = this.edges.map((edge, i) => {
        const current_edge = this.smooth_edges[i] || edge
        return edge.map((xy, j) => {
          const current_xy = [...current_edge[j] || xy]
          return current_xy
        })
      })
      this.smooth_edges.map((edge, i) => {
        const new_edge = this.edges[i];
        edge.map((xy, j) => {
          const new_xy = new_edge[j];
          gsap.to(xy, { duration: 1, "0": new_xy[0], "1": new_xy[1] })
        });
      });
    })
  },
  props: ["edges", "width", "height"],
});
</script>
