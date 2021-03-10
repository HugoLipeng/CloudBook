# ViewPager2实现内部Item的动态滚动



## 需求决定起因

最近接到了一个需求，大概类似如下图所示的一个样式(省略了部分细节，不影响大概)。

<img src="https://tva1.sinaimg.cn/large/008eGmZEly1gmxfokv1z8j312k0u0jtz.jpg" alt="image-20210123104608972" style="zoom: 50%;" />

我们这是一个视频播放页+详情页，考虑到简单快捷，就想到了一个 `ViewPager2` 就可以实现，简单又快捷，为自己点赞。一想到如此easy,瞬时笑出了猪叫。当然RecyclerView也可以，用一个仿抖音的那种 `LayoutManager` 就行，但是为什么不呢，因为涉及到了视频播放，手动去处理一些生命周期和懒加载，总是非常麻烦，而且ViewPager2本身就是基于 `RecyclerView` ,所以何乐而不为呢。

当然有些同学会说了，这个玩意自定义一个可滑动的ViewGroup就行啊，这个方案也可以。但是首先你要考虑的东西就很多，如果视频详情页超出一屏呢，也就是内部用了 **RecyclerView或者NestedScrollView** 呢，是不是还需要处理一下滑动冲突，当然这也不是很困难,内部拦截法就可以搞定。然后写完后，相应的加载回调是不是得自己再手动定义一个接口去伪造。比如不可见，页面加载，总体相对来说并不是那么容易。



就在我以为又可以摸鱼一个ViewPager2就可以搞定之时。突然，产品同学发了新指示，下意识预感不妙。

产品：得加一个第一次使用时的提示啊，要不然用户都不知道页面可以下滑呢？效果我发你了，你看看：

> 下图为我实现好的简单样式，大意体会即可。

<img src="https://tva1.sinaimg.cn/large/008eGmZEly1gmxg4c6ovrg30c00om1kz.gif" alt="Jan-23-2021 11-00-49" style="zoom:50%;" />

好家伙，不按套路出牌啊，我故作深沉，实则稳如老狗( ViewPager2 不是有一个 `fakeDragBy()` 方法设置偏移量吗)，这个有点麻烦，我得考虑考虑。

接下来不却知道自己要开启了啪啪打脸时刻，满心欢喜，太easy啊，ViewPager2 真香🤣！



## 打脸时刻

于是熟练的开分支，切分支，写demo，调用方法，走起！

先看一下这个方法。

**fakeDragBy()**

> 用于模拟手指拖动效果，需要先调用 `beginFakeDrag()` 开启，结束后，需要调用 `endFakeDrag()` 关闭。

既然有这个方法，那不就很简单吗，伪代码如下：

![image-20210123113905769](https://tva1.sinaimg.cn/large/008eGmZEly1gmxh7mag2rj31cc0fon0v.jpg)

查看效果如下：

|                             示例                             |                                                              |
| :----------------------------------------------------------: | :----------------------------------------------------------- |
| <img src="https://tva1.sinaimg.cn/large/008eGmZEly1gmxgzd46hmg30c00omhdv.gif" alt="Jan-23-2021 11-30-53" style="zoom:50%;" /> | ![我他妈直接裂开_直接_裂开表情](https://tva1.sinaimg.cn/large/008eGmZEly1gmxh0acs0aj30c80c774o.jpg) |

我裂开了😨，为什么会这样，我就属性动画里调了一下而已，去看一下源码。

**ViewPager2.fakeDragBy(x)**

![image-20210123113813708](https://tva1.sinaimg.cn/large/008eGmZEly1gmxh6q61v2j31ai0r647v.jpg)

内部最终是调用了RecyclerView的 `scrollBy()` ，也就是相对滑动,哦原来如此，难怪调了一下，滑了这么远。

## 解决方法 

既然如此，ViewPager2是基于RecyclerView，那么我去调用RecyclerView滚动不就行吗，思路如下：

1. ViewPager2-> RecyclerView, RecyclerView默认是私有的，可以通过`反射`或者 **getChildAt(0)** 获取
2. RecyclerView不支持 **scrollTo()** ,可以通过 `LinearLayouManager` 去滚动
3. LinearLayoutManager-`scrollToPositionWithOffset()` 支持滚动到偏移位置

伪代码如下：

```kotlin
val layoutManager = (getChildAt(0) as? RecyclerView
        ?: return@apply).layoutManager as? LinearLayoutManager ?: return@apply
val oneAnimator = ValueAnimator.ofFloat(0f, -450f).apply {
    duration = 500
}
oneAnimator.addUpdateListener {
    layoutManager.scrollToPositionWithOffset(0, it.animatedValue as Int)
}
oneAnimator.start()
```

效果如最上面示例gif所示，这样就解决了ViewPager2-item动态滚动问题。



## 需要注意的点

就如我上面最开始分析时所述，如果详情页是可滑动的，那么就必须处理一下滑动冲突，相应的方式也很简单，使用内部拦截法，让滑动的View优先获得事件即可，当处于滑动View顶部时，再将事件还给父View.

![image-20210123133854397](https://tva1.sinaimg.cn/large/008eGmZEly1gmxkoagmo4j31ap0u0jzf.jpg)



## 后续

当然用ViewPager2去写仍然有种大材小用的感觉，毕竟只有两个item,所以，比较好的方式依然是使用自定义的滑动ViewGroup实现，所以我会在下篇博客来以一个自定义的方式来解决此问题。

