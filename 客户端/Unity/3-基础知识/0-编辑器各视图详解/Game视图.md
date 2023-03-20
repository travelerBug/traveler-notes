| **FPS**                          | 当前 Unity 每秒能够绘制的帧数。                              |
| -------------------------------- | ------------------------------------------------------------ |
| **cpu**                          | **Main**：处理一帧所花费的总时间。这个数字包括 Unity 处理应用程序的帧更新所花费的时间，以及 Unity 在编辑器中更新场景视图、其他编辑器窗口或处理仅编辑器任务所花费的时间。 **渲染**：渲染一帧所花费的时间。这个数字包括 Unity 处理 Game 视图的帧更新所花费的时间；它不包括 Unity 在编辑器中花费的时间。 |
| **Batches**                      | 在一帧期间 Unity 处理的[绘图调用批处理](https://docs.unity.cn/cn/2021.3/Manual/DrawCallBatching.html)的总数。此数字包括静态、动态和[实例](https://docs.unity.cn/cn/2021.3/Manual/GPUInstancing.html)批次。 |
| **Saved by batching**            | Unity 合并成批次的绘图调用数。为确保良好的绘制调用批处理，请尽可能多地在不同**游戏对象之间共享材质。**批处理将具有相同渲染状态的绘制调用分组，因此更改材质会导致 Unity 创建一个新批处理。 |
| **Tris**                         | Unity 在一帧期间处理的[三角形](https://docs.unity.cn/cn/2021.3/ScriptReference/Mesh-triangles.html)数量。[在针对低端硬件进行优化](https://docs.unity.cn/cn/2021.3/Manual/OptimizingGraphicsPerformance.html)时，这个值很重要。 |
| **Verts**                        | 一帧期间 Unity 处理的[顶点数。](https://docs.unity.cn/cn/2021.3/ScriptReference/Mesh-vertices.html)[在针对低端硬件进行优化](https://docs.unity.cn/cn/2021.3/Manual/OptimizingGraphicsPerformance.html)时，这个值很重要。 |
| **Screen**                       | 屏幕的分辨率，以及屏幕使用的内存量。                         |
| **SetPass**                      | Unity 在帧期间切换它用于渲染游戏对象的着色器的次数。一个着色器可能包含多个着色器通道，每个通道以不同的方式渲染游戏对象。每个通道都需要 Unity 绑定一个新的着色器，这可能会引入 CPU 开销。 |
| **Shadow casters**               | 帧中投射阴影的游戏对象的数量。                               |
| **Visible skinned meshes**       | 帧中[蒙皮网格渲染器](https://docs.unity.cn/cn/2021.3/Manual/class-SkinnedMeshRenderer.html)的数量。 |
| **Animation components playing** | 帧期间播放的[动画](https://docs.unity.cn/cn/2021.3/Manual/class-Animation.html)组件的数量。 |
| **Animator components playing**  | 在帧中播放的[Animator](https://docs.unity.cn/cn/2021.3/Manual/class-Animator.html)组件的数量。 |