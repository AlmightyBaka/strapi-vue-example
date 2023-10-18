<template>
  <div id="layout">
    <div ref="map" class="map-container"></div>
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
import type { MapMouseEvent, EventData, Map } from 'mapbox-gl'

export default {
  data: () => ({
    map: {} as Map,
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
    // public token
    mapboxgl.accessToken =
      'pk.eyJ1IjoiYWxtaWdodHliYWthIiwiYSI6ImNsbm9uZHRnODBreWEycW9jdndjaTY3djgifQ.ulHCa8Y6P5nem5H-I2jEqg'

    // map setup
    const { lng, lat, zoom, bearing, pitch } = this.location
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
      this.location = {
        ...this.map.getCenter(),
        bearing: this.map.getBearing(),
        pitch: this.map.getPitch(),
        zoom: this.map.getZoom()
      }
    }
    const addPoint = (event: MapMouseEvent & EventData) => {
      this.points.push({ ...event.lngLat })
      if (this.points.length > 2) {
        this.drawPoints(this.points)
      }
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
      if (this.map.getSource('layer') === undefined) return

      this.map.removeLayer('fill')
      this.map.removeLayer('outline')
      this.map.removeSource('layer')
    },
    drawPoints(points: Array<{ lng: number; lat: number }>) {
      if (!points || points.length === 0) return

      this.resetMap()
      const coordinates = [[...points.map((point) => [point.lng, point.lat])]]

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

#sidebar {
  display: flex;
  flex-direction: column;
  background-color: hsl(0, 0%, 93%);
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
