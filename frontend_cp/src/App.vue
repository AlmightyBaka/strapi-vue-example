<template>
  <div id="layout">
    <div id="map" class="map-container"></div>
    <div id="sidebar">
      <h1>Area viewer</h1>
      <hr />
      <div>
        Longitude: {{ location.lng.toFixed(3) }}<br />
        Latitude: {{ location.lat.toFixed(3) }} <br />
        Zoom: {{ location.zoom.toFixed(0) }}<br />
        <!-- {{ points }} -->
      </div>
      <div class="bottom">
        <div>Level:</div>
        <input type="number" placeholder="level" v-model="level" @click="resetMap" /><br />
        <div v-if="sent">Points sent!</div>
        <button role="button" @click="getPoints">Get</button>
        <button role="button" @click="postPoints">Send</button>
        <button role="button" @click="resetPoints">Reset</button>
      </div>
    </div>
  </div>
</template>

<script lang="ts">
import '../node_modules/mapbox-gl/dist/mapbox-gl.css'
import mapboxgl from 'mapbox-gl'
import type { MapMouseEvent, EventData } from 'mapbox-gl'

mapboxgl.accessToken =
  'pk.eyJ1IjoiYWxtaWdodHliYWthIiwiYSI6ImNsbm9uZHRnODBreWEycW9jdndjaTY3djgifQ.ulHCa8Y6P5nem5H-I2jEqg'

let map: mapboxgl.Map

export default {
  props: ['modelValue'],
  emits: ['update:modelValue', 'click:modelValue'],
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
  mounted() {
    const { lng, lat, zoom, bearing, pitch } = this.location

    map = new mapboxgl.Map({
      container: 'map',
      style: 'mapbox://styles/mapbox/dark-v11',
      center: [lng, lat],
      bearing,
      pitch,
      zoom
    })

    const updateLocation = () => {
      this.location = this.getLocation()
    }
    const addPoint = (event: MapMouseEvent & EventData) => {
      this.points.push({ ...event.lngLat })
      if (this.points.length > 2) {
        this.drawPoints(this.points)
      }
    }

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
    addPoint(point: { lng: number; lat: number }) {
      this.points.push(point)
    },
    getLocation() {
      return {
        ...map.getCenter(),
        bearing: map.getBearing(),
        pitch: map.getPitch(),
        zoom: map.getZoom()
      }
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
      console.log(res)
      console.log(`http://localhost:1337/api/plots?filters[level][$eq]=${this.level.toString()}`)
      if (!res.ok) return
      const json = await res.json()
      console.log(json.data)
      if (json.data.length === 0) return

      console.log(json.data[0]?.attributes?.geolocation)
      this.points = json.data[0]?.attributes?.geolocation
      this.drawPoints(this.points)
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
      if (map.getSource('layer') !== undefined) {
        map.removeLayer('fill')
        map.removeLayer('outline')
        map.removeSource('layer')
      }
    },
    drawPoints(points: Array<{ lng: number; lat: number }>) {
      if (!points || points.length === 0) return
      const position = [[...points.map((point) => [point.lng, point.lat])]]

      this.resetMap()

      map.addSource('layer', {
        type: 'geojson',
        data: {
          type: 'Feature',
          geometry: {
            type: 'Polygon',
            coordinates: position
          },

          properties: {}
        }
      })
      map.addLayer({
        id: 'fill',
        type: 'fill',
        source: 'layer',
        layout: {},
        paint: {
          'fill-color': '#0080ff',
          'fill-opacity': 0.5
        }
      })
      map.addLayer({
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

#sidebar {
  display: flex;
  flex-direction: column;
  background-color: hsl(0, 2%, 70%);
  color: #2f243a;
  padding: 20px 12px;
  font-family: monospace;
  z-index: 1;
  position: absolute;
  height: 90%;
  width: 30%;
  min-width: 200px;
  margin: 2%;
  border-radius: 6px;
  overflow-y: auto;
}

#sidebar > div {
  flex: 1;
  flex-direction: column;
}

#sidebar > div:last-of-type {
  flex: 0;
  margin-top: auto;
}

h1 {
  text-align: center;
}

hr {
  margin-top: 10px;
  margin-bottom: 10px;
}

button {
  background-color: #db8a74;
  border-radius: 8px;
  border-style: none;
  box-shadow: rgba(255, 255, 255, 0.4) 0 1px 0 0 inset;
  box-sizing: border-box;
  color: #444054;
  cursor: pointer;
  display: inline-block;
  font-family: 'Haas Grot Text R Web', 'Helvetica Neue', Helvetica, Arial, sans-serif;
  font-size: 14px;
  font-weight: 600;
  height: 40px;
  line-height: 20px;
  list-style: none;
  margin: 10px;
  outline: none;
  padding: 10px 16px;
  position: relative;
  text-align: center;
  text-decoration: none;
  transition: color 100ms;
  vertical-align: baseline;
  user-select: none;
  -webkit-user-select: none;
  touch-action: manipulation;
}
button:hover {
  background-color: #c47c68;
}
button:active {
  background-color: #ac6653;
  box-shadow: none;
}

input {
  margin-top: 10px;
  margin-bottom: 10px;
  width: 70%;
}
</style>
