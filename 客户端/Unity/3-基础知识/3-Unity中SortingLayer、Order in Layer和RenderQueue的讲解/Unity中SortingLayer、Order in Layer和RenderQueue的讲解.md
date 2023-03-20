# Unity中SortingLayer、Order in Layer和RenderQueue的讲解

**我们在实际项目中经常会用到多个相机，多个Canvans,Sprite Renderer,ParticleSystem等混合使用的情况，会遇到层级渲染先后顺序的问题，本文详细解释了渲染的先后关系。**

> Camera的Depth > Canvas的Sorting Layer > Canvas的Order in Layer

## Camera的depth和Clear Flags设置

项目拆分两个相机一个主相机和UI相机

```
Clear Flags`推荐设置为`Depth only
```

`Culling Mask` 选择相应的`Layer` 主相机取消`UI` Layer 层 UI相机选择UI层

```
Depth` 值越小,先渲染。 主相机默认`-1` UI相机 `0
```

## Sorting Layer

为什么要有这个Sorting Layer呢，因为我们可以创建很多个Canvas，默认Sorting Layer是Default，这个时候，渲染顺序是根据Canvas的节点在Hierarchy窗口中的顺序来决定的，上层的节点先渲染，下层的节点后渲染。
而有时候，可能需要打破这个顺序，让上层节点的Canvas后渲染，这个时候，就可以设置这个Sorting Layer为高的值，当然，也可以保持相等，通过设置Order in Layer。

> 拓展：UGUI会自动合并批次，原理是它会把一个Canvas下的所有元素合并在一个Mesh里，如果Canvas下的元素很多，任意一个元素发生位置、大小的改变，就需要重新合并所有元素的Mesh。如果元素非常多的话，就可能会造成卡顿。
> 一个比较好的做法是每个UI界面都设置成一个Canvas。如果这个界面下的元素比较多，可以考虑嵌套多几个Canvas。尤其是会频繁改变位置大小的元素，这样可以降低它们合并Mesh的开销。但是Canvas嵌套太多也不好，Mesh合并是降低了，但是DrawCall又上去了，因为每个Canvas都会单独占用一个DrawCall。

> 注意：Layer与Sorting Layer无关，它们是两个概念。

## **Order in Layer**

`Order in Layer`顾名思义，就是`Sorting Layer`的内部排序，这样配合`Sorting Layer`就是两级的排序，可以解决大部分情况的渲染顺序需求。

当然，如果创建多个`UI`摄像机，不同`Canvas`绑定不同的`UI`摄像机，再配合摄像机的`Depth`，就是三级排序，但一般不创建太多的`UI`摄像机，除非逼不得已。

> 小结: 渲染排序级别：Camera的Depth > Canvas的Sorting Layer > Canvas的Order in Layer

## Shader的RenderQueue

Unity提供给我们一些默认的渲染队列，每一个对应一个唯一的值，来指导Unity绘制对象到屏幕上。这些内置的渲染队列被称为Background, Geometry, AlphaTest, Transparent, Overlay。这些队列不是随便创建的，它们是为了让我们更容易地编写Shader并处理实时渲染的。

| Properties   | Value | 渲染队列描述                                                 | 备注                                                         |
| ------------ | ----- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| Background   | 1000  | This render queue is rendered before any others.             | 这个队列通常被最先渲染（比如 天空盒）。                      |
| Geometry     | 2000  | Opaque geometry uses this queue.                             | 这是默认的渲染队列。它被用于绝大多数对象。不透明几何体使用该队列。 |
| AlphaTest    | 2450  | Alpha tested geometry uses this queue.                       | 需要开启透明度测试的物体。Unity5以后从Geometry队列中拆出来,因为在所有不透明物体渲染完之后再渲染会比较高效。 |
| GeometryLast | 2500  | Last render queue that is considered “opaque”                | 所有Geometry和AlphaTest队列的物体渲染完后                    |
| Transparent  | 3000  | This render queue is rendered after Geometry and AlphaTest, in back-to-front order. | 所有Geometry和AlphaTest队列的物体渲染完后，再按照从后往前的顺序进行渲染,任何使用了透明度混合的物体都应该使用该队列（例如玻璃和粒子效果） |
| Overlay      | 4000  | This render queue is meant for overlay effects.              | 该队列用于实现一些叠加效果，适合最后渲染的物体（如镜头光晕）。 |

当`RenderQueue`填-1是使用`shader`自定义的值，否则使用手动填的值。
**2500是关键值，它是透明跟不透明的分界点。**
知识点:
`RenderQueue > 2500`的物体绝对会在`RenderQueue <= 2500`的物体前面，即渲染时`RenderQueue`大的会挡住`RenderQueue`小的，不论它的`Sorting Layer`和`Order in Layer`怎么设置都是不起作用的。

当两个的`RenderQueue`都在同一侧时，在`Sorting Layer`高的绝对会在`Sorting Layer`前面，无视`RenderQueue`跟`Order in Layer`，只有在`Sorting Layer`相同的前提下，`Order in Layer`高的会在`Order in Layer`低的前面，无视`RenderQueue`。当`Sorting Layer`跟`Order in Layer`相同时，才看`RenderQueue`的高低，高的在前面。