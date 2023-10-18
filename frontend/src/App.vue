<template>
  <Mapbox
    ref="map"
    @move="updateLocation"
    @add-point="addPoint"
    :location="location"
    :points="points"
  />
  <Sidebar
    ref="sidebar"
    @reset-points="resetPoints"
    @update-points="updatePoints"
    :location="location"
    :points="points"
  />
</template>

<script lang="ts">
import Mapbox from './components/Map-box.vue'
import Sidebar from './components/Side-bar.vue'

export default {
  components: {
    Mapbox,
    Sidebar
  },
  data: () => ({
    location: {
      lng: 37.6139,
      lat: 55.7402,
      bearing: 0,
      pitch: 0,
      zoom: 11
    },
    points: new Array<{ lng: number; lat: number }>(),
    sent: false,
    level: 0
  }),
  methods: {
    updateLocation(newLocation: any) {
      this.location = { ...this.location, ...newLocation }
    },
    addPoint(point: { lng: number; lat: number }) {
      this.points.push(point)

      const map = this.$refs.map as any
      map.drawPoints()
    },
    updatePoints(points: Array<{ lng: number; lat: number }>) {
      this.points = points

      const map = this.$refs.map as any
      map.drawPoints()
    },
    async postPoints() {
      if (this.points.length <= 2) return

      this.sent = true
      this.resetMap()

      let res = await this.fetch(
        'PUT',
        { geolocation: this.points, level: this.level },
        `/${this.level.toString()}`
      )
      if (res.status === 404) {
        res = await this.fetch('POST', { geolocation: this.points, level: this.level })
      }

      this.points = []
    },
    async getPoints() {
      this.sent = false
      this.resetMap()

      const res = await this.fetch('GET', null, `?filters[level][$eq]=${this.level.toString()}`)
      if (!res.ok) return
      const json = await res.json()
      if (json.data.length === 0) return

      this.points = json.data[0]?.attributes?.geolocation
      // this.drawPoints(this.points)
    },
    resetPoints() {
      this.sent = false
      this.points = []
      this.resetMap()
    },
    async fetch(method: string, data?: any, url: string = '') {
      return await fetch(`http://localhost:1337/api/plots${url}`, {
        method,
        headers: {
          'Content-Type': 'application/json'
        },
        body: data ? JSON.stringify({ data: { ...data } }) : null
      })
    },
    resetMap() {
      const map = this.$refs.map as any
      map.resetMap()

      this.points = []
    }
  }
}
</script>

<style></style>
