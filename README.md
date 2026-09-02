# NOËL · Gesture Christmas Tree

基于 **Three.js + MediaPipe Hands** 的手势控制 3D 粒子圣诞树。单文件应用，打开即用，无需构建。

圣诞树由 1,640 个实例化元素（哑光绿松针、金属金彩带、红/金装饰球、礼物方块、扭扭糖果棍）与可上传的**照片云**构成，支持**合拢 / 散开 / 照片放大**三态平滑切换。

---

## 在线体验

> 仓库：<https://github.com/zheng-qinghe/tree>
> GitHub Pages：<https://zheng-qinghe.github.io/tree/>
> （摄像头需要 HTTPS 或 localhost，Pages 为 HTTPS，可直接授权）

## 三种状态

| 状态 | 说明 | 视觉 |
|---|---|---|
| **合拢态** | 全部元素收拢聚合为一棵圣诞树 | 松针密铺圆锥 + 金色螺旋彩带 + 顶部伯利恒之星 + 底部光环 |
| **散开态** | 元素在球壳内无序漂浮 | 3,190 粒子云，照片缓慢自转并软朝相机 |
| **照片放大态** | 背景保持散开，单张照片推至镜头前放大 4× | 其余照片半透明虚化，辉光增强 |

状态切换使用**全局形变进度 + 逐元素错峰延迟**（0~0.55s），配合 smoothstep 缓动，无跳变。

## 手势映射

| 手势 | 判定 | 动作 |
|---|---|---|
| ✊ 握拳 | 伸展手指 ≤ 1 | 回到合拢态 |
| ✋ 张开五指 | 伸展手指 ≥ 4 | 进入散开态 |
| ✋ 手掌平移 | 掌心坐标增量 | 相机环绕（theta / phi，带阻尼） |
| ✋ 翻腕 | 腕→中指根向量夹角增量 | 整片星云持续旋转（spinVel 阻尼衰减） |
| 🤏 捏合抓取 | 拇指-指尖距 / 掌宽 < 0.34 | 抓取掌心处最近的照片并放大 |

手势识别采用 **6 帧多数表决**去抖，状态切换带 **520ms 冷却**，捏合带 latch 防重复触发。

## 无摄像头回退

| 操作 | 效果 |
|---|---|
| 拖拽 | 相机环绕 |
| 滚轮 | 缩放 |
| 点击照片 / 空白 | 放大 / 退出放大 |
| `1` `2` `3` | 合拢 / 散开 / 随机放大 |
| `Esc` | 退出放大 |
| `C` / `H` | 开关摄像头 / 隐藏面板 |
| 演示模式按钮 | 每 7s 自动轮播三态 |

MediaPipe 模型加载失败时自动进入演示模式。

## 本地运行

摄像头权限要求安全上下文（`localhost` 或 HTTPS），直接双击 `file://` 打开在 Safari 下无法调用摄像头。

```bash
cd noel-gesture-tree
python3 -m http.server 8123
# 打开 http://localhost:8123
```

## 技术栈

| 技术 | 用途 |
|---|---|
| Three.js r160 | 场景 / InstancedMesh / PMREM |
| WebGL + GLSL | 粒子着色器（GPU 位移，零 CPU 开销） |
| EffectComposer + UnrealBloom + OutputPass | 电影感辉光，ACES tone mapping |
| MediaPipe Hands | 21 点手部关键点 |
| 原生 CSS | 面板 / 暗角 / 胶片噪点 |

## 实现要点

- **环境光照**：自定义渐变 equirect canvas（金顶→绿中→黑底 + 两块柔光灯）经 PMREM 生成环境贴图，金属材质才有"金碧辉煌"的反射
- **坐标系**：照片挂在 `treeGroup` 下，放大时需用 `parentQuaternion⁻¹ × worldTarget` 把相机前方目标点转回父级局部空间
- **朝向**：普通 `Object3D` 的 `Matrix4.lookAt` 需传 `eye=目标点 / target=自身位置` 才能让 +Z 朝外
- **粒子尺寸**：`gl_PointSize = aSize × pixelRatio × (140 / -mvPosition.z)`，系数过大会糊成大饼遮挡视野

## License

MIT
