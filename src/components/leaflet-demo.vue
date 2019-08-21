<template>
  <div class="container">
    <div id="map"></div>
    <!--<button @click="changeData">click</button>-->
  </div>
</template>
<script>
  // import leaflet from 'leaflet';

  export default {
    name: 'leaflet-demo',
    data() {
      return {
        markerData: '别名1-0.01'
      };
    },
    mounted() {
      let map = L.map('map', {
        crs: L.CRS.Simple
      });
      let bounds = [[0, 0], [4.22, 7.50]];
      L.imageOverlay('machine-view-overview.png', bounds).addTo(map);
      // map.setView([2.11, 3.75], 6); // 中心点，scale
      map.fitBounds(bounds);
      // L.control.layers

      let myIconRect_1 = L.divIcon({
        className: 'my-div-icon-rect',
        html: `<span>${ this.markerData }</span>`,
        iconSize: [80, 20],
        iconAnchor: [80, 10]
      });
      let myIconCircle_1 = L.divIcon({
        className: 'my-div-icon-circle',
      });
      // you can set .my-div-icon styles in CSS
      // L.marker([50.505, 30.57], { icon: myIcon }).addTo(map);

      let linkLine_latlngs_1 = [ // lat:维度-Y lng:经度-X
        [1, 1],
        [1, 2],
        [3, 2]
      ];
      let linkLine_1 = L.polyline(linkLine_latlngs_1, {
        color: 'red',
        draggable: true
      });
      // zoom the map to the polyline
      // map.fitBounds(polyline.getBounds());

      // 添加信息矩形
      let labelRect_1 = L.marker([1, 1], {
        draggable: true,
        icon: myIconRect_1,
      }).bindPopup('真名1');
      labelRect_1.on('dragend', e => {
        // 首先判断处理磁吸效果
        let autoXLng = undefined;
        let autoYLat = undefined;
        let curLatlng = e.target._latlng;
        let xLng = (curLatlng.lng).toFixed(6);
        let yLat = (curLatlng.lat).toFixed(6);
        console.log(xLng, yLat);
        let xLngFrac = Number(xLng.split('.')[1][0]);
        let yLngFrac = Number(yLat.split('.')[1][0]);
        if (xLngFrac <= 5 && yLngFrac <= 5) {
          autoXLng = Math.floor(Number(xLng));
          autoYLat = Math.floor(Number(yLat));
        } else if (xLngFrac <= 5 && yLngFrac > 5) {
          autoXLng = Math.floor(Number(xLng));
          autoYLat = Math.ceil(Number(yLat));
        } else if (xLngFrac > 5 && yLngFrac <= 5) {
          autoXLng = Math.ceil(Number(xLng));
          autoYLat = Math.floor(Number(yLat));
        } else if (xLngFrac > 5 && yLngFrac > 5) {
          autoXLng = Math.ceil(Number(xLng));
          autoYLat = Math.ceil(Number(yLat));
        }
        labelRect_1.setLatLng([autoYLat || yLat, autoXLng || xLng]);

        linkLine_latlngs_1[0] = [autoYLat || yLat, autoXLng || xLng];
        linkLine_1.setLatLngs(linkLine_latlngs_1);

      });
      // 指示圆形
      let labelCircle_1 = L.marker([3, 2], {
        draggable: true,
        icon: myIconCircle_1
      });
      labelCircle_1.on('dragend', e => {
        console.log(e.target._latlng);
        linkLine_latlngs_1[2] = e.target._latlng;
        linkLine_1.setLatLngs(linkLine_latlngs_1);

        this.markerData = '别名' + (Math.random() * 100).toFixed(0);
        labelRect_1.setIcon(
          L.divIcon({
            className: 'my-div-icon-rect',
            html: `<span>${ this.markerData }</span>`,
            iconSize: [80, 20],
            iconAnchor: [80, 10]
          })
        );
      });

      let labelRect_2 = L.marker([4, 6]).bindPopup('This is Denver, CO.');

      let cities = L.layerGroup([labelRect_1, labelCircle_1, labelRect_2, linkLine_1]);

      // test
      let littleton1 = L.marker([0, 1]).bindPopup('This is Littleton, CO.');
      let denver1 = L.marker([1, 2]).bindPopup('This is Denver, CO.');
      let aurora1 = L.marker([2, 3]).bindPopup('This is Aurora, CO.');
      let golden1 = L.marker([3, 4]).bindPopup('This is Golden, CO.');
      let cities1 = L.layerGroup([littleton1, denver1, aurora1, golden1]);

      let overlayMaps = {
        'Cities': cities,
        'Cities1': cities1
      };

      L.control.layers(undefined, overlayMaps).addTo(map);

    },
    methods: {
      changeData() {
        this.markerData = '123';
      }
    },
  };
</script>
<style>
  #map {
    /*width: 400px;*/
    height: 400px;
  }

  .my-div-icon-rect {
    /*width: 80px !important;*/
    /*height: 20px !important;*/
    border: 1px solid #fff;
    background-color: #fff;
    /*position: relative;*/
    /*left: -50%;*/
    /*top: -50% !important;*/
  }

  .my-div-icon-circle {
    width: 3px;
    height: 3px;
    border-radius: 50%;
    background-color: #fff;
  }
</style>
