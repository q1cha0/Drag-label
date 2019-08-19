<template>
  <div class="canvas">
    <!--<div-->
    <!--  class="info-user-input"-->
    <!--  v-show="isUserIptShow"-->
    <!--  :style="{left: userIptPosLeft, top: userIptTop}"-->
    <!--&gt;-->
    <!--  <input type="text" v-model="iptVal">-->
    <!--</div>-->
    <div id="drawing-container"></div>
    <!--<button @click="changeRectPos">change</button>-->
  </div>
</template>

<script>
  import zrender from 'zrender';
  import { debounce } from 'lodash';

  const xSpace = 50;
  const ySpace = 50;

  export default {
    name: 'my-canvas',
    data() {
      return {
        userIptPosLeft: 0,
        userIptTop: 0,
        isUserIptShow: false,
        iptVal: '',
        // 被动获得的信息的变量，这里目前初始化用
        infoRectPos: [[50, 50], [350, 50]], // [[], []]
        infoRectSize: [[50, 25], [50, 25]], // [[], []]
        linkLinePos: [
          [[100, 63], [125, 63], [125, 130]],
          [[350, 63], [325, 63], [325, 160]]
        ], // [[[]], [[]], [[]]]
        markCirclePos: [[125, 130], [325, 160]],
        // 暂存的本地变量，逻辑层中使用
        // infoRectPrevX: undefined, // 保存矩形前一次渲染的坐标
        // infoRectPrevY: undefined,
        // markCirclePrevX: undefined, // 保存标记点前一次渲染的坐标
        // markCirclePrevY: undefined,
        linkLinePoints: undefined, // 绘制线段的三个点集合
        group: undefined,
        zr: undefined // ZRender 实例
      };
    },
    watch: {
      iptVal(newValue) {
        console.log(newValue);
      }
    },
    mounted() {
      this.zr = zrender.init(document.getElementById('drawing-container'));
      let zr = this.zr;
      this.group = new zrender.Group();
      let group = this.group;
// ****************** 生成坐标刻度点 ******************
      // 每隔 xSpace && ySpace 生成一个定位圆
      for (let i = 1; i * xSpace < 600; i++) {
        for (let j = 1; j * ySpace < 500; j++) {
          let bgCircle = new zrender.Circle({
            shape: {
              cx: i * 50,
              cy: j * 50,
              r: 2
            },
            style: {
              fill: '#ccc'
            }
          });
          zr.add(bgCircle);
        }
      }

// ****************** 信息框 ******************
      let infoRects = this.infoRectPos;
      for (let i = 0; i < infoRects.length; i++) {
        let infoRect = new zrender.Rect({
          name: `rect-${ i }`,
          shape: {
            x: this.infoRectPos[i][0],
            y: this.infoRectPos[i][1],
            width: this.infoRectSize[i][0],
            height: this.infoRectSize[i][1]
          },
          draggable: true,
          style: {
            fill: 'rgba(255, 255, 255, 0)',
            stroke: '#000',
            text: `别名${ i }`
          }
        });
        // hover时添加的节点
        let hoverInfoRect = new zrender.Rect({
          name: `hover-rect-${ i }`,
          shape: {
            x: this.infoRectPos[i][0] + this.infoRectSize[i][0] / 2,
            y: this.infoRectPos[i][1] + this.infoRectSize[i][1] + 5,
            width: 100,
            height: 50
          },
          style: {
            fill: '#fff',
            stroke: '#ccc',
            text: `真名${ i }`
          }
        });
        // 信息矩形的鼠标事件监听处理
        let hoverTimer = undefined;
        infoRect.on('mouseover', () => {
          console.log('rect over');
          hoverTimer = setTimeout(() => {
            group.add(hoverInfoRect);
          }, 500);
        });
        infoRect.on('mouseout', () => {
          console.log('rect out');
          clearTimeout(hoverTimer);
          group.remove(hoverInfoRect);
        });
        infoRect.on('click', () => {
          console.log('rect click');
          clearTimeout(hoverTimer);
          console.log('去修改别名');
          // this.isUserIptShow = true;
        });
        infoRect.on('dragstart', () => {
          console.log('rect dragstart');
          infoRect.off('mouseover');
          clearTimeout(hoverTimer);
          group.remove(hoverInfoRect);
        });

        // infoRect.on('dragend', () => {
        //   infoRect.on('mouseover', () => {
        //     console.log('new rect over');
        //     hoverTimer = setTimeout(() => {
        //       group.add(hoverInfoRect);
        //     }, 800);
        //   });
        // });
        let rectDragEnd = cbVal => {
          console.log('rect dragend');
          // 回复mouseover事件
          infoRect.on('mouseover', () => {
            console.log('new rect over');
            hoverTimer = setTimeout(() => {
              group.add(hoverInfoRect);
            }, 800);
          });
          // 处理牵引线重绘位置
          let tag = cbVal.target.name.split('-')[1];
          let findRect = group.childOfName(`rect-${ tag }`);
          let findCircle = group.childOfName(`circle-${ tag }`);
          // 信息框的宽高
          let infoRectWidth = findRect.shape.width;
          let infoRectHeight = findRect.shape.height;
          // 移动后的相对位置
          let pos = cbVal.target.position;
          let relativeX = pos[0];
          let relativeY = pos[1];
          // 换算成绝对坐标位置
          let curRectX = Math.abs(relativeX + findRect.shape.x);
          let curRectY = Math.abs(relativeY + findRect.shape.y);
          // 和标记点的坐标作对比，重绘牵引线
          // 获取目前标记点的坐标
          let circleX = findCircle.position[0] + findCircle.shape.cx;
          let circleY = findCircle.position[1] + findCircle.shape.cy;
          let normalRange = (
            circleX < curRectX || circleX > curRectX + infoRectWidth
          );
          let arr1 = [];
          let arr2 = [];
          let arr3 = [];
          if (normalRange) {
            this.linkLinePoints = [];
            arr1 = []; // 矩形上
            arr2 = [circleX, undefined]; // 拐角
            arr3 = [circleX, circleY]; // 圆上
            arr1[0] = circleX < curRectX ? curRectX : curRectX + infoRectWidth;
            arr1[1] = curRectY + infoRectHeight / 2;
            arr2[1] = arr1[1];
          } else {
            this.linkLinePoints = [];
            arr1 = [];
            arr2 = [undefined, circleY];
            arr3 = [circleX, circleY];
            arr1[0] = curRectX + infoRectWidth / 2;
            arr1[1] = circleY <= curRectY ? curRectY : curRectY + infoRectHeight;
            arr2[0] = arr1[0];
          }
          this.linkLinePoints.push(arr1);
          this.linkLinePoints.push(arr2);
          this.linkLinePoints.push(arr3);

          linkLine.attr('shape', {
            points: this.linkLinePoints
          });
          hoverInfoRect.attr('shape', {
            x: curRectX + this.infoRectSize[i][0] / 2,
            y: curRectY + this.infoRectSize[i][1] + 5
          });
        };

        // 将mouseup统一为dragend
        // dragend： 1、执行牵引线重绘；2、判断是否要磁吸
        infoRect.on('dragend', cbVal => {
          let tag = cbVal.target.name.split('-')[1];
          let target = cbVal.target;
          let relativePos = target.position;
          let curRectPosX = target.shape.x;
          let curRectPosY = target.shape.y;
          let dragPosX = relativePos[0] + curRectPosX;
          let dragPosY = relativePos[1] + curRectPosY;
          // 判断最靠近哪个点（磁吸效果）
          let handlePosX = undefined;
          let handlePosY = undefined;
          console.log(dragPosX, dragPosY);
          let multiX = Math.floor(dragPosX / xSpace);
          let multiY = Math.floor(dragPosY / ySpace);

          if (dragPosX % xSpace <= 24 && dragPosY % ySpace <= 24) {
            handlePosX = (multiX > 0 ? multiX : 1) * xSpace;
            handlePosY = (multiY > 0 ? multiY : 1) * ySpace;
          } if (dragPosX % xSpace <= 24 && dragPosY % ySpace > 24) {
            handlePosX = (multiX > 0 ? multiX : 1) * xSpace;
            handlePosY = (multiY > 0 ? multiY + 1 : 1) * ySpace;
          } else if (dragPosX % xSpace > 24 || dragPosY % ySpace > 24) {
            handlePosX = (multiX > 0 ? multiX + 1 : 1) * xSpace;
            handlePosY = (multiY > 0 ? multiY + 1 : 1) * ySpace;
          }
          let curRectEl = group.childOfName(`rect-${ tag }`);
          curRectEl.attr({
            // 注意position是要换算成相对位置的
            position: [handlePosX - curRectPosX, handlePosY - curRectPosY],
          });
          rectDragEnd(cbVal);

          // console.log('rect dragend');
          // infoRect.on('mouseover', () => {
          //   console.log('new rect over');
          //   hoverTimer = setTimeout(() => {
          //     group.add(hoverInfoRect);
          //   }, 800);
          // });
          // let tag = cbVal.target.name.split('-')[1];
          // let findRect = group.childOfName(`rect-${ tag }`);
          // let findCircle = group.childOfName(`circle-${ tag }`);
          // // 信息框的宽高
          // let infoRectWidth = findRect.shape.width;
          // let infoRectHeight = findRect.shape.height;
          // // 移动后的相对位置
          // let pos = cbVal.target.position;
          // let relativeX = pos[0];
          // let relativeY = pos[1];
          // // 换算成绝对坐标位置
          // let curRectX = Math.abs(relativeX + findRect.shape.x);
          // let curRectY = Math.abs(relativeY + findRect.shape.y);
          // // 和标记点的坐标作对比，重绘牵引线
          // // 获取目前标记点的坐标
          // let circleX = findCircle.position[0] + findCircle.shape.cx;
          // let circleY = findCircle.position[1] + findCircle.shape.cy;
          // let normalRange = (
          //   circleX < curRectX || circleX > curRectX + infoRectWidth
          // );
          // if (normalRange) {
          //   this.linkLinePoints = [];
          //   let arr1 = []; // 矩形上
          //   let arr2 = [circleX, undefined]; // 拐角
          //   let arr3 = [circleX, circleY]; // 圆上
          //   arr1[0] = circleX < curRectX ? curRectX : curRectX + infoRectWidth;
          //   arr1[1] = curRectY + infoRectHeight / 2;
          //   arr2[1] = arr1[1];
          //   this.linkLinePoints.push(arr1);
          //   this.linkLinePoints.push(arr2);
          //   this.linkLinePoints.push(arr3);
          // } else {
          //   this.linkLinePoints = [];
          //   let arr1 = [];
          //   let arr2 = [circleX, circleY];
          //   arr1[0] = curRectX + infoRectWidth / 2;
          //   arr1[1] = circleY <= curRectY ? curRectY : curRectY + infoRectHeight;
          //   this.linkLinePoints.push(arr1);
          //   this.linkLinePoints.push(arr2);
          // }
          //
          // linkLine.attr('shape', {
          //   points: this.linkLinePoints
          // });
          // hoverInfoRect.attr('shape', {
          //   x: curRectX + this.infoRectSize[i][0] / 2,
          //   y: curRectY + this.infoRectSize[i][1] + 5
          // });

        });
        group.add(infoRect);

// ****************** 牵引线 ******************
        let linkLine = new zrender.Polyline({
          name: `line-${ i }`,
          shape: {
            points: this.linkLinePos[i]
          },
          cursor: ''
        });
        group.add(linkLine);

// ****************** 标记点 ******************
        let markCircle = new zrender.Circle({
          name: `circle-${ i }`,
          shape: {
            cx: this.markCirclePos[i][0],
            cy: this.markCirclePos[i][1],
            r: 5
          },
          draggable: true
        });
        // 标记点发生拖动事件时
        // 将mouseup改成了dragend
        markCircle.on('dragend', val => {
          console.log(val);
          let tag = val.target.name.split('-')[1];
          let findRect = group.childOfName(`rect-${ tag }`);
          let findCircle = group.childOfName(`circle-${ tag }`);
          console.log(findRect);
          // 获取标记圆的新坐标
          // 相对坐标
          let pos = val.target.position;
          // 绝对坐标
          let mouseupX = pos[0] + findCircle.shape.cx;
          let mouseupY = pos[1] + findCircle.shape.cy;

          // TODO：若拖拽到边缘位置
          // if (mouseupX <= 10 || 500 - mouseupX <= 10) {
          //   console.log('范围小');
          //   markCircle.attr({
          //     position: {
          //       x: 180,
          //       y: 180
          //     }
          //   });
          //   return false;
          // }

          // 处理牵引线
          if (
            // 判断是否第一次
            typeof this.linkLinePoints === 'undefined'
          ) this.linkLinePoints = linkLine.shape.points;
          // 计算新的连线位置
          // 计算矩形水平垂直位置
          // 水平范围
          let rectShape = findRect.shape;
          let infoRectX = findRect.position[0] + rectShape.x;
          let infoRectY = findRect.position[1] + rectShape.y;
          let infoRectWidth = rectShape.width;
          let infoRectHeight = rectShape.height;
          let noBelongRect = (
            mouseupX < infoRectX || mouseupX > infoRectX + infoRectWidth
          );
          let arr1 = [];
          let arr2 = [];
          let arr3 = [];
          if (noBelongRect) {
            this.linkLinePoints = [];
            arr1 = [];
            arr2 = [mouseupX, undefined];
            arr3 = [mouseupX, mouseupY];
            arr1[0] = mouseupX < infoRectX ? infoRectX : infoRectX + infoRectWidth;
            arr1[1] = infoRectY + infoRectHeight / 2;
            arr2[1] = arr1[1];
          } else {
            this.linkLinePoints = [];
            arr1 = [];
            arr2 = [undefined, mouseupY];
            arr3 = [mouseupX, mouseupY];
            arr1[0] = infoRectX + infoRectWidth / 2;
            arr1[1] = mouseupY <= infoRectY ? infoRectY : infoRectY + infoRectHeight;
            arr2[0] = arr1[0];
          }
          this.linkLinePoints.push(arr1);
          this.linkLinePoints.push(arr2);
          this.linkLinePoints.push(arr3);
          linkLine.attr('shape', {
            points: this.linkLinePoints
          });
        });
        group.add(markCircle);

        zr.add(group);
      }

    },
    methods: {
      showInfo() {
        let val = this.group.childOfName('info2');
        // let val = this.group.children();
        console.log(val);
      },
      // changeRectPos() {
      //   let el = this.group.childOfName('rect-1');
      //   console.log(el);
      //   el.attr('position', [100, 100]);
      // }
    },
  };
</script>

<style scoped>
  #drawing-container {
    width: 600px;
    height: 500px;
    border: 1px solid #ccc;
  }

  .info-user-input {
    position: absolute;
    top: 0;
    left: 0;
  }
</style>
