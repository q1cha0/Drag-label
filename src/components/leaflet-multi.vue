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
        markerData: '别名001',
        curInfoRectLatlng: [[100, 100], [350, 450]], // 暂存当前信息矩形的经纬度
        curMarkCircleLatlng: [[200, 200], [200, 350]],
        linkLineLatlngs: [
          [
            [90, 180], // 矩形
            [90, 200], // 拐点
            [200, 200] // 圆上
          ],
          [
            [340, 450],
            [340, 350],
            [200, 350]
          ]
        ],
        infoRectIconSize: [80, 20], // w h
        infoRectIconColor: ['blue', 'orange']
      };
    },
    mounted() {
      let linkLine = undefined;
      let linkLineLatlngs = undefined;
      let markerLayer = undefined;
      let markerLayerGroupArr = [];
      for (let i = 0; i < this.curInfoRectLatlng.length; i++) {
        L.layerGroup().eachLayer(layer => {
          markerLayerGroupArr.push(layer);
        });
        // console.log(markerLayerGroupArr);
        // marker Div icon 类
        let MyIconRect = L.DivIcon.extend({
          options: {
            className: 'my-div-icon-rect',
            iconSize: this.infoRectIconSize,
            iconAnchor: [0, 0],
            popupAnchor: [40, 0]
          }
        });
        // div icon 实例
        // label Rect
        let myIconRect = new MyIconRect({
          html: `<div style="border: 1px solid ${
            this.infoRectIconColor[i]
          };">${ this.markerData }</div>`,
        });
        // label circle
        let myIconCircle = L.divIcon({
          className: 'my-div-icon-circle',
          html: `<div style="background-color: ${
            this.infoRectIconColor[i]
          };"></div>`,
        });

// *************** 牵引线 ***************
        linkLineLatlngs = this.linkLineLatlngs[i];
        linkLine = L.polyline(linkLineLatlngs, {
          color: this.infoRectIconColor[i],
          weight: 1,
          draggable: true
        });
        markerLayerGroupArr.push(linkLine);

        // zoom the map to the polyline
        // map.fitBounds(polyline.getBounds());

// *************** 信息矩形 ***************
        let infoRect = L.marker(this.curInfoRectLatlng[i], {
          draggable: true,
          icon: myIconRect
        }).bindPopup('真名1');
        // 拖拽释放信息矩形
        infoRect.on('dragend', e => {
          let flag = i;
          // 磁吸
          let autoXLng = undefined;
          let autoYLat = undefined;
          let newLatlng = e.target._latlng;
          let newRectXLng = newLatlng.lng;
          let newRectYLat = newLatlng.lat;
          let rmXLng = newRectXLng % 50; // 取余 remainder
          let rmYLat = newRectYLat % 50;
          let rvRmXLng = (rmXLng > 0 ? 50 : -50) - rmXLng; // 余数的反面 reverse
          let rvRmYLat = (rmYLat > 0 ? 50 : -50) - rmYLat;
          let absRmXLng = Math.abs(rmXLng);
          let absRmYLat = Math.abs(rmYLat);
          if (absRmXLng <= 25 && absRmYLat <= 25) {
            autoXLng = newRectXLng - rmXLng;
            autoYLat = newRectYLat - rmYLat;
          } else if (absRmXLng <= 25 && absRmYLat > 25) {
            autoXLng = newRectXLng - rmXLng;
            autoYLat = newRectYLat + rvRmYLat;
          } else if (absRmXLng > 25 && absRmYLat <= 25) {
            autoXLng = newRectXLng + rvRmXLng;
            autoYLat = newRectYLat - rmYLat;
          } else if (absRmXLng > 25 && absRmYLat > 25) {
            autoXLng = newRectXLng + rvRmXLng;
            autoYLat = newRectYLat + rvRmYLat;
          }
          let autoLatLng = [autoYLat, autoXLng];
          infoRect.setLatLng(autoLatLng);

          // 改变牵引线经纬度
          let curMarkCircleXLng = undefined;
          let curMarkCircleYLat = undefined;
          let linkLineLayer = undefined;
          markerLayer.eachLayer(layer => {
            if (
              layer['name'] && layer['name'] === `circle-${ flag }`
            ) {
              curMarkCircleXLng = layer._latlng.lng;
              curMarkCircleYLat = layer._latlng.lat;
            }
            if (
              layer['name'] && layer['name'] === `line-${ flag }`
            ) linkLineLayer = layer;
          });
          updateLinkLine(
            curMarkCircleXLng,
            curMarkCircleYLat,
            autoXLng,
            autoYLat,
            linkLineLayer
          );

          // 更新信息矩形的实时坐标
          this.curInfoRectLatlng[i] = autoLatLng;

        });
        // 开始拖拽矩形
        infoRect.on('dragstart', () => {
          infoRect.closePopup();
        });
        // 鼠标移入矩形，显示浮窗信息
        infoRect.on('mouseover', () => {
          // infoRect.togglePopup();
        });
        // 鼠标移除矩形，隐藏浮窗信息
        infoRect.on('mouseout', () => {
          // infoRect.togglePopup();
        });
        markerLayerGroupArr.push(infoRect);
// *************** 标记圆形 ***************
        let markCircle_1 = L.marker(this.curMarkCircleLatlng[i], {
          draggable: true,
          icon: myIconCircle
        });
        // 拖拽释放标记圆形
        markCircle_1.on('dragend', e => {
          let flag = i;
          let newLatlng = e.target._latlng;
          let newCircleXLng = newLatlng.lng;
          let newCircleYLat = newLatlng.lat;
          // 重绘牵引线
          let curInfoRectXLng = undefined;
          let curInfoRectYLat = undefined;
          let linkLineLayer = undefined;
          markerLayer.eachLayer(layer => {
            if (
              layer['name'] && layer['name'] === `rect-${ flag }`
            ) {
              curInfoRectXLng = layer._latlng.lng;
              curInfoRectYLat = layer._latlng.lat;
            }
            if (
              layer['name'] && layer['name'] === `line-${ flag }`
            ) linkLineLayer = layer;
          });
          updateLinkLine(
            newCircleXLng,
            newCircleYLat,
            curInfoRectXLng,
            curInfoRectYLat,
            linkLineLayer
          );

          // 更新标记圆形的实时坐标
          this.curMarkCircleLatlng[i] = [newCircleYLat, newCircleXLng];
          // 更新icon中的信息
          this.markerData = '别名' + (Math.random() * 100).toFixed(0);
          infoRect.setIcon(
            new MyIconRect({
              html: `<span>${ this.markerData }</span>`
            })
          );
        });

        markerLayerGroupArr.push(markCircle_1);

        let newLayers = [];
        // 将上述layer放入group
        markerLayer = L.layerGroup(markerLayerGroupArr);
        markerLayer.eachLayer(layer => {
          if (
            !layer.hasOwnProperty('name')
          ) newLayers.push(layer);
        });
        newLayers.forEach((val, idx) => {
          switch (idx) {
            case 0:
              val['name'] = `line-${ i }`;
              break;
            case 1:
              val['name'] = `rect-${ i }`;
              break;
            case 2:
              val['name'] = `circle-${ i }`;
          }
        });
      }

      // markerLayer.eachLayer(layer => {
      //   console.log(layer);
      // });

      // test
      let littleton1 = L.marker([0, 1]).bindPopup('This is Littleton, CO.');
      let denver1 = L.marker([1, 2]).bindPopup('This is Denver, CO.');
      let aurora1 = L.marker([2, 3]).bindPopup('This is Aurora, CO.');
      let golden1 = L.marker([3, 4]).bindPopup('This is Golden, CO.');
      let markerLayer_2 = L.layerGroup([littleton1, denver1, aurora1, golden1]);

      let overlayLabels = {
        'Markers_1': markerLayer,
        'Markers_2': markerLayer_2
      };

// *************** map ***************
      let latlngBounds = [[0, 0], [338, 600]]; // “经纬度”边界
      let map = L.map('map', {
        maxBounds: latlngBounds,
        crs: L.CRS.Simple,
        dragging: false,
        minZoom: 0,
        maxZoom: 0,
        // center: [0, 0],
        layers: [markerLayer] // 默认显示的图层~
      });
      // map.setView([0, 0], 0); // 中心点，scale
      map.fitBounds(latlngBounds);
      L.imageOverlay('machine-view-overview.png', latlngBounds).addTo(map);
      L.control.layers(null, overlayLabels).addTo(map);

      // 添加磁吸对齐线层
      // 扩展gridlayer类
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
      // 实例
      L.gridLayer.autoAlignLine = function (opts) {
        return new L.GridLayer.AutoAlignLine(opts);
      };
      // 添加到layer
      map.addLayer(
        L.gridLayer.autoAlignLine({ pane: 'overlayPane', tileSize: 50, opacity: 0.3 })
      );

      // 光标从画布之外移入其中，同时发生了鼠标事件，可以监听到经纬度
      map.on('mouseup', e => {
        console.log(e);
      });

      function updateLinkLine(cXLng, cYLat, rXLng, rYLat, lineLayer) {
        // 第1个就是矩形，第2个坐标y和矩形同，x和圆形同，第3个总是当前圆的经纬度
        let arr1 = []; // 矩形
        let arr2 = []; // 拐角
        let arr3 = [cYLat, cXLng]; // 圆上
        let normalRange = (
          cXLng < rXLng || // 圆形在矩形左边界的左边
          cXLng > rXLng + 80 // 圆形在矩形右边界的右边
        );
        linkLineLatlngs = [];
        if (normalRange) {
          arr1[1] = cXLng < rXLng ? rXLng : rXLng + 80;
          arr1[0] = rYLat - 20 / 2;
          arr2[1] = cXLng;
          arr2[0] = arr1[0];
        } else {
          arr1[1] = rXLng + 80 / 2;
          arr1[0] = cYLat > rYLat ? rYLat : rYLat - 20;
          arr2[1] = arr1[1];
          arr2[0] = cYLat;
        }
        linkLineLatlngs.push(arr1);
        linkLineLatlngs.push(arr2);
        linkLineLatlngs.push(arr3);
        lineLayer.setLatLngs(linkLineLatlngs);
      }

    }
  };
</script>
<style>
  #map {
    width: 600px;
    height: 600px;
  }

  .my-div-icon-rect {
    /*border: 1px solid #fff;*/
    background-color: #fff;
  }

  .my-div-icon-rect div {
    position: absolute;
    top: 0;
    right: 0;
    bottom: 0;
    left: 0;
  }

  .my-div-icon-circle {
    /*background-color: red;*/
  }
  .my-div-icon-circle div {
    border-radius: 50%;
    position: absolute;
    top: 0;
    right: 0;
    bottom: 0;
    left: 0;
  }
</style>
