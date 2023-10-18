<template>
  <div id="sidebar">
    <h1>Area viewer</h1>
    <hr />
    <div>
      Longitude: {{ location.lng.toFixed(3) }}<br />
      Latitude: {{ location.lat.toFixed(3) }} <br />
      Zoom: {{ location.zoom.toFixed(0) }}<br />
    </div>
    <div class="bottom">
      <div>Level:</div>
      <input
        type="number"
        placeholder="level"
        v-model="level"
        @click="$emit('resetPoints')"
      /><br />
      <div>{{ message }}</div>
      <button role="button" @click="getPoints">Get</button>
      <button role="button" @click="postPoints">Send</button>
      <button role="button" @click="resetPoints">Clear</button>
    </div>
  </div>
</template>

<script lang="ts">
export default {
  props: ['location', 'points'],
  emits: ['resetPoints', 'updatePoints'],
  data: () => ({
    message: '',
    level: 0
  }),
  methods: {
    async postPoints() {
      if (this.points.length <= 2) return

      this.message = ''

      let res = await this.fetch(
        'PUT',
        { geolocation: this.points, level: this.level },
        `/${this.level.toString()}`
      )

      this.message = res.status === 404 ? 'Level added!' : 'Level updated!'

      if (res.status === 404) {
        res = await this.fetch(
          'POST',
          { geolocation: this.points, level: this.level },
          `/${this.level.toString()}`
        )
      }

      // this.$emit('resetPoints')
    },
    async getPoints() {
      this.message = ''

      const res = await this.fetch('GET', null, `?filters[level][$eq]=${this.level.toString()}`)
      if (!res.ok) {
        this.message = 'Level not found'

        return
      }
      const json = await res.json()
      if (json.data.length === 0) {
        this.message = 'Level not found'

        return
      }
      this.message = 'Loaded level'

      const points = json.data[0]?.attributes?.geolocation
      this.$emit('updatePoints', points)
    },
    resetPoints() {
      this.message = ''
      this.$emit('resetPoints')
    },
    async fetch(method: string, data?: any, url: string = '') {
      return await fetch(`http://localhost:1337/api/plots${url}`, {
        method,
        headers: {
          'Content-Type': 'application/json'
        },
        body: data ? JSON.stringify({ data: { ...data } }) : null
      })
    }
  }
}
</script>

<style>
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
  background-color: hsl(203, 86%, 80%);
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
  background-color: hsl(203, 58%, 69%);
}
button:active {
  background-color: hsl(203, 43%, 54%);
  box-shadow: none;
}

input {
  margin-top: 10px;
  margin-bottom: 10px;
  width: 70%;
}
</style>
