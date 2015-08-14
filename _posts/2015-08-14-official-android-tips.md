---
layout: post
title:  "官方 Android 性能优化典范总结!"
date:   2015-08-14 18:07:20
categories: android tips
---
# 官方 Android 性能优化典范总结

### 界面优化

- 16 ms 帧规定，可使用 GPU Profile 观察
- 渲染的流程： Measure -> Layout -> Record Display List -> Execute Display List
- alpha 渐变动画使用 setLayerType(View.LAYER_TYPE_HARDWARE, null) ViewPropertyAnimator.alpha.withLayer() 避免 2-pass 渲染, View.hasOverlappingRendering() {return false} 也可以
- RelativeLayout 使用 alignTop/alignBelow，LinearLayout 设置 measureWithLargestChild() 都会导致 2-pass layout, @see Double Layout Taxation


### Bitmap 优化

- 使用更小的 pixel 格式 替代 RGBA_8888，如 RGB_565 或 RGBA_4444
- 使用索引颜色的 PNG 格式 (scriptPNG 工具，Photoshop 导出WEB格式)
- in-factor-of-2 缩放 Bitmap 时可直接用 BitmapOptions.inSampleSize = 比例，例如 4 (1/4 大小)，避免OOM
- 查看资源图片大小使用 inJustDecodeBounds
- 复用 bitmap 内存可使用 inBitmap 属性 (要求大小要一致)


### 内存优化

- 内存抖动、内存溢出影响 GC 效率，导致丢帧
- Memory Profile 工具
	- Memory Monitor：查看整个app所占用的内存，以及发生GC的时刻，短时间内发生大量的GC操作是一个危险的信号。
	- Allocation Tracker：使用此工具来追踪内存的分配
	- Heap Tool：查看当前内存快照，便于对比分析哪些对象有可能是泄漏了的
- Object Pool 解决大量小对象分配导致的内存抖动
- WeakHashMap 会将内存用尽后调用特殊方法后释放内存，HashMap<key,WeakRef<value>> 会更及时释放 (Demo 验证过)


### Java 优化

- 使用 for(int i=0, c={size}; i<c; i++) 提高2倍+迭代效率
- 活用 LRU Cache (Least Recently Use)
- Profile 工具: Battery History Tool
- 需要遍历速度较高时使用 ArrayMap，减少创建对象数量
- 键类型为 primitive 时，为避免 Auto-boxing，使用 Sparse*Map
- WeakHashMap **并不是对 value** ，而是对 key 进行弱引用，不能自动回收内存，慎重使用


### 网络同步

- 后台同步网络活用 Prefetch, batch
- prefetch 例子：加载 listView 图片时，预加载数据列表内的图片，改善滑动列表时的体验
- 计算View大小，并尝试获取服务端 resized 的图片
- 使用 Gzip 压缩预优化，FlatBuffers，图片使用 WebP 或 索引颜色PNG


### 电量消耗

- 使用 JobScheduler API, AlarmManager.setInexact*()
