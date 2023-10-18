<template>
  <div id="layout">
    <div ref="map" class="map-container"></div>
  </div>
</template>

<script lang="ts">
import '../../node_modules/mapbox-gl/dist/mapbox-gl.css'
import mapboxgl from 'mapbox-gl'
import type { MapMouseEvent, EventData, Map } from 'mapbox-gl'
import { nextTick } from 'vue'

export default {
  props: ['location', 'points'],
  emits: ['move', 'addPoint'],
  data: () => ({
    map: {} as Map
  }),
  mounted() {
    // public token
    mapboxgl.accessToken =
      'pk.eyJ1IjoiYWxtaWdodHliYWthIiwiYSI6ImNsbm9uZHRnODBreWEycW9jdndjaTY3djgifQ.ulHCa8Y6P5nem5H-I2jEqg'

    // map setup
    const { lng, lat, zoom, bearing, pitch } = this.$props.location
    this.map = new mapboxgl.Map({
      container: this.$refs.map as HTMLElement,
      style: 'mapbox://styles/mapbox/dark-v11',
      center: [lng, lat],
      bearing,
      pitch,
      zoom
    })

    // map events setup
    const updateLocation = () => {
      this.$emit('move', {
        ...this.map.getCenter(),
        bearing: this.map.getBearing(),
        pitch: this.map.getPitch(),
        zoom: this.map.getZoom()
      })
    }
    const addPoint = (event: MapMouseEvent & EventData) => {
      this.$emit('addPoint', { ...event.lngLat })
    }

    this.map.on('move', updateLocation)
    this.map.on('zoom', updateLocation)
    this.map.on('rotate', updateLocation)
    this.map.on('pitch', updateLocation)
    this.map.on('click', addPoint)
  },
  unmounted() {
    this.map.remove()
  },
  methods: {
    resetMap() {
      if (this.map.getSource('layer') === undefined) return

      this.map.removeLayer('fill')
      this.map.removeLayer('outline')
      this.map.removeSource('layer')
    },
    async drawPoints() {
      await nextTick()
      if (!this.$props.points || this.$props.points.length === 0) return

      this.resetMap()
      const coordinates = [
        [...this.$props.points.map((point: { lng: number; lat: number }) => [point.lng, point.lat])]
      ]

      this.map.addSource('layer', {
        type: 'geojson',
        data: {
          type: 'Feature',
          geometry: {
            type: 'Polygon',
            coordinates
          },
          properties: {}
        }
      })
      this.map.addLayer({
        id: 'fill',
        type: 'fill',
        source: 'layer',
        layout: {},
        paint: {
          'fill-color': '#0080ff',
          'fill-opacity': 0.5
        }
      })
      this.map.addLayer({
        id: 'outline',
        type: 'line',
        source: 'layer',
        layout: {},
        paint: {
          'line-color': '#000',
          'line-width': 4
        }
      })
    }
  }
}
</script>

<style>
#layout {
  flex: 1;
  display: flex;
  position: relative;
}

.map-container {
  flex: 1;
}
</style>
