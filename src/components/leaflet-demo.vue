<template>
  <div class="container">
    <div id="map"></div>
  </div>
</template>
<script>
  // import leaflet from 'leaflet';

  console.log(123);

  export default {
    name: 'leaflet-demo',
    data() {
      return {};
    },
    mounted() {
      console.log(456);
      let map = L.map('map', {
        crs: L.CRS.Simple
      });
      let bounds = [[0, 0], [4.22, 7.50]];
      let image = L.imageOverlay('machine-view-overview.png', bounds).addTo(map);
      map.setView([2.11, 3.75], 6);
      // map.fitBounds(bounds);
      // L.control.layers
      var littleton = L.marker([1, 1]).bindPopup('This is Littleton, CO.'),
        denver = L.marker([2, 2]).bindPopup('This is Denver, CO.'),
        aurora = L.marker([3, 3]).bindPopup('This is Aurora, CO.'),
        golden = L.marker([4, 4]).bindPopup('This is Golden, CO.');

      var cities = L.layerGroup([littleton, denver, aurora, golden]);

      var littleton1 = L.marker([0, 1]).bindPopup('This is Littleton, CO.'),
        denver1 = L.marker([1, 2]).bindPopup('This is Denver, CO.'),
        aurora1 = L.marker([2, 3]).bindPopup('This is Aurora, CO.'),
        golden1 = L.marker([3, 4]).bindPopup('This is Golden, CO.');
      var cities1 = L.layerGroup([littleton1, denver1, aurora1, golden1]);

      var overlayMaps = {
        'Cities': cities,
        'Cities1': cities1
      };

      L.control.layers(undefined, overlayMaps).addTo(map);

    }
  };
</script>
<style>
  #map {
    /*width: 400px;*/
    height: 400px;
  }
</style>
