# threejs-basic


## 资料
### 动画库（gsap）
- [文档](https://greensock.com/docs/)
- [文档指导例子](https://greensock.com/get-started/)

### ui界面控制库
- [dat.gui](https://www.npmjs.com/package/dat.gui)


## 搭建项目
使用parcel

- [官网v1](https://www.parceljs.cn/)
- [官网v2](https://v2.parceljs.cn/blog/alpha1/)
```bash
npm init -y
# pnpm add parcel-bundler --save-dev
pnpm add parcel --save-dev
# 动画库安装gsap
pnpm add three gsap dat.gui
```

```js
{
  "scripts": {
    "dev": "parcel <your entry file>",
    "build": "parcel build <your entry file>"
  }
}
```


## 几何体属性
geometry
### BufferGeometry
```js
cubeGeometry： {
  attributes： {
    // 法相量
    normal,

    // 顶点位置
    position,

    // UV 坐标(二维平面展开图)
    uv
  }
}
```


## 材质
material


## PBR
### 什么是PBR
- 基于物理渲染
- 以前的渲染是在模仿灯光的外观
- 现在是在模仿光的实际行为
- 试图形看起来更真实

### PBR组成部分
- 灯光属性：直接照明、间接照明、直接高光、间接高光、阴影、环境光闭塞
- 表面属性：基础色、法线、高光、粗糙度、金属度

### 灯光属性
#### 光线类型
入射
- 直接照明：直接从光源发射阴影物体表面的光。
- 间接照明：环境光和直接光经过反射第二次进入的光。
反射光
- 镜面光：在经过表面反射聚焦在同一方向上进入人眼的高亮光。
- 漫发射：光被散射并沿着各个方向离开表面。

#### 光与表面相互作用类型
- 直接漫反射：从源头到四面八方散发出来的直接高光
- 直接高光：直接来自光源并被集中反射的光
- 间接漫反射：来自环境的光被表面散射的光
- 间接高光：来自环境光并被集中反射的光

##### 直接漫反射
- 直接来自光源的光
- 撞击表面后散落在各个方向
- 在着色器中使用简单的数学计算

##### 直接高光
- 直接来自光源的光
- 反射在一个更集中的方向上
- 在着色器中使用简单的数学计算
直接镜面反射的计算成本比漫反射低很多

##### 间接漫反射
- 来自环境中各个方向的光
- 撞击表面后散落在各个方向
- 因为计算昂贵，所以引擎的全局照明解决方案通常会离线渲染，并被烘培成灯光地图

##### 镜面反射
- 来自环境中各个方向的光
- 反射在一个更集中的方向上
- 引擎中使用反射探头，平面反射，SSR，或射线追踪计算

### 表面属性
#### 基础色
- 定义表面的漫反射颜色
- 真实世界的材料不会比20暗或比240sRGB亮
- 粗糙表面具有更高的最低 - 50srgb
- 超出范围的值不能正确发光，所以保持在范围内是至关重要的

基础色贴图制作注意点：
- 不包括任何照明或阴影
- 基本颜色纹理看起来应该非常平坦
- 使用真实世界的度量或获取最佳结果的数据

#### 法线
- 定义曲面的形状每个像素代表一个矢量
- 该矢量指示表面所面对的方向即便网格是完全平坦的
- 法线贴图会使表面显的凹凸不平
- 用于添加便面形状的细节，这里三角形是实现不了的
- 因为它们表示矢量数据，所以法线贴图是无法手工绘制的

#### 镜面
- 用于直接和间接镜面照明的叠加
- 当直视表面时，定义反射率
- 非金属表面反射约4%的光
- 0.5代表4%的反射
- 1.0 代表8%的反射但对于大多数物体来说太高了
- 在掠射角下，所有表面都是100%反射的，内置于引擎中的菲涅耳项

镜面贴图制作注意点：
- 高光贴图应该大多在0.5
- 使用深色的阴影来遮盖不应该反射的裂缝
- 一个裂缝贴图乘以0.5就是一个很好的高光贴图

#### 粗糙度
- 表面在微观尺度上的粗糙度
- 白色是粗糙的
- 黑色是光滑的
- 控制反射的焦点
- 平滑=强烈的反射
- 粗糙=模糊的，漫反射

粗糙度贴图制作注意点：
- 没有技术限制-完全艺术的选择
- 艺术家可以使用这张地图来定义表面的特性，并展示它的历史
- 考虑一下被打磨光滑，磨损或者老化的表面

#### 金属度
- 两个不同的着色器通过金属度混合它们
- 基本色变成高光色而不是漫反射颜色
- 金属漫反射是黑色的
- 在底色下，镜面范围可达100%
- 大多数金属的反射性在60%到100%之间
- 确保对金属颜色值使用真实世界的测量点，并保持它们明亮
- 当金属为1时，镜面输入将被忽略

金属度贴图制作注意点：
- 将着色器切换到金属模式
- 灰度值会很奇怪，最好使用纯白色或黑色
- 当金属色为白色时，请确保使用正确的金属底色值
- 没有黑暗金属这回事
- 所有金属均为180srgb或更亮

非金属
- 基础颜色=漫反射
- 镜面反射=0~8%

金属
- 基础颜色=0~100%的镜面反射
- 镜面=0%
- 漫反射总是黑色的

### 总结
- PBR是基于物理渲染的着色模型，PBR着色模型分为材质和灯光两个属性
- 材质部分由：基础色、法线、高光、粗糙度、金属度来定义材质表面属性的
- 灯光部分由：直接照明、间接照明、直接高光、间接高光、阴影、环境光闭塞来定义照明属性的
- 通常我们写材质的时候只需要关注材质部分的属性即可，灯光属性都是引擎定义好的直接使用即可
- PBR渲染模型不但指的是PBR材质，还有灯光，两者缺一不可


## 贴图资料网站
- [poliigon](https://www.poliigon.com/)
- [3dtextures](https://3dtextures.me/)
- [arroway-textures](https://www.arroway-textures.ch/)
- Quixel Bridge 软件
  - 得注册一个虚幻引擎的账号，登录账号后，所有贴图都是免费的。


## 点材质贴图网站
- [particle-pack](https://kenney.nl/assets/particle-pack)
- [iconfont](https://iconfont.cn)
- [aigei](https://www.aigei.com/)


## HDR详解
HDR技术是一种改善动态对比度的技术。 