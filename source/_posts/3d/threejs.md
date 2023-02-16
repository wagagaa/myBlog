---
title: threejs记录
date: 2023-02-15 21:58:34
---

问题记录

## 多模型移动

```js

const controls = new DragControls( objects, camera, renderer.domElement );
this.dragControls.transformGroup = true

```

+ ransformGroup属性，当 数组objects 内只有一个THREE.Group的时，会把整个group拖动。
另外，如果想要拖动的mesh，已经在其他不同group中，可以用 .attach()解决， 也就是说 先把要拖动的模型放入同一个待拖拽的group中，等拖动完成后，再把模型用同样的方法放回原来的parent中。
