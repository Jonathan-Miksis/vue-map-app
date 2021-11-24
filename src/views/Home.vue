<template>
  <div class="home">
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
        places: [
          { lat: 40.6892, lng: -74.0445, description: "Statue of Liberty"},
          { lat: 42.3359079, lng: -71.169176, description: "Doug Flutie Statue"},
          { lat: 38.8895, lng: -77.0353, description: "Washington Monument"},
          { lat: 41.8902, lng: 12.4922, description: "The Colosseum"}
        ]
      };
    },
    created: function () {},
    mounted: function () {
      this.generateMap();
    },
    methods: {
      generateMap: function () {
        mapboxgl.accessToken = process.env.VUE_APP_MAP_KEY;
        const monument = [-77.0353, 38.8895];
        const map = new mapboxgl.Map({
            container: 'map', // container ID
            style: 'mapbox://styles/mapbox/dark-v10', // style URL
            center: monument, // starting position [lng, lat]
            zoom: 9 // starting zoom
        });
        // Create a default Marker and add it to the map.
          for(var i = 0; i < this.places.length; i++) {
          const marker1 = new mapboxgl.Marker()
          .setLngLat(this.places[i])
          .addTo(map);
          }
          // // Create a default Marker, colored black, rotated 45 degrees.
          // const marker2 = new mapboxgl.Marker({ color: 'black', rotation: 45 })
          // .setLngLat([12.65147, 55.608166])
          // .addTo(map);

          // create the popup
          const popup = new mapboxgl.Popup({ offset: 25 }).setText(
          'Construction on the Washington Monument began in 1848.'
          );
          
          // create DOM element for the marker
          const el = document.createElement('div');
          el.id = 'marker';
          
          // create the marker
          new mapboxgl.Marker(el)
          .setLngLat(monument)
          .setPopup(popup) // sets a popup on this marker
          .addTo(map);

          const coordinatesGeocoder = function (query) {
          // Match anything which looks like
          // decimal degrees coordinate pair.
          const matches = query.match(
          /^[ ]*(?:Lat: )?(-?\d+\.?\d*)[, ]+(?:Lng: )?(-?\d+\.?\d*)[ ]*$/i
          );
          if (!matches) {
          return null;
          }
          
          function coordinateFeature(lng, lat) {
          return {
          center: [lng, lat],
          geometry: {
          type: 'Point',
          coordinates: [lng, lat]
          },
          place_name: 'Lat: ' + lat + ' Lng: ' + lng,
          place_type: ['coordinate'],
          properties: {},
          type: 'Feature'
          };
          }
          
          const coord1 = Number(matches[1]);
          const coord2 = Number(matches[2]);
          const geocodes = [];
          
          if (coord1 < -90 || coord1 > 90) {
          // must be lng, lat
          geocodes.push(coordinateFeature(coord1, coord2));
          }
          
          if (coord2 < -90 || coord2 > 90) {
          // must be lat, lng
          geocodes.push(coordinateFeature(coord2, coord1));
          }
          
          if (geocodes.length === 0) {
          // else could be either lng, lat or lat, lng
          geocodes.push(coordinateFeature(coord1, coord2));
          geocodes.push(coordinateFeature(coord2, coord1));
          }
          
          return geocodes;
          };
          
          // Add the control to the map.
          map.addControl(
            new MapboxGeocoder({
            accessToken: mapboxgl.accessToken,
            localGeocoder: coordinatesGeocoder,
            zoom: 4,
            placeholder: 'Try: -40, 170',
            mapboxgl: mapboxgl,
            reverseGeocode: true
            })
            );
        map.on('load', () => {
        // Add a geojson point source.
        // Heatmap layers also work with a vector tile source.
        map.addSource('earthquakes', {
        'type': 'geojson',
        'data': 'https://docs.mapbox.com/mapbox-gl-js/assets/earthquakes.geojson'
        });
        
        map.addLayer(
        {
        'id': 'earthquakes-heat',
        'type': 'heatmap',
        'source': 'earthquakes',
        'maxzoom': 9,
        'paint': {
        // Increase the heatmap weight based on frequency and property magnitude
        'heatmap-weight': [
        'interpolate',
        ['linear'],
        ['get', 'mag'],
        0,
        0,
        6,
        1
        ],
        // Increase the heatmap color weight weight by zoom level
        // heatmap-intensity is a multiplier on top of heatmap-weight
        'heatmap-intensity': [
        'interpolate',
        ['linear'],
        ['zoom'],
        0,
        1,
        9,
        3
        ],
        // Color ramp for heatmap.  Domain is 0 (low) to 1 (high).
        // Begin color ramp at 0-stop with a 0-transparancy color
        // to create a blur-like effect.
        'heatmap-color': [
        'interpolate',
        ['linear'],
        ['heatmap-density'],
        0,
        'rgba(33,102,172,0)',
        0.2,
        'rgb(103,169,207)',
        0.4,
        'rgb(209,229,240)',
        0.6,
        'rgb(253,219,199)',
        0.8,
        'rgb(239,138,98)',
        1,
        'rgb(178,24,43)'
        ],
        // Adjust the heatmap radius by zoom level
        'heatmap-radius': [
        'interpolate',
        ['linear'],
        ['zoom'],
        0,
        2,
        9,
        20
        ],
        // Transition from heatmap to circle layer by zoom level
        'heatmap-opacity': [
        'interpolate',
        ['linear'],
        ['zoom'],
        7,
        1,
        9,
        0
        ]
        }
        },
        'waterway-label'
        );
        
        map.addLayer(
        {
        'id': 'earthquakes-point',
        'type': 'circle',
        'source': 'earthquakes',
        'minzoom': 7,
        'paint': {
        // Size circle radius by earthquake magnitude and zoom level
        'circle-radius': [
        'interpolate',
        ['linear'],
        ['zoom'],
        7,
        ['interpolate', ['linear'], ['get', 'mag'], 1, 1, 6, 4],
        16,
        ['interpolate', ['linear'], ['get', 'mag'], 1, 5, 6, 50]
        ],
        // Color circle by earthquake magnitude
        'circle-color': [
        'interpolate',
        ['linear'],
        ['get', 'mag'],
        1,
        'rgba(33,102,172,0)',
        2,
        'rgb(103,169,207)',
        3,
        'rgb(209,229,240)',
        4,
        'rgb(253,219,199)',
        5,
        'rgb(239,138,98)',
        6,
        'rgb(178,24,43)'
        ],
        'circle-stroke-color': 'white',
        'circle-stroke-width': 1,
        // Transition from heatmap to circle layer by zoom level
        'circle-opacity': [
        'interpolate',
        ['linear'],
        ['zoom'],
        7,
        0,
        8,
        1
        ]
        }
        },
        'waterway-label'
        );
        });
      }
    },
  };
</script>
