1. 根据初始化配置数据，绘制目标图形
2. 开启 `ZRender ` 默认可拖拽效果
3. 获得实时位置，这个 `ZRender` 中目前没找到接口，使用监听鼠标事件和 `BOM `接口实现
4. 根据3，update/refresh 对应图形位置
5. 图形字体文本的配置和更新的问题
6. 拖拽：超出边界；重叠
7. 修改图形大小，若获取的还是初始属性值，是否要remove后再add（★）
8. 如何获取到拖拽之后的位置，`target / topTarget`
9. 矩形 圆形，更新位置后，相互更新的处理
10. `drag`和 `mouseup` 冲突导致的问题（★）——搞个定时器？
11. 初始化，若没有 牵引线和标记点的位置，要如何处理？





https://github.com/1wheel/swoopy-drag/blob/gh-pages/lib/d3v4%2Bjetpack.js