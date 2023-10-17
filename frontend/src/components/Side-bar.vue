<template>
  <div id="sidebar">
    <h1>Area viewer</h1>
    <hr />
    <div>Longitude: {{ lng }} | Latitude: {{ lat }} | Zoom: {{ zoom }}</div>
    <div>
      <p>Add area:</p>
      <button role="button">Send</button>
    </div>
  </div>
</template>

<script lang="ts">
export default {
  props: ['location'],
  emits: ['update:location'],
  data() {
    return {
      lng: this.location?.lng,
      lat: this.location?.lat,
      zoom: this.location?.zoom,
      bearing: this.location?.bearing,
      pitch: this.location?.pitch
    }
  }
}
</script>

<style>
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
}

#sidebar > div {
  flex: 1;
  flex-direction: column;
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
  font-weight: 500;
  height: 40px;
  line-height: 20px;
  list-style: none;
  margin: 0;
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
</style>
