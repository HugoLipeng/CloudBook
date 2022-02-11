# Gradle基础|自定义插件并上传到JitPack

![gradle](https://tva1.sinaimg.cn/large/008i3skNly1gz5u056f47j30zk0hs0ta.jpg)

> 「**2022 年什么会火？什么该学？本文正在参与[“聊聊 2022 技术趋势”](https://juejin.cn/post/7049621519219195918)征文活动** 」



Hi👋🏻 ，很高兴见到你！

开发两年了，我想认真学一下Gradle,这是我的2022技术进阶计划，Gradle系列的第二篇，希望对你有所帮助。

## 引言

每一个使用 `Gradle` 的同学，肯定都听过或者写过插件，因为其本身并不难，但碍于现在网上的文章千篇一律，大部分都比较老，新同学一上手反而是和我一样，花了大把时间在最基础的第一步如何写一个简单demo上。再者如果大家使用 **AndroidStudio BumBlebee** 去创建项目，那对照网上教程差别更大，甚是花费时间，而本篇就是帮你省掉这些时间。

本篇主要概括创建插件的三种方式，并如何上传到 `JitPack` 中。

> - 开发环境基于最新的 **Gradle7.0.4** , **AndroidStudio BumBlebee** ；
> - 本文相关示例代码,[github](https://github.com/Petterpx/GradlePluginSImple)

## 

## 什么是插件？

在 `Gradle` 中，插件相当于打包了可重用的一些构建片段，使其可复用为多个项目去构建。如下所示：

```groovy
// 新版写法
plugins {
    id 'com.android.application'
}
// 旧版写法
apply plugin: 'com.android.library'
```

上述就是我们最常见的两个插件，比如当我们在创建一个 **android-model** 时，就会自动添加相应的 `library` 插件，这些插件的工作就是帮我们把一些重复的工作或者代码，以一句代码的形式引入，极大程度上减少了我们的代码量。

在 `Gradle` 中，我们可以使用 `Java` ，`Kotlin` 以及 `Groovy` 来写自己的插件，一般而言，使用 `Java` 和 `Koltin` 要比使用 `Groovy` 的执行效果会更好。题外话: 写法上，`Java` 与 `kotlin` 也更符合开发习惯。

## 

## 插件的用途有哪些？

插件的作用就是添加我们自己的一些逻辑到项目执行过程中，这个做法在 `Gradle` 中称其为任务，或者说 `Task` ，从而对项目进行测试、编译、打包等;

也可以对项目中现有的对象类型添加新的扩展属性、方法等、也可以配置和优化项目的构建，比如常见的 `android{}` 就是 `Android Gradle` 插件为 `Project` 对象添加的一个扩展。

日常开发中，我们还有很多插件会在开发中见到，比如 `didibooster` 的插件，阿里路由插件，一些第三方的打点插件等。

**有一个比较有意思的问题，我觉得你可能会有？** 

> `这些插件一般还要在model中再依赖其他组件，如果我只用代码组件，而不启用这些插件，那还能正常使用吗？`
>
> 其实一般情况下，不影响你在开发中正常使用，一个`合格`的三方库，在插件没启用时也不会影响最终的使用效果，无非就是最终的实现方式上会有所差别，比如性能上。类似阿里路由插件，如果不启用插件，只依赖代码组件依赖，则在最终找路由表时就只能通过反射去找，而不是通过编译期间生成的路径映射，所以一般我们在debug下可以关闭某些依赖，从而减少debug时间，不过一般而言，这些插件所花费的时间并不多，所以根据自身需求而定。

## 

## 创建插件方式

### 脚本插件

我们可以直接在构建脚本中包含插件的源代码，这种是最简单易懂的一种方式，具体示例如下：

![image-20220208084346582](https://tva1.sinaimg.cn/large/008i3skNly1gz5t8obt06j31y20ma78w.jpg)

直接在 **app model** 中写插件，这样做的好处就是插件会自动编译并包含在构建脚本的类路径中，而无需做其他操作。相应的，如果要跨项目复用，就比较难解决，而且因为缺少统一的维护路径，也增加了后期成本。

---

### buildSrc

官方建议我们可以将本地插件的代码放到 `buildSrc` 这个目录中。这个目录比较特殊，对于每一个工程而言，有且只能有一个 `buildSrc` 目录，并且必须位于项目的根目录，如果存在 `buildSrc` 这个目录，那么 `Gradle` 在运行时会自动编译并测试这里面的代码，并将其放入构建脚本的类路径中，与上述的脚本插件相比，其更易于维护和管理测试。	

> 示例代码：[buildSrc](https://github.com/Petterpx/GradlePluginSImple/tree/main/buildSrc)

使用教程:

我们新建一个名为 `buildSrc` 的目录，然后直接创建一个 **build.gradle** 文件，如下所示:

![image-20220130115339054](https://tva1.sinaimg.cn/large/008i3skNly1gyvk5f6hysj31m40h0q5d.jpg)

代码如下：

```groovy
apply plugin: 'kotlin'
buildscript {
    repositories {
        google()
        mavenCentral()
    }
    dependencies {
        // 因为想使用kotlin，所以这里增加kotlin插件
        classpath "org.jetbrains.kotlin:kotlin-gradle-plugin:1.6.10-RC"
    }
}

repositories {
    google()
    mavenCentral()
}

dependencies {
    implementation "org.jetbrains.kotlin:kotlin-stdlib-jdk8:1.6.10"
}
```

接着 `sync` 一下，然后你就会发现，这个 **buildSrc** 已经被 Android Studio 自动识别为了一个 **java** 项目。如下：

![image-20220130120232167](https://tva1.sinaimg.cn/large/008i3skNly1gyvkenlnnej31gi0akq4m.jpg)

> 当然你也可以直接创建一个 `java-model`,然后去更改 `build` 文件,这样的好处是省去了手动创建文件夹的功夫，相应的，你也需要去`settings.gradle` 去删除这个model的声明，原因是：**buildSrc是一个特殊的目录，禁止手动声明**；

到了这里，那我们如何创建自己的插件文件呢，直接在 `src` 下创建相应的文件夹即可，然后创建我们自己的插件文件，如下所示：

![image-20220130121509594](https://tva1.sinaimg.cn/large/008i3skNly1gyvkrso77hj31za0oedk1.jpg)

上面的这个目录格式，根据你自己的写法而定，比如这里我想使用 `kotlin` 去写插件代码，就使用如下，默认官方推荐了三种目录配置写法：

- src/main/kotlin
- src/main/java
- src/main/groovy

> ps：**当使用As在buildSrc创建目录时，会自动提示选择合适的目录。**

上述我们创建了自定义的插件实现类，现在就去改一下我们的 `build.gradle` 文件，增加下述代码：

```groovy
//java-gradle插件
apply plugin: 'java-gradle-plugin'
...

//本地依赖插件时使用
gradlePlugin {
    plugins {
        //插件名,每一个插件都可以有
        buildSrcTestPlugin {
        		// 你的插件id，外部项目引用时需要
            id = 'com.petterp.gradle.buildSrc'
            // 插件的具体实现类
            implementationClass = 'com.petterp.gradle.BuildSrcGradlePlugin'
        }
      	//第二个插件
//        test2Plugin {
//            id = xxx
//            implementationClass = xxx
//        }
    }
}
```

改完之后，然后在我们的app-build.gradle(具体取决于你的使用位置)中进行依赖引入即可：

![image-20220130122223386](https://tva1.sinaimg.cn/large/008i3skNly1gyvkzbpkyfj32ko0i8n1c.jpg)

到这里就结束了，是不是超简单，如果你搜过老的教程，你会发现你需要 **手动创建resource即META-xxx等等** 。

其实并不是我们没有创建，而是我们使用的 `java-gradle-plugin` 插件会自动创建，并且以 `api` 的方式引入 **gradleApi()** ,会自动帮我们将上述的步骤实现。具体最终生成在这里，如下图所示：

<img src="https://tva1.sinaimg.cn/large/008i3skNly1gyvl34b4mjj30q60ioq3r.jpg" alt="image-20220130122602719" style="zoom:33%;" />

---

### Standalone project

上面的两种方式只能在当前项目中进行使用，如果我们想要在其他项目中使用，此时一般我们会将插件发布到 `Maven` 上，与其他人进行共享，这也是开发插件最常用的一种方式了,本篇会上传到 `JitPack` 上。

> 示例代码：[standlone](https://github.com/Petterpx/GradlePluginSImple/tree/main/stand-gradle-plugin)

我们复制粘贴上面教程里的 **buildSrc** 包，并对其进行改名如下，比如改为 `stand-gradle-plugin` ,然后在我们的项目 `settings.build` 中引入此model，如下所示：

<img src="https://tva1.sinaimg.cn/large/008i3skNly1gyvosw0au3j31xi0u00xk.jpg" alt="image-20220130143436155" style="zoom:50%;" />

看着好像过于简单了点，也没啥不同，你可能会想，那这样的话，那 `buildSrc` 的用处是啥？反正似乎啥名字都可以啊？

我们先改一下 **stand-gradle-plugin** 的 `插件id` ，及相应的 `插件实现类类名` ，如下所示：

![image-20220130143837496](https://tva1.sinaimg.cn/large/008i3skNly1gyvox2pcvqj321u0hs0v8.jpg)

然后直接去app-model中进行引入，此时会发生什么问题呢？

<img src="https://tva1.sinaimg.cn/large/008i3skNly1gyvp0fll5ij31qe0u078t.jpg" alt="image-20220130144151276" style="zoom:50%;" />

提示找不到这个插件，为什么？我不是在 `settings.gradle` 中引入了吗？

我们在最上面说过了，`buildSrc` 本身是一个特殊的项目，**Gradle** 会自动编译并引入。而其他插件，如果没有采用 `maven` 的方式引入，就需要单独配置了，如下所示：

我们更改一下 `settings.gradle` 文件：

```groovy
includeBuild('stand-gradle-plugin')
```

更改 `项目根build.gradle`:

```groovy
plugins {
    ...
    id 'com.petterp.gradle.stand' apply false
}
```

重新sync就不会报错了，结果如下图所示：

<img src="https://tva1.sinaimg.cn/large/008i3skNly1gyvpmh4alxj31800akaav.jpg" alt="image-20220130150302326"  />

这其实就相当于我们使用了本地的一个插件，如果你使用过复合构建去配置过你的项目依赖配置，可能对这种方式就非常熟悉了。

## 上传到JitPack中

一般而言，我们会将插件上传到 `Maven` 上，便于跨项目使用。我们以 `Standalone Project` 这种方式为例，更改 **Standalone** 方式中的`build.gradle`，如下所示,增加下列代码：

> 示例代码：[stabdlobe-build.gradle](https://github.com/Petterpx/GradlePluginSImple/blob/main/stand-gradle-plugin/build.gradle)

```groovy
apply plugin: 'maven-publish'

// 组名，可以理解为插件在那个分组下放着，最终是一个文件夹
// com/petterp/gradle/plugins/xxx
group = 'com.petterp.gradle'
// 描述
description = '这是一个独立插件'
// 版本号
version = '1.0.0'

sourceCompatibility = JavaVersion.VERSION_11
publishing {
    publications {
        maven(MavenPublication) {
          	// 版本id，最终会根据这个id-version生成相应的插件
            artifactId = 'com.petterp.gradle.plugin'
            from components.java
        }
    }
   	
    repositories {
        maven {
            // 生成的插件位置
            url = uri('../repo')
        }
    }
}
```

更改settings.gradle中插件依赖的方式为

```groovy
include ':stand-gradle-plugin'
```

然后在命令行执行：`gradlew publish`

此时我们的项目里就会多出一个 `repo` 的文件夹，如下图所示，这就是我们最终打包好的插件包。

![image-20220130170438979](https://tva1.sinaimg.cn/large/008i3skNly1gyvt502535j31im0c0q4v.jpg)

接下来去 github 打 Tag,并打开 [Jitpack](https://jitpack.io/) 网站，搜索我们的项目名称，进行构建。

![image-20220208085127886](https://tva1.sinaimg.cn/large/008i3skNly1gz5tgmo62oj30y60egdgt.jpg)

完成后我们就可以在项目中进行引用了，如下图所示：

![image-20220207115147019](https://tva1.sinaimg.cn/large/008i3skNly1gz4t1z9mkwj318w0du75f.jpg)

因为我们生成的是插件，所以相应的依赖方式改为 `classpath`

```groovy
classpath "com.github.Petterpx:GradlePluginSImple:1.0.0"
```

然后在相应的 model 里引入即可

```groovy
plugins {
    ...
    id 'com.petterp.gradle.stand'
}
```

这里为止，一个简单的插件就算创建完成了。

## 参考

- [Gradle文档-开发自定义Gradle插件](https://docs.gradle.org/current/userguide/custom_plugins.html)
- [Android文档-使用 Maven Publish 插件](https://developer.android.com/studio/build/maven-publish-plugin?hl=zh-cn)
- [自定义Android Gradle插件(Kotlin)](https://apon.me/2021/02/24/%E8%87%AA%E5%AE%9A%E4%B9%89Android-Gradle%E6%8F%92%E4%BB%B6-Kotlin-%E4%B9%8B%E4%B8%80-Hello-World/)

## 相关文章

往期 `Gradle` 相关文章如下：

- [哪怕不学Gradle,这些常见操作，你也值得掌握](https://juejin.cn/post/7053985196906905636)
- [Gradle基础|自定义插件并上传到JitPack](https://juejin.cn/post/7062141046255255583)



> 我是Petterp,一个三流开发，如果本文对你有所帮助，欢迎点赞支持。