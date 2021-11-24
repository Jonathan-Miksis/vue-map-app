<template>
  <div class="home">
    <h1>{{ message }}</h1>
    <div id='map' style='width: 700px; height: 600px;'></div>
  </div>
</template>

<style>
#marker {
background-image: url('https://docs.mapbox.com/mapbox-gl-js/assets/washington-monument.jpg');
background-size: cover;
width: 50px;
height: 50px;
border-radius: 50%;
cursor: pointer;
}
 
.mapboxgl-popup {
max-width: 200px;
}
</style>

<script>
import mapboxgl from 'mapbox-gl'; // or "const mapboxgl = require('mapbox-gl');"


  export default {
    data: function () {
      return {
        message: "Welcome to Vue.js!",
      };
    },
    created: function () {},
    mounted: function () {
      this.generateMap();
    },
    methods: {
      generateMap: function () {
        mapboxgl.accessToken = process.env.VUE_APP_MAP_KEY;
        const monument = [-71.0513, 42.3522]
        const map = new mapboxgl.Map({
            container: 'map', // container ID
            style: 'mapbox://styles/mapbox/streets-v11', // style URL
            center: monument, // starting position [lng, lat]
            zoom: 9 // starting zoom
        });
        // Create a default Marker and add it to the map.
          const marker1 = new mapboxgl.Marker()
          .setLngLat([12.554729, 55.70651])
          .addTo(map);
          
          // Create a default Marker, colored black, rotated 45 degrees.
          const marker2 = new mapboxgl.Marker({ color: 'black', rotation: 45 })
          .setLngLat([12.65147, 55.608166])
          .addTo(map);
          // create the popup
          const popup = new mapboxgl.Popup({ offset: 25 }).setText(
          'Boston Tea Party Ship'
          );
          
          // create DOM element for the marker
          const el = document.createElement('div');
          el.id = 'marker';
          
          // create the marker
          new mapboxgl.Marker(el)
          .setLngLat(monument)
          .setPopup(popup) // sets a popup on this marker
          .addTo(map);
      }
    },
  };
</script>
