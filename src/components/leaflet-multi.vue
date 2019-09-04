<template>
  <div class="container">
    <div id="map"></div>
    <button id="replace-img">换图</button>
    <div></div>
    <button id="alter-label">换标</button>
  </div>
</template>
<script>
  // import leaflet from 'leaflet';

  export default {
    name: 'leaflet-demo',
    data() {
      return {
        spanFontSize: '16px',
        spanVal: '',
        markerData: '别名001',
        rectPopupInfo: ['第一个', '第二个'],
        curInfoRectLatlng: [[100, 100], [350, 450], [400, 100], [400, 250]], // 暂存当前信息矩形的经纬度
        curMarkCircleLatlng: [[200, 200], [200, 350], undefined, undefined],
        linkLineLatlngs: [
          [
            [100, 100], // 矩形
            [100, 200], // 拐点
            [200, 200] // 圆上
          ],
          [
            [350, 450],
            [350, 350],
            [200, 350]
          ], undefined, undefined
        ],
        // infoRectIconSize: [80, 20], // w h
        infoRectIconColor: ['red', 'orange', 'red', 'orange'],
        rectTxtInfo: [
          {
            1: '膨胀机一组',
            2: '5685 RPM',
            fontSize: '16px'
          },
          {
            1: 'V1A10688',
            2: '18.49 um',
            fontSize: '16px'
          },
          {
            1: '--膨胀机一组--',
            2: '-5685 RPM-',
            fontSize: '16px'
          },
          {
            1: '--V1A10688--',
            2: '-18.49 um-',
            fontSize: '16px'
          }
        ]
      };
    },
    mounted() {
      let linkLine = undefined;
      let linkLineLatlngs = undefined;
      let markerLayer = undefined;
      let markerLayerGroupArr = [];
      let MyIconRect = L.DivIcon.extend({
        options: {
          className: 'my-div-icon-rect',
          // iconSize: this.infoRectIconSize,
          iconAnchor: [0, 0],
          popupAnchor: [40, 0]
        }
      });
      let selectedMarker = false;
      for (let i = 0; i < this.curInfoRectLatlng.length; i++) {
        // L.layerGroup().eachLayer(layer => {
        //   console.log(layer);
        //   markerLayerGroupArr.push(layer);
        // });
        // console.log(markerLayerGroupArr);
        // marker Div icon 类

        // div icon 实例
        // label Rect
        // 计算 rect divIcon 的宽度
        let curRectTxtInfo = this.rectTxtInfo[i];
        let resWidth = computeInfoRectWidth(curRectTxtInfo);
        let myIconRect = new MyIconRect({
          html: `
            <div style="
              font-size: ${ curRectTxtInfo.fontSize };border: 1px solid ${ this.infoRectIconColor[i] };
            ">
              <div style="border-bottom: 1px solid ${ this.infoRectIconColor[i] };">
                ${ curRectTxtInfo[1] }
              </div>
              <div>
                ${ curRectTxtInfo[2] }
              </div>
            </div>
          `,
          iconSize: [resWidth + 2, undefined]
        });
        // 圆形 div icon
        let myIconCircle = L.divIcon({
          className: 'my-div-icon-circle',
          html: `<div style="background-color: ${
            this.infoRectIconColor[i]
          };"></div>`,
        });

// *************** 牵引线 ***************
        linkLineLatlngs = this.linkLineLatlngs[i];
        if (linkLineLatlngs) {
          linkLine = L.polyline(linkLineLatlngs, {
            color: this.infoRectIconColor[i],
            weight: 1,
            className: 'polyline-shadow'
            // draggable: true
          });
          markerLayerGroupArr.push(linkLine);
        }

        // zoom the map to the polyline
        // map.fitBounds(polyline.getBounds());

// *************** 信息矩形 ***************
        let infoRect = L.marker(this.curInfoRectLatlng[i], {
          draggable: true,
          icon: myIconRect
        }).bindPopup(`${ this.rectPopupInfo[i] }`, {
          closeButton: false,
          className: 'ins-popup-style',
          offset: [0, 120]
        });
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

        });
        let popupTimer = undefined;
        // 开始拖拽矩形
        infoRect.on('drag', () => {
          infoRect.closePopup();
          popupTimer && clearTimeout(popupTimer);
        });
        infoRect.on('click', e => {
          // 关闭默认marker打开的popup
          infoRect.closePopup();
          popupTimer && clearTimeout(popupTimer);

          // test 点击后高亮背景
          // console.log(e.target);
          let _infoRectIconColor = this.infoRectIconColor;
          let _rectTxtInfo = this.rectTxtInfo;
          let isIconRectHighlight = function (idx, isShadow) {
            let resWidth = computeInfoRectWidth(_rectTxtInfo[idx]);
            return new MyIconRect({
              html: `
                <div style="
                  font-size: ${
                    curRectTxtInfo.fontSize
                  };
                  border: 1px solid ${
                    _infoRectIconColor[idx]
                  };
                ">
                  <div style="border-bottom: 1px solid ${ _infoRectIconColor[idx] };">
                    ${ _rectTxtInfo[idx][1] }
                  </div>
                  <div>
                    ${ _rectTxtInfo[idx][2] }
                  </div>
                </div>
              `,
              iconSize: [resWidth + 2, undefined],
              className: isShadow ? 'my-div-icon-rect-shadow' : 'my-div-icon-rect'
            });
          };
          let isIconCircleHighlight = function(idx, isShodow) {
            return L.divIcon({
              className: 'my-div-icon-circle',
              html: `<div style="background-color: ${
                _infoRectIconColor[idx]
              };"></div>`,
            });
          };
          let target = e.target;
          let tgName = target.name;
          let idx = tgName.split('-')[1];
          if (selectedMarker) {
            let prevIdx = selectedMarker.name.split('-')[1];
            if (selectedMarker !== target) {
              selectedMarker.setIcon(isIconRectHighlight(prevIdx, false));
              target.setIcon(isIconRectHighlight(idx, true));
              selectedMarker = target;
            } else {
              selectedMarker.setIcon(isIconRectHighlight(prevIdx, false));
              selectedMarker = false;
            }
          } else {
            selectedMarker = target;
            target.setIcon(isIconRectHighlight(idx, true));
          }
        });
        // 鼠标移入矩形，显示浮窗信息
        infoRect.on('mouseover', () => {
          popupTimer = setTimeout(() => {
            infoRect.openPopup();
          }, 500);
        });
        // 鼠标移除矩形，隐藏浮窗信息
        infoRect.on('mouseout', () => {
          infoRect.closePopup();
          popupTimer && clearTimeout(popupTimer);
        });
        markerLayerGroupArr.push(infoRect);
// *************** 标记圆形 ***************
        if (this.curMarkCircleLatlng[i]) {
          let markCircle = L.marker(this.curMarkCircleLatlng[i], {
            draggable: true,
            icon: myIconCircle
          });
          // 拖拽释放标记圆形
          markCircle.on('dragend', e => {
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

            // test 模拟更新 rect 中的信息
            let changedTxt = {
              1: '膨胀机一组检测设备',
              2: '5685 RPM',
              fontSize: '20px'
            };

            let changedResWidth = computeInfoRectWidth(changedTxt);
            let rectColor = this.infoRectIconColor;
            infoRect.setIcon(
              new MyIconRect({
                html: `
                  <div style="
                    font-size: ${ changedTxt.fontSize };border: 1px solid ${ rectColor[i] };
                  ">
                    <div style="border-bottom: 1px solid ${ rectColor[i] };">
                      ${ changedTxt[1] }
                    </div>
                    <div>
                      ${ changedTxt[2] }
                    </div>
                  </div>
                `,
                iconSize: [changedResWidth + 2, undefined]
                // className: 'my-div-icon-rect-shadow'
              })
            );
            // test 模拟更新 popup 中的信息
            infoRect.setPopupContent(`update${ (Math.random() * 100).toFixed(0) }`);
          });

          // TODO：这里需要根据是否有 入库坐标 来确定？？？
          // 如果使用的是入库坐标，是否又需要一个变量来储存么？？？
          markerLayerGroupArr.push(markCircle);
        }


        let newLayers = [];
        // 将layers放入group
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
              break;
          }
        });
      }

      // markerLayer.eachLayer(layer => {
      //   console.log(layer);
      // });

      // test
      // let littleton1 = L.marker([0, 1]).bindPopup('This is Littleton, CO.');
      // let denver1 = L.marker([1, 2]).bindPopup('This is Denver, CO.');
      // let aurora1 = L.marker([2, 3]).bindPopup('This is Aurora, CO.');
      // let golden1 = L.marker([3, 4]).bindPopup('This is Golden, CO.');
      // let markerLayer_2 = L.layerGroup([littleton1, denver1, aurora1, golden1]);

      let overlayLabels = {
        'Markers_1': markerLayer,
        // 'Markers_2': markerLayer_2
      };

// *************** map ***************
      let latlngBounds = [[0, 0], [338, 600]]; // “经纬度”边界
      let map = L.map('map', {
        maxBounds: latlngBounds,
        crs: L.CRS.Simple,
        // dragging: false,
        // minZoom: 0,
        // maxZoom: 0,
        // center: [0, 0],
        boxZoomBounds: latlngBounds,
        trackResize: false,
        zoomAnimation: false,
        inertia: false,
        layers: [markerLayer] // 默认显示的图层~
      });
      // map.setView([0, 0], 0); // 中心点，scale
      map.fitBounds(latlngBounds);
      let imgOverlay = L.imageOverlay('machine-view-overview.png', latlngBounds);
      imgOverlay.addTo(map);
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

      // test: 更改 image 图层
      // TODO: 更换图片，是否会导致比例的变更？
      let replaceBtn = document.getElementById('replace-img');
      let switchFlag = false;
      replaceBtn.addEventListener('click', () => {
        imgOverlay.setUrl(switchFlag ? 'machine-view-overview.png' : 'test-replace-pic.png');
        switchFlag = !switchFlag;
      });

      // test 窗口resize
      setTimeout(() => {
        map.invalidateSize();
      }, 5000);

      // invalidateSize 之后，map resize 触发
      map.on('resize', () => {
        alert(123);
      });

      // test: 移除layer
      // setTimeout(() => {
      //   map.removeLayer(markerLayer);
      //   console.log('finish');
      //   markerLayer.eachLayer(l => {
      //     console.log(l);
      //   })
      // }, 1000);

      // 光标从画布之外移入其中，同时发生了鼠标事件，可以监听到经纬度
      // map.on('mouseup', e => {
      //   console.log(e);
      // });

      // test 模拟功能键+鼠标多选 marker
      map.on('boxzoomend', e => {
        // console.log(e);
        let markers = [[400, 100], [400, 250]];
        let storeArr = [];
        for (let i = 0; i < markers.length; i++) {
          if (
            e.boxZoomBounds.contains(markers[i])
          ) storeArr.push(markers[i]);
        }
        if (storeArr.length === markers.length) {
          alert('全中');
          // console.log(L.layerGroup());
          markerLayer.eachLayer(layer => {
            // console.log(layer.name);
            // TODO 这里的情况，原来的name算法是有问题的
            if (layer.name && (layer.name === 'line-2' || layer.name === 'line-3')) {
              let info = this.rectTxtInfo;
              let changedResWidth = computeInfoRectWidth(info[3]);
              layer.setIcon(
                new MyIconRect({
                  html: `
                    <div style="
                      font-size: ${ info[3].fontSize };border: 1px solid ${ this.infoRectIconColor[3] };
                    ">
                      <div style="border-bottom: 1px solid ${ this.infoRectIconColor[3] };">
                        ${ info[3][1] }
                      </div>
                      <div>
                        ${ info[3][2] }
                      </div>
                    </div>
                  `,
                  iconSize: [changedResWidth + 2, undefined],
                  className: 'my-div-icon-rect-shadow'
                })
              );
            }
          });
        }
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
          // arr1[1] = cXLng < rXLng ? rXLng : rXLng + 80; // 在矩形中间的算法
          arr1[1] = rXLng;
          // arr1[0] = rYLat - 20 / 2;
          arr1[0] = rYLat;
          arr2[1] = cXLng;
          arr2[0] = arr1[0];
        } else {
          // arr1[1] = rXLng + 80 / 2;
          arr1[1] = rXLng;
          // arr1[0] = cYLat > rYLat ? rYLat : rYLat - 20;
          arr1[0] = rYLat;
          arr2[1] = arr1[1];
          arr2[0] = cYLat;
        }
        linkLineLatlngs.push(arr1);
        linkLineLatlngs.push(arr2);
        linkLineLatlngs.push(arr3);
        lineLayer.setLatLngs(linkLineLatlngs);
      }

      function computeInfoRectWidth(txtInfo) {
        let el = document.createElement('span');
        let bodyDom = document.getElementsByTagName('body')[0];
        bodyDom.appendChild(el);
        el.setAttribute('style', `font-size: ${ txtInfo.fontSize };display: inline-block;`);
        el.setAttribute('class', 'delete-span');
        let tempWidth = 0;
        for (let key in txtInfo) {
          if (txtInfo.hasOwnProperty(key) && (key === '1' || key === '2')) {
            el.innerHTML = txtInfo[key];
            let width = Number(window.getComputedStyle(el).width.split('px')[0]);
            if (width > tempWidth) tempWidth = width;
          }
        }
        let delDom = document.getElementsByClassName('delete-span')[0];
        bodyDom.removeChild(delDom);
        return tempWidth;
      }

      (function () {
        L.Map.BoxZoom.prototype._onMouseUp = function (e) {
          if ((e.which !== 1) && (e.button !== 1)) return;
          this._finish();
          if (!this._moved) return;
          this._clearDeferredResetState();
          this._resetStateTimeout = setTimeout(L.Util.bind(this._resetState, this), 0);
          let bounds = new L.LatLngBounds(
            this._map.containerPointToLatLng(this._startPoint),
            this._map.containerPointToLatLng(this._point)
          );
          this._map.fire('boxzoomend', { boxZoomBounds: bounds });
        };
      })();

    }
  };
</script>
<style>
  #map {
    /*width: 600px;*/
    width: 100%;
    height: 600px;
  }

  /*正常的矩形样式*/
  .my-div-icon-rect {
    /*border: 1px solid #fff;*/
    background-color: #fff;
    /*box-shadow: 4px 4px 3px grey;*/
  }

  /*矩形阴影*/
  .my-div-icon-rect-shadow {
    /*border: 1px solid #fff;*/
    background-color: #fff;
    box-shadow: 4px 4px 3px grey;
  }

  /*矩形内的元素*/
  .my-div-icon-rect div {
    /*position: absolute;*/
    /*top: 0;*/
    /*right: 0;*/
    /*bottom: 0;*/
    /*left: 0;*/
  }

  /*标记圆*/
  .my-div-icon-circle {
    /*background-color: red;*/
  }

  /*标记圆内的元素*/
  .my-div-icon-circle div {
    border-radius: 50%;
    position: absolute;
    top: 0;
    right: 0;
    bottom: 0;
    left: 0;
  }

  .polyline-shadow {
    /*box-shadow: 5px 5px 10px black;*/
    filter: drop-shadow(5px 5px 5px #000) !important;
    /*-webkit-filter: drop-shadow( 5px 2px 5px #000 ) !important;*/
  }

  .ins-popup-style div:nth-child(1) {
    border-radius: 0 !important;
  }

  .ins-popup-style div:nth-child(2) {
    display: none;
  }
</style>
