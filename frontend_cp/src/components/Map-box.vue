<template>
  <div id="map" class="map-container"></div>
</template>

<script lang="ts">
import mapboxgl from 'mapbox-gl'
import type { MapMouseEvent, EventData } from 'mapbox-gl'

mapboxgl.accessToken =
  'pk.eyJ1IjoiYWxtaWdodHliYWthIiwiYSI6ImNsbm9uZHRnODBreWEycW9jdndjaTY3djgifQ.ulHCa8Y6P5nem5H-I2jEqg'

let map: mapboxgl.Map

export default {
  props: ['modelValue'],
  emits: ['update:modelValue', 'click:modelValue'],
  data() {
    return { map: null, points: new Array<{ lng: number; lat: number }>() }
  },
  mounted() {
    const { lng, lat, zoom, bearing, pitch } = this.modelValue

    map = new mapboxgl.Map({
      container: 'map',
      style: 'mapbox://styles/mapbox/dark-v11',
      center: [lng, lat],
      bearing,
      pitch,
      zoom
    })

    const updateLocation = () => this.$emit('update:modelValue', this.getLocation())
    const addPoint = (event: MapMouseEvent & EventData) => this.points.push({ ...event.lngLat })

    map.on('move', updateLocation)
    map.on('zoom', updateLocation)
    map.on('rotate', updateLocation)
    map.on('pitch', updateLocation)
    map.on('click', addPoint)
  },
  unmounted() {
    map.remove()
  },
  methods: {
    getLocation() {
      return {
        ...map.getCenter(),
        bearing: map.getBearing(),
        pitch: map.getPitch(),
        zoom: map.getZoom()
      }
    },
    getPoints() {
      return this.points
    }
  }
}
</script>

<style>
.map-container {
  flex: 1;
}
</style>
