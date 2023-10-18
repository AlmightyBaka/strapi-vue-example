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
    resetPoints() {
      this.sent = false
      this.points = []

      const map = this.$refs.map as any
      map.resetMap()
    }
  }
}
</script>

<style></style>
