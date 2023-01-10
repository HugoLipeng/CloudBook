# 由浅入深，详解 Lifecycle 生命周期组件的那些事

![undraw_Photo_session_re_c0cp](../Downloads/202211222342022.png)

Hi,你好:)

## 引言

在2022的今天，`AndroidX` 普遍的情况下，**JetPack Lifecycle** 也早已经成为了开发中的基础设施，小到 `View(扩展库)` ,大到 `Activity`，都隐藏着它的身影，而了解 `Lifecycle` 也正是理解 `JetPack` 组件系列库生命感知设计的基础。

> 本篇定位中级，将从背景到源码实现，从而理解其背后的设计思想。

## 导航

看完本篇，你将会搞清以下问题：

- `Lifecycle` 的前世今生；
- `Lifecycle` 可以干什么；
- `Lifecycle` 源码解析；

## 背景

在开始前，我们先聊聊关于 `Lifecycle` 的前世今生，从而便于我们更好的理解 `Lifecycle` 存在的意义。

### 洪荒之时

在 `Lifecycle` 之前(不排除现在😂)，如果我们要在某个生命周期去执行一些操作时，经常会在Act或者Fragment写很多模版代码，如下两个示例：

1. 比如，有一个定时器，我们需要在 `Activity` 关闭时将其关闭，从而避免因此导致的内存问题，所以我们自然会在 `onDestory()` 中去stop一下。这些事看起来似乎不麻烦，但如果是一个重复多处使用的代码，细心的开发者会将其单独抽离成为一个 **case** ,从而通过组合的方式降低我们主类中的逻辑，但不可避免我们依然还要存在好多模版代码，因为每次都需要 `onStop()` 清理或者其他操作(除非写到base,但不可接受☹️)。

**📌 如果能不需要开发者自己手动，该有多好？**



2. 在老版本的友盟中，我们经常甚至需要在基类的 `Activity` 中复写 `onResume()` 、`onPause()` 等方法，这些事情说麻烦也不麻烦，但总是感觉很不舒服。不过，有经验的开发者肯定会想喷，为什么一个三方库，你就自己不会通过`application.registerActivityLifecycleCallbacks` 监听吗🤌🏼。

**📌 注意，Application有监听全局Act生命周期的方法，Act也有这个方法。🤖**

### 盘古开天

在 **JetPack** 之前，`Android` 一直秉承着传统的 **MVC** 架构，即 `xml` 作为 **View**, `Activity` 作为 **Control** ，`Model` 层由数据模型或者仓库而定。虽然说官方曾经有一个MVP的示例,但真正用的人并不多。再加上官方一直也没推荐过 `Android` 的架构指南，这就导致传统的Android开发方式和系统的碎片化一样☹️，五花八门。随着时间的推移，眼看前端的MVVM已愈加成熟，后有Flutter，再加上开发者的需求等背景下，Google于2017年发布了新的开发架构: **AAC**,全名 **Architecture**，并且伴随着一系列相关组件，从而帮助开发者提高开发体验，降低错误概率，减少模版代码。

而本篇的主题 `Lifecycle` 正是其中作为基础设施的存在，在 **sdk26** 之后，更是被写入了基础库中。

**那Lifecycle到底是干什么的呢？**

> `Lifecycle` 做的事情很简单，其就是用于检测组件(`Fragment`、`Act`) 的生命周期，从而不必强依赖于 `Activity` 与 `Fragment` ,帮助开发者降低模版代码。

## 常见用法

在官网中，对于 `Lifecycle` 的整个流程如下所示：

![生命周期状态示意图](https://developer.android.com/static/images/topic/libraries/architecture/lifecycle-states.svg?hl=zh-cn)

### Api介绍

**相关字段**

- **Event**

  生命周期事件，对应具体的生命周期:

  `ON_CREATE`, `ON_START`, `ON_RESUME`, `ON_PAUSE`, `ON_STOP`, `ON_DESTROY`, `ON_ANY`

- **State**

  生命周期状态节点，与 **Event** 紧密关联，**Event** 是这些结点的具体边界,换句话说，**State** 代表了这一时刻的具体状态,其相当于一个范围集，在这个范围里，都是这个状态。而Event代表这一状态集里面触发的具体事件是什么。

  - `INITIALIZED`

    构造函数初始化时，且未收到 `onCreate()` 事件时的状态;

  - `CREATED`

    在 `onCreate()` 调用之后，`onStop()` 调用之前的状态;

  - `STARTED`

    在 `onStart()` 调用之后，`onPause()` 调用之前的状态;

  - `RESUMED`

    在 `onResume()` 调用时的状态;
    
  - `DESTROYED`
  
    `onDestory()` 调用时的状态;

**相关接口**

- **LifecycleObserver**

  基础 `Lifecycler` 实现接口,一般调用不到,可用于自定义的组件，从而避免在 `Act` 或者 `Fragment` 中的模版代码,例如`ViewTreeLifecyleOwner` ；

- **LifecycleEventObserver**

  可以接受所有生命周期变化的接口；

- **FullLifecycleObserver**

  用于监听生命周期变化的接口，提供了 `onCreate()`、`onStart()`、`onPause()`、`onStop()`、`onDestory()`、`onAny()`;

- **DefaultLifecycleObserver**

  `FullLifecycleObserver` 的默认实现版本,相比前者，增加了默认 `null` 实现；

### 举个栗子

如下所示，通过实现 `DefaultLifecycleObserver` 接口，我们可以在中重写我们想要监听的生命周期方法，从而将业务代码从主类中拆离出来，且更便于复用。最后在相关类中直接使用 `lifecycle.addObserver()` 方法添加实例即可，这也是google推荐的用法。

![image-20221124221438346](https://raw.githubusercontent.com/Petterpx/ImageRespoisty/main/img/202211242214425.png)

> 上述示例中，我们使用了 `viewLifecycle` ,而不是 `lifecycle` ,这里顺便提一下。
>
> 见名之意，前者是视图(view)生命周期，后者则是非视图的生命周期，具体区别如下：
>
> `viewLifecycle` 只会在 **onCreateView-onDestroyView** 之间有效。
>
> `lifecycle` 则是代表 Fragment 的生命周期,在视图未创建时，`onCreate()`,`onDestory()` 时也会被调用到。

---

或者你有某个自定义View,想感知Fragment或者Act的生命周期，从而做一些事情，比如Banner组件等，与上面示例类似：

![image-20221124221449371](https://raw.githubusercontent.com/Petterpx/ImageRespoisty/main/img/202211242214391.png)

> 当然你也可以选择依赖：**androidx.lifecycle:lifecycle-runtime** 扩展库。
>
> 从而使用 `view.findViewTreeLifecycleOwner()` 的扩展函数获得一个 `LifecycleOwner`,从而在View内部自行监听生命周期,免除在Activity手动添加观察者的模版代码。
>
> ~~lifecycle.addObserver(view)~~

## 源码解析

### Lifecycle

> 在官方的解释里，Lifecycle 是一个类，用于存储有关组件(Act或Fragment)声明周期状态新的类，并允许其他对象观察此类。

直接去看 `Lifecycle` 的源码，其实现方式如下：

![image-20221124221501409](https://raw.githubusercontent.com/Petterpx/ImageRespoisty/main/img/202211242215446.png)

总体设计如上所示,比较简单,就是观察者模式的接口模版：

> 使用者实现 `LifecycleObserver` 接口(),然后调用 **addObserver()** 添加到观察者列表，取消观察者时调用 **rmeoveObserver()** 移除掉即可。在相应的生命周期变动时，遍历观察者列表，然后通知实现了 `LifecycleObserver` 的实例，从而调用相应的方法。

因为其是一个抽象类，所以我们调用的一般都是它的具体实现类，也就是 `LifecycleRegistry` ,目前也是其的唯一实现类。

---

### LifecycleRegistry

`Lifecycle` 的具体实现者,如名所示，主要用于管理当前订阅的 观察者对象 ,所以也承担了 `Lifecycle` 具体的实现逻辑。因为源码较长,所以我们做了一些删减，只需关注主流程即可，伪代码如下：

```java
public class LifecycleRegistry extends Lifecycle {
	
  	// 生命周期观察者map,LifecycleObserver是观察者接口,ObserverWithState具体的状态分发的包装类
    private FastSafeIterableMap<LifecycleObserver, ObserverWithState> mObserverMap;
  	// 当前生命周期状态
    private State mState = INITIALIZED;
  	// 持有生命周期的提供商,Activity或者Fragment的弱引用
    private final WeakReference<LifecycleOwner> mLifecycleOwner;
  	// 当前正在添加的观察者数量,默认为0,超过0则认为多线程调用
    private int mAddingObserverCounter = 0;
  	// 是否正在分发事件
    private boolean mHandlingEvent = false;
  	// 是否有新的事件产生
    private boolean mNewEventOccurred = false;
  	// 存储主类的事件state
    private ArrayList<State> mParentStates = new ArrayList<>();

    @Override
    public void addObserver(@NonNull LifecycleObserver observer) {
        // 初始化状态,destory or init
        State initialState = mState == DESTROYED ? DESTROYED : INITIALIZED;
      	// 📌 初始化实际分发状态的包装类
        ObserverWithState statefulObserver = new ObserverWithState(observer, initialState);
      	// 将观察者添加到具体的map中,如果已经存在了则返回之前的,否则创建新的添加到map中
        ObserverWithState previous = mObserverMap.putIfAbsent(observer, statefulObserver);
				// 如果上一步添加成功了,putIfAbsent会返回null
        if (previous != null) {
            return;
        }
      	// 如果act或者ff被回收了,直接return
        LifecycleOwner lifecycleOwner = mLifecycleOwner.get();
        if (lifecycleOwner == null) {
            return;
        }
				// 当前添加的观察者数量!=0||正在处理事件
        boolean isReentrance = mAddingObserverCounter != 0 || mHandlingEvent;
      	// 📌 取得观察者当前的状态
        State targetState = calculateTargetState(observer);
        mAddingObserverCounter++;
      	// 📌 如果当前观察者状态小于当前生命周期所在状态&&这个观察者已经被存到了观察者列表中
        while ((statefulObserver.mState.compareTo(targetState) < 0
                && mObserverMap.contains(observer))) {
          	// 保存当前的生命周期状态
            pushParentState(statefulObserver.mState);
          	// 返回当前生命周期状态对应的接下来的事件序列
            final Event event = Event.upFrom(statefulObserver.mState);
            ...
            // 分发事件
            statefulObserver.dispatchEvent(lifecycleOwner, event);
          	// 移除当前的生命周期状态
            popParentState();
            // 再次获得当前的状态,以便继续执行
            targetState = calculateTargetState(observer);
        }
				
      	// 处理一遍事件,保证事件同步
        if (!isReentrance) {
            sync();
        }
      	// 回归默认值
        mAddingObserverCounter--;
    }
		...
    static class ObserverWithState {
        State mState;
        LifecycleEventObserver mLifecycleObserver;

        ObserverWithState(LifecycleObserver observer, State initialState) {
          	// 初始化事件观察者
            mLifecycleObserver = Lifecycling.lifecycleEventObserver(observer);
            mState = initialState;
        }

        void dispatchEvent(LifecycleOwner owner, Event event) {
            State newState = event.getTargetState();
            mState = min(mState, newState);
            mLifecycleObserver.onStateChanged(owner, event);
            mState = newState;
        }
    }
  	...
}
```

我们重点关注的是 `addObserver()` 即订阅生命周期变更时的逻辑,具体如下：

1. 先初始化当前观察者的状态，默认两种，即 `DESTROYED`(销毁) 或者 `INITIALIZED`(无效)，分别对应 `onDestory()` 和 `onCreate()` 之前；
2. 初始化 **状态观察者(ObserverWithState)**，内部使用 `Lifecycling.lifecycleEventObserver()` 将我们传递进来的 **生命周期观察者(LifecycleObser)** 包装为一个 **生命周期[事件]观察者LifecycleEventObserver**，从而在状态变更时触发事件通知；
3. 将第二步生成的状态观察者添加到缓存map中,如果之前已经存在，则停止接下来的操作,否则继续初始化；
4. 调用 `calculateTargetState()` 获得当前真正的状态。
5. 开始事件轮训，如果 **当前观察者的状态小于此时真正的状态** && **观察者已经被添加到了缓存列表** 中，则获得当前观察者下一个状态，并触发相应的事件通知 `dispatchEvent()`,然后继续轮训。直到不满足判断条件；

> 需要注意的是, 关于状态的判断,这里使用了**compareTo()**,从而判断当前状态枚举是否小于指定状态。

---



### Activity中的实现

**Tips:** 

写过权限检测库的小伙伴，应该很熟悉，为了避免在 `Activity` 中手动实现 `onActivityRequest()` ,从而实现以回调的方式获得权限结果，我们往往会使用一个透明的 `Fragment` ,从而将模版方法拆离到单独类中，而这种实现方式正是组合的思想。

而 `Lifecycle` 在 **Activity** 中的实现正是上述的方式。

---

如下所示，当我们在 `Activity` 中调用 `lifecycle` 对象时,内部实际上是调用了 **ComponentActivity.mLifecycleRegistry**,具体逻辑如下：

![image-20221124221515122](https://raw.githubusercontent.com/Petterpx/ImageRespoisty/main/img/202211242215162.png)

不难发现，在我们的 `Activity` 初始化时,相应的 `LifecycleRegistry` 已经被初始化。

在上面我们说过，为了避免对基类的入侵，我们一般会用组合的方式，所以这里的 `ReportFragment` 正是 `Lifecycle` 在 `Activity` 中具体的逻辑承载方,具体逻辑如下：

**ReportFragment.injectIfNeededIn**

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3d1468d2af65413ba589f9337f72d9af~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp?)

------



内部会对sdk进行判断，对应着两套流程，对于 **sdk>=29** 的,通过注册 `Activity.registerActivityLifecycleCallbacks()` 事件实现监听，对于 **sdk<29** 的,重写 `Fragment` 相应的生命周期方法完成。



**ReportFragment 具体逻辑如下：**

![image-20221124221612104](https://raw.githubusercontent.com/Petterpx/ImageRespoisty/main/img/202211242216146.png)



---

### Fragment中的实现

直接去 Fragment.`lifecycle` 中看一下即可,伪代码如下:

![image-20221123105600533](https://raw.githubusercontent.com/Petterpx/ImageRespoisty/main/img/202211231056561.png)

总结如下：`lifecycle` 实例会在 **Fragment** 构造函数 中进行初始化,而 `mViewLifecycleOwner` 会在 **performCreateView()** 执行时初始化,然后在相应的 **performXxx** 生命周期执行时,调用相应的 `lifecycle.handleLifecycleEvent()` 从而完成事件的通知。

## 总结

`Lifecycle` 作为 `JetPack` 的基石，而理解其是我们贯穿相应生命周期的关键所在。

关于生命周期的通知，`Lifecycle` 并没有采用直接通知的方式,而是采用了 **Event(事件)** + **State(状态)** 的设计方式。

> - 对于外部生命周期订阅者而言，只需要关注事件 `Event` 的调用；
> - 而对于`Lifecycle`而言,其内部只关注 `State` ，并将生命周期划分为了多个阶段,每个状态又代表了一个事件集，从而对于外部调用者而言，无需拘泥于当前具体的状态是什么。

在具体的实现底层上面: 

> - 在 `Activity` 中，采用组合的方式而非继承，在 **Activity.onCreate()** 触发时，初始化了一个透明`Fragment`,从而将逻辑存于其中。对于**sdk>=29**的版本，因为 `Activity` 本身有监听生命周期的方法,从而直接注册监听器，并在相应的会调用触发生命周期事件更新;对于**sdk<29**的版本，因为需要兼容旧版本,所以重写了 `Fragment` 相应的生命周期方法，从而实现手动触发更新我们的生命周期观察者。
> - 在 `Fragment` 中,会在 `Fragment` 构造函数中初始化相应的 `Lifecycle` ,并重写相应的生命周期方法，从而触发事件通知,实现生命周期观察者的更新。

每当我们调用 `addObserver()` 添加新的观察者时：

> 内部都会对我们的观察者进行包装，并将其包装为一个具体的事件观察者 `LifecycleEventObserver`,以及生成当前的观察者对应的状态实例(内部持有`LifecycleEventObserver`)，并将其保存到 **缓存map** 中。接着会去对比当前的 **观察者的状态** 和 **lifecycle此时实际状态** ,如果 **当前观察者状态<lifecycle对应的状态** ，则触发相应 `Event` 的通知,并 **更新此观察者对应的状态** ，不断轮训，直到当前观察者状态 >= `lifecycle` 对应状态。

## 参阅

- [Android-使用生命周期感知型组件处理生命周期](https://developer.android.com/topic/libraries/architecture/lifecycle?hl=zh-cn#additional-resources)



## 关于我

我是 *Petterp* ,一个 Android工程师 ，如果本文对你有所帮助，欢迎点赞支持，你的支持是我持续创作的最大鼓励！



