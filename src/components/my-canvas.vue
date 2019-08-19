<template>
  <div class="canvas">
    <div id="drawing-container"></div>
  </div>
</template>

<script>
  import zrender from 'zrender';

  export default {
    name: 'my-canvas',
    data() {
      return {
        // 被动获得的信息的变量，这里目前初始化用
        infoRectPos: [50, 50],
        infoRectSize: [50, 25],
        infoRectAlias: '别名',
        infoRectName: '真名',
        linkLinePos: [[100, 63], [125, 63], [125, 130]],
        markCirclePos: [125, 130],
        // 暂存的本地变量，逻辑层中使用
        infoRectPrevX: undefined, // 保存矩形前一次渲染的坐标
        infoRectPrevY: undefined,
        markCirclePrevX: undefined, // 保存标记点前一次渲染的坐标
        markCirclePrevY: undefined,
        linkLinePoints: undefined, // 绘制线段的三个点集合
        zr: undefined // ZRender 实例
      };
    },
    mounted() {
      this.zr = zrender.init(document.getElementById('drawing-container'));
      let zr = this.zr;

// ****************** 信息框 ******************
      let infoRect = new zrender.Rect({
        shape: {
          x: this.infoRectPos[0],
          y: this.infoRectPos[1],
          width: this.infoRectSize[0],
          height: this.infoRectSize[1]
        },
        draggable: true,
        style: {
          fill: 'rgba(255, 255, 255, 0)',
          stroke: '#000',
          text: this.infoRectAlias
        }
      });
      // hover时添加的节点
      let hoverInfoRect = new zrender.Rect({
        shape: {
          x: this.infoRectPos[0] + this.infoRectSize[0] / 2,
          y: this.infoRectPos[1] + this.infoRectSize[1] + 5,
          width: 100,
          height: 50
        },
        style: {
          fill: '#fff',
          stroke: '#ccc',
          text: this.infoRectName
        }
      });
      // 信息矩形的鼠标事件监听处理
      let hoverTimer = undefined;
      infoRect.on('mouseover', () => {
        // this.infoRectAlias = '别名1';
        infoRect.attr('style', {
          text: '别名11'
        });
        console.log('rect over');
        hoverTimer = setTimeout(() => {
          zr.add(hoverInfoRect);
        }, 500);
      });
      infoRect.on('mouseout', () => {
        console.log('rect out');
        clearTimeout(hoverTimer);
        zr.remove(hoverInfoRect);
      });
      infoRect.on('click', () => {
        console.log('rect click');
        clearTimeout(hoverTimer);
        console.log('去修改别名');
      });
      infoRect.on('dragstart', () => {
        console.log('rect dragstart');
        infoRect.off('mouseover');
        clearTimeout(hoverTimer);
        zr.remove(hoverInfoRect);
      });
      infoRect.on('dragend', () => {
        infoRect.on('mouseover', () => {
          console.log('new rect over');
          hoverTimer = setTimeout(() => {
            zr.add(hoverInfoRect);
          }, 800);
        });
      });
      infoRect.on('mouseup', cbVal => {
        console.log('rect mouseup');
        // 信息框的宽高
        let infoRectWidth = infoRect.shape.width;
        let infoRectHeight = infoRect.shape.height;
        // 移动后的相对位置
        let pos = cbVal.target.position;
        let relativeX = pos[0];
        let relativeY = pos[1];
        // 换算成绝对坐标位置
        let curRectX = Math.abs(relativeX + infoRect.shape.x);
        let curRectY = Math.abs(relativeY + infoRect.shape.x);
        // 和标记点的坐标作对比，重绘牵引线
        // 获取目前标记点的坐标
        let circleX = this.markCirclePrevX;
        let circleY = this.markCirclePrevY;
        let normalRange = (
          circleX < curRectX || circleX > curRectX + infoRectWidth
        );
        if (normalRange) {
          this.linkLinePoints = [];
          let arr1 = []; // 矩形上
          let arr2 = [circleX, undefined]; // 拐角上
          let arr3 = [circleX, circleY]; // 圆上
          arr1[0] = circleX < curRectX ? curRectX : curRectX + infoRectWidth;
          arr1[1] = curRectY + infoRectHeight / 2;
          arr2[1] = arr1[1];
          this.linkLinePoints.push(arr1);
          this.linkLinePoints.push(arr2);
          this.linkLinePoints.push(arr3);
        } else {
          this.linkLinePoints = [];
          let arr1 = [];
          let arr2 = [circleX, circleY];
          arr1[0] = curRectX + infoRectWidth / 2;
          arr1[1] = circleY <= curRectY ? curRectY : curRectY + infoRectHeight;
          this.linkLinePoints.push(arr1);
          this.linkLinePoints.push(arr2);
        }

        linkLine.attr('shape', {
          points: this.linkLinePoints
        });
        hoverInfoRect.attr('shape', {
          x: curRectX + this.infoRectSize[0] / 2,
          y: curRectY + this.infoRectSize[1] + 5
        });

        // 更新矩形坐标值
        this.infoRectPrevX = curRectX;
        this.infoRectPrevY = curRectY;

      });
      zr.add(infoRect);
      this.infoRectPrevX = infoRect.shape.x;
      this.infoRectPrevY = infoRect.shape.y;

// ****************** 牵引线 ******************
      let linkLine = new zrender.Polyline({
        shape: {
          points: this.linkLinePos
        }
      });
      zr.add(linkLine);

// ****************** 标记点 ******************
      let markCircle = new zrender.Circle({
        shape: {
          cx: this.markCirclePos[0],
          cy: this.markCirclePos[1],
          r: 5
        },
        draggable: true
      });
      // 标记点发生拖动事件时
      markCircle.on('mouseup', val => {
        // 获取标记点的新旧坐标
        // let mouseupX = val.offsetX;
        // let mouseupY = val.offsetY;
        let pos = val.target.position;
        let mouseupX = pos[0] + markCircle.shape.cx;
        let mouseupY = pos[1] + markCircle.shape.cy;


        if (
          // 判断是否第一次
          typeof this.markCirclePrevX === 'undefined'
          && typeof this.markCirclePrevY === 'undefined'
        ) {
          this.markCirclePrevX = markCircle.shape.cx;
          this.markCirclePrevY = markCircle.shape.cy;
        }
        // console.log(mouseupX);
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

        // 计算差值
        // let dX = mouseupX - this.markCirclePrevX;
        // let dY = mouseupY - this.markCirclePrevY;
        // 之后，当前鼠标坐标就成了下一次的 preVal
        this.markCirclePrevX = mouseupX;
        this.markCirclePrevY = mouseupY;

        // 处理牵引线
        if (
          // 判断是否第一次
          typeof this.linkLinePoints === 'undefined'
        ) this.linkLinePoints = linkLine.shape.points;
        // 计算新的连线位置
        // 计算矩形水平垂直位置
        // 水平范围
        // console.log(infoRect);
        let rectShape = infoRect.shape;
        // let infoRectX = rectShape.x;
        let infoRectX = this.infoRectPrevX;
        // let infoRectY = rectShape.y;
        let infoRectY = this.infoRectPrevY;
        let infoRectWidth = rectShape.width;
        let infoRectHeight = rectShape.height;
        let noBelongRect = /*(( // 鼠标位置不在矩形投影范围内
          (mouseupX < infoRectX) || (mouseupX > infoRectX + infoRectWidth)
        ) && (
          (mouseupY < infoRectY) || (mouseupY > infoRectY + infoRectHeight)
        )) || */(
          mouseupX < infoRectX || mouseupX > infoRectX + infoRectWidth
        );
        if (noBelongRect) {
          // this.linkLinePoints.forEach((val, idx) => {
          //   if (idx === 0) val[0] = mouseupX < infoRectX ? infoRectX : infoRectX + infoRectWidth;
          //   if (idx !== 0) val[0] = val[0] + dX; // 横坐标
          //   if (idx === 2) val[1] = val[1] + dY; // 纵坐标
          // });
          this.linkLinePoints = [];
          let arr1 = [];
          let arr2 = [mouseupX, undefined];
          let arr3 = [mouseupX, mouseupY];
          arr1[0] = mouseupX < infoRectX ? infoRectX : infoRectX + infoRectWidth;
          arr1[1] = infoRectY + infoRectHeight / 2;
          arr2[1] = arr1[1];
          this.linkLinePoints.push(arr1);
          this.linkLinePoints.push(arr2);
          this.linkLinePoints.push(arr3);
        } else {
          // this.linkLinePoints.splice(1, 1); // 删除中间点
          // this.linkLinePoints.forEach((val, idx) => {
          //   if (idx === 0) {
          //     if (mouseupX >= infoRectX && mouseupX <= infoRectX + infoRectWidth) {
          //       val[0] = infoRectX + infoRectWidth / 2;
          //       val[1] = mouseupY <= infoRectY ? infoRectY : infoRectY + infoRectHeight;
          //     } else if (mouseupY >= infoRectY && mouseupY <= infoRectY + infoRectHeight) {
          //       val[0] = mouseupX <= infoRectX ? infoRectX : infoRectX + infoRectWidth;
          //       val[1] = infoRectY + infoRectHeight / 2;
          //     }
          //   } else {
          //     val[0] = val[0] + dX;
          //     val[1] = val[1] + dY;
          //   }
          // });
          this.linkLinePoints = [];
          let arr1 = [];
          let arr2 = [mouseupX, mouseupY];
          // if (mouseupX >= infoRectX && mouseupX <= infoRectX + infoRectWidth) {
          arr1[0] = infoRectX + infoRectWidth / 2;
          arr1[1] = mouseupY <= infoRectY ? infoRectY : infoRectY + infoRectHeight;
          // }
          this.linkLinePoints.push(arr1);
          this.linkLinePoints.push(arr2);
        }

        // this.linkLinePoints.forEach((val, idx) => {
        //   // 目前就牵引线三点情况考虑分类处理
        //   // 还没有考虑重叠的情况
        //   if (idx !== 0) val[0] = val[0] + dX; // 横坐标
        //   if (idx === 2) val[1] = val[1] + dY; // 纵坐标
        // });
        linkLine.attr('shape', {
          points: this.linkLinePoints
        });
        // TODO：拖拽点后，线以拖拽鼠标为坐标点，但圆并不是——使用position换算呢？
        // markCircle.attr('shape', {
        //   cx: mouseupX,
        //   cy: mouseupY,
        //   r: 5
        // });
      });
      zr.add(markCircle);
      this.markCirclePrevX = markCircle.shape.cx;
      this.markCirclePrevY = markCircle.shape.cy;
    }
  };
</script>

<style scoped>
  #drawing-container {
    width: 500px;
    height: 500px;
    border: 1px solid #ccc;
  }
</style>
