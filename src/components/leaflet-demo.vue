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
        markerData: '别名001'
      };
    },
    mounted() {
      // marker Div icon 类
      let MyIconRect = L.DivIcon.extend({
        options: {
          className: 'my-div-icon-rect',
          iconSize: [80, 20],
          // iconAnchor: [85, 10]
        }
      });
      // 具体的实例配置
      // label Rect 的
      let myIconRect_1 = new MyIconRect({
        html: `<span>${this.markerData}</span>`,
      });
      // label circle 的
      let myIconCircle_1 = L.divIcon({
        className: 'my-div-icon-circle',
      });

      // 牵引线
      let linkLine_latlngs_1 = [ // lat:维度-Y lng:经度-X
        [100, 100],
        [100, 200],
        [300, 200]
      ];
      let linkLine_1 = L.polyline(linkLine_latlngs_1, {
        color: 'red',
        draggable: true
      });
      // zoom the map to the polyline
      // map.fitBounds(polyline.getBounds());

      // 绘制信息矩形及其事件
      let labelRect_1 = L.marker([100, 100], {
        draggable: true,
        icon: myIconRect_1,
      }).bindPopup('真名1');
      labelRect_1.on('dragend', e => {
        // 首先处理磁吸效果
        let autoXLng = undefined;
        let autoYLat = undefined;
        let curLatlng = e.target._latlng;
        let xLng = curLatlng.lng;
        let yLat = curLatlng.lat;
        console.log(xLng, yLat);
        // let xLngFrac = Number(xLng.split('.')[1][0]);
        // let yLngFrac = Number(yLat.split('.')[1][0]);
        let yuXLng = xLng % 50; // 取余
        let reYuXLng = (yuXLng > 0 ? 50 : -50) - yuXLng; // 余数的反面
        let yuYLat = yLat % 50;
        let reYuYLat = (yuYLat > 0 ? 50 : -50) - yuYLat;
        if (Math.abs(yuXLng) <= 25 && Math.abs(yuYLat) <= 25) {
          autoXLng = xLng - yuXLng;
          autoYLat = yLat - yuYLat;
        } else if (Math.abs(yuXLng) <= 25 && Math.abs(yuYLat) > 25) {
          autoXLng = xLng - yuXLng;
          autoYLat = yLat + reYuYLat;
        } else if (Math.abs(yuXLng) > 25 && Math.abs(yuYLat) <= 25) {
          autoXLng = xLng + reYuXLng;
          autoYLat = yLat - yuYLat;
        } else if (Math.abs(yuXLng) > 25 && Math.abs(yuYLat) > 25) {
          autoXLng = xLng + reYuXLng;
          autoYLat = yLat + reYuYLat;
        }
        labelRect_1.setLatLng([autoYLat, autoXLng]);

        // 重置牵引线的坐标
        linkLine_latlngs_1[0] = [autoYLat, autoXLng];
        linkLine_1.setLatLngs(linkLine_latlngs_1);

      });
      // 标记圆形
      let labelCircle_1 = L.marker([300, 200], {
        draggable: true,
        icon: myIconCircle_1
      });
      labelCircle_1.on('dragend', e => {
        console.log(e.target._latlng);
        // 更新牵引线坐标
        let lineY = linkLine_latlngs_1[1][0];
        linkLine_latlngs_1[1] = [lineY, e.target._latlng.lng];
        linkLine_latlngs_1[2] = e.target._latlng;
        linkLine_1.setLatLngs(linkLine_latlngs_1);

        // test 更新icon中的信息
        this.markerData = '别名' + (Math.random() * 100).toFixed(0);
        labelRect_1.setIcon(
          new MyIconRect({
            html: `<span>${this.markerData}</span>`
          })
        );
      });

      let labelRect_2 = L.marker([400, 600]).bindPopup('This is Denver, CO.');
      let markerLayer_1 = L.layerGroup([labelRect_1, labelCircle_1, linkLine_1, labelRect_2]);

      // test
      let littleton1 = L.marker([0, 1]).bindPopup('This is Littleton, CO.');
      let denver1 = L.marker([1, 2]).bindPopup('This is Denver, CO.');
      let aurora1 = L.marker([2, 3]).bindPopup('This is Aurora, CO.');
      let golden1 = L.marker([3, 4]).bindPopup('This is Golden, CO.');
      let markerLayer_2 = L.layerGroup([littleton1, denver1, aurora1, golden1]);

      let overlayLabels = {
        'Markers_1': markerLayer_1,
        'Markers_2': markerLayer_2
      };

      let latlngBounds = [[0, 0], [338, 600]]; // 经纬度边界
      let map = L.map('map', {
        maxBounds: latlngBounds,
        crs: L.CRS.Simple,
        dragging: false,
        // minZoom: 7,
        // maxZoom: 7,
        // center: [0, 0],
        layers: [markerLayer_1] // 初始化默认显示的layer
      });
      // map.setView([0, 0], 0); // 中心点，scale
      map.fitBounds(latlngBounds);
      L.imageOverlay('machine-view-overview.png', latlngBounds).addTo(map);
      L.control.layers(null, overlayLabels).addTo(map);

      // 刻度线层
      L.GridLayer.AutoAlignLine = L.GridLayer.extend({
        createTile: function () {
          let tile = document.createElement('div');
          let size = this.getTileSize();
          tile.style.outline = '1px solid #999';
          tile.width = size.x;
          tile.height = size.y;
          return tile;
        }
      });
      L.gridLayer.autoAlignLine = function (opts) {
        return new L.GridLayer.AutoAlignLine(opts);
      };

      map.addLayer(
        L.gridLayer
          .autoAlignLine(
            {
              tileSize: 50,
              pane: 'overlayPane',
              opacity: 0.3
            }
          )
      );

      // map.on('click', e => {
      //   L.popup()
      //     .setLatLng(e.latlng)
      //     .setContent("You clicked the map at " + e.latlng.toString())
      //     .openOn(map);
      // });

      // check some info
      // setTimeout(() => {
      //   console.log(map.getSize());
      // }, 2000);
      // console.log(map.getSize());
      // console.log(map.getBounds());
      // console.log(map.getPixelBounds());

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
    width: 600px;
    height: 600px;
  }

  .my-div-icon-rect {
    border: 1px solid #fff;
    background-color: #fff;
  }

  .my-div-icon-circle {
    width: 3px;
    height: 3px;
    border-radius: 50%;
    background-color: #fff;
  }
</style>
