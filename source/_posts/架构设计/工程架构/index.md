# 工程架构

主要介绍Android App工程架构：单一工程、模块化、组件化、插件化、容器化的演进之路。

## 一、传统单一工程结构

![](DraggedImage.tiff)
![](DraggedImage.png)

![](DraggedImage-1.tiff)

### 1.1. 传统单一工程特点

1. 按照java文件类型进行分包。适合人数较少(1-3)，较小规模。
2. 模块关系：业务与业务之间强耦合，可以互相访问调用。**你中有我，我中有你**。

![](DraggedImage-2.tiff)

### 1.2. 传统单一工程的问题

1. 单一app工程，代码结构混乱，代码维护困难；
2. 单一app工程，代码量指数级膨胀，带来一系列的问题，如：代码合并，编译时间等；
	- 早期优酷：改一行代码，编译10几分钟；
	- 每次改代码，就要立即提交代码，否则就可能要解决与别人的代码冲突；非常痛苦；
3. 业务逻辑与基础功能没有做区分，紧密耦合在一起，代码复用性为0；

## 二、组件化工程结构

[ 一键搭建整体组件架构, 让您体验纯傻瓜式的组件化开发 ArmsComponent](https://github.com/JessYanCoding/ArmsComponent/wiki)
[ 字节跳动总监对Android组件化的最佳实战总结 ](https://segmentfault.com/a/1190000039092651)
[ “终于懂了” 系列：Android组件化，全面掌握 ](https://juejin.cn/post/6881116198889586701)
[ Android组件化开发思想与实践 ](https://juejin.cn/post/6844904147641171981)
[ Android组件化开发实践 ](https://www.jianshu.com/p/186fa07fc48a)
[MVVM的组件化方案](https://www.wanandroid.com/blog/show/2461)

### 2.1. 为什么要组件化？

组件化架构是为了解决如下问题：

1. 解决传统单一工程业务相互依赖的问题；
2. 无法多人并行开发的问题；合代码经常发生冲突？很烦？
3. 业务逻辑与基础功能区分开，降低耦合，提高代码的复用性；
4. 被人偷偷改了自己模块的代码？很不爽？
5. 做一个需求，发现还要去改动很多别人模块的代码？
6. 别的模块已实现的类似功能，自己要用只能去复制一份代码再改改？
7. “这个不是我负责的，我不管”，代码责任范围不明确？
8. 只做了一个模块的功能，但改动点很多，所以要完整回归测试？
9. 做了个需求，但不知不觉导致其他模块出现bug？

### 2.2. 组件化后的工程结构

与单一app工程的区别：有了分层与组件话的概念。**越往下层越通用**。分层的结构可以方便代码的扩展。

1. 宿主层：只有一个宿主app，只有一个作用，依赖下面不用的业务，然后编译生成apk；
2. 业务组件层：主要完成App各种功能；
3. 基础业务组件层：整个公司的通用UI、通用设计规范、通用业务逻辑；（通用UI怎么不放在基础组件层）
4. 基础组件层：与业务完全无关，甚至可以直接开源的组件；

![](DraggedImage-1.png)

![](DraggedImage-3.tiff)

模块化&组件化：

1. 模块指的是独立的业务模块。模块和组件最明显的区别是模块相对组件来说粒度更大。一个模块中可能包含多个组件。在划分的时候，模块化是业务导向，组件化是功能导向。组件化是建立在模块化思想上的一次演进。使得模块可以在集成和独立调试之间切换的特性。 在打包时是library，在调试时是application。
2. 模块化的特点：业务模块分离，高内聚，低耦合，代码边界清晰。有利于团队协作多线开发。加快编译速度，提高开发效率。
3. **狭义上：**指的是Android studio支持了多个module开发时，提出的模块化概念。具体实践：把常用的功能、控件、基础类、第三方库、权限等公共部分抽离封装，把业务拆分成N个模块进行独立(module)的管理。而所有的业务组件都依赖于封装的基础库，业务组件之间不做依赖，这样的目的是为了让**每个业务模块边界清晰**。
4. **广义上**： 将一个复杂业务实现，根据功能、页面或者其他进行不同粒度的划分程不同的模块，模块之间解耦，分别进行实现，也就是编程的模块化思想。
5. 壳工程负责打包环境、签名、混淆规格、业务模块集成、APP主题等配置；
6. 每个业务模块以aar的形式单独集成或下架，可以直接添加common.aar作为基础依赖。

组件化后的工程结构：

![](DraggedImage-4.tiff)

模块间关系：

1. app: 壳工程。只用于模块集成，签名，主题配置，app logo, 渠道配置…
2. biz\_xx：业务组件；
3. common：通用UI、通用业务逻辑；
4. hi\_xx：基础组件；
5. pub\_mod：用于个别业务模块间资源、entity，逻辑共享。之所以不放在common组件，是因为希望common模块更加纯粹；
6. server\_login: 暴露拉起登录，判断是否已登陆，获取登陆token等信息的接口。

![](DraggedImage-5.tiff)

### 2.3. 组件化之后的问题与解决方案

1. 不同模块之间怎么实现方法调用？
2. 模块之间如何进行通信？
3. 模块之前的javabean如何共享？
4. 公共资源文件如何共享？
5. 如何将各个模块打包成一个独立的 APP 进行调试？
6. 如何防止资源名称冲突？
7. 如何解决 library 重复依赖以及 sdk 和依赖的第三方版本号控制问题？

#### 组件间如何通信

主要是页面跳转：ARouter框架

![](DraggedImage-6.tiff)

#### 不同模块之间怎么实现方法调用？SPI

spi 机制是为了将 服务使用者 与 服务提供者 解耦，而它们之间使用 ServiceLoader 进行连接，通过类 java.util.ServiceLoader 来获取服务。服务定位即查找 jar 包的 ‘META-INFO/services’目录。 spi 本身并没提供任何接口，只是用于服务发现/获取。

![](DraggedImage-7.tiff)

```js
//1. 将这个接口打包成jar对外提供
public interface ILoginService {
    void login();
}
//2. 实现接口中的方法，并按照下图方式注册
public class LoginServiceImpl implements ILoginService{
     void login(){
       .......
     }
}
//3. 使用时根据接口类名查找具体的实现类
 public static ServiceLoader load(Class<S> service)
```

![](DraggedImage-8.tiff)

方法二：用ARouter IProvider暴露业务组件接口也可以。

#### 模块之间的javaBean,公共资源如何共享? 提取共用

![](DraggedImage-9.tiff)

#### 如何防止资源名称冲突？ gradle约束

```js
android{
    resourcePrefix 'home_'
}
```

如果想对所有module应用，并不需要一一配置。可以在项目根目录的 build.gradle统一设置，使用 module 名称 加上下划线作为 资源前缀。

```js
subprojects {
    afterEvaluate {
        android {
            resourcePrefix "${project.name}_"
        }
    }
}
```

其次：也可以将资源按照**[文件夹分类存放](http://cashow.github.io/Android-Refactoring-And-Modularization-Record.html)**，方便查找。将 res 文件夹下的所有子文件夹取出，并设为资源文件夹。通过这个配置，你就能在 res 文件夹下加各个模块的子文件夹。需要注意的是，有时候在 res 文件夹添加新的子文件夹时，Android Studio 不会将新文件夹视为资源文件夹，这时你需要将上面的代码注释掉，Sync 一遍，取消注释后再 Sync 一遍即可。

```js
def listSubFile = {
    def resFolder = 'src/main/res'
    def files = file(resFolder).listFiles()
    def folders = []
    files.each {
        item -> folders.add(item.absolutePath)
    }
    folders.add(file(resFolder))
    return folders
}
sourceSets {
    main {
        jniLibs.srcDirs = ['libs']
        res.srcDirs += listSubFile()
    }
}
```

#### 如何解决 library 重复依赖以及 sdk 和依赖的第三方版本号控制问题? 统一管理，远程依赖

```js
android{
  configurations.all{resolutionStrategy->
     force 'com.alibaba:arouter:api:1.1'
  }
}
```

```js
apply from:'../../dependencies.gradle'
ext {
    versions = [
            applicationId          : "org.devio.as.proj.main",
            versionCode            : 1,
            versionName            : "1.0.0",

            compileSdkVersion      : 28,
            minSdkVersion          : 15,
            targetSdkVersion       : 28,
            buildToolsVersion      : "29.0.0",

            constraintlayoutVersion: "1.1.3",
            appcompatVersion       : "28.0.0",

            arouterApiVersion      : "1.5.0",
            arouterCompilerVersion : "1.2.2",
    ]

    dependencies = [
            "appcompat"       : "com.android.support:appcompat-v7:${versions["appcompatVersion"]}",
            "constraintlayout": "com.android.support.constraint:constraint-layout:${versions["constraintlayoutVersion"]}",

            //阿里路由
            "arouter_api"     : "com.alibaba:arouter-api:${versions["arouterApiVersion"]}",
            "arouter_compiler": "com.alibaba:arouter-compiler:${versions["arouterCompilerVersion"]}",
    ]
}

//使用远程依赖，防止人为修改，被覆盖。方便统一升级，权限管理。
apply from:'https://api.devio.org/as/project/depdencies.gradle'
```

#### 如何将各个模块打包成一个独立的 APP 进行调试？

1. 伪需求
2. 必定会存在多个模块间互相通信的场景

```js
biz_order/build.gradle
if (isRunAlone) {
    apply plugin: 'com.android.application'
} else {
    apply plugin: 'com.android.library'
}

android{
  sourceSets {
        main {
            if (Boolean.valueOf(rootProject.ext.isModule_North)) {//apk
                manifest.srcFile 'src/main/manifest/AndroidManifest.xml'
            } else {
                manifest.srcFile 'src/main/AndroidManifest.xml'
                java {
                    //library模式下，排除java/debug文件夹下的所有文件
                    exclude '*module'
                }
            }
        }
    }
}
```

实际管理和操作起来非常麻烦，还要维护2套manifest，还需要专门为模块写主Activity页面。

### 2.4. 组件化的缺点

组件化无法解决的问题：

1. 工程编译时间相对传统单一工程结构没有得到提升，调试成本依然很高；最终所有的代码还是打包在apk中；
2. 业务组件之间依然有依赖；例如首页想要调用登录或搜索，那么必须依赖（虽然是暴露服务的方式）；如果app非常复杂，业务组件非常多，那么可能导致非常多的依赖；
3. 无法动态化发布；

## 三、插件化

参考资料：

[ 插件化技术的演进之路 ](https://juejin.cn/post/6844904166557515784)
[App动态化](https://www.jianshu.com/p/e177c374558a)
[ 深入理解Android插件化技术 ](https://zhuanlan.zhihu.com/p/33017826)
[ 搞懂插件化，看这一篇就够了 ](https://juejin.cn/post/6911942017853440007)
[ 安卓插件化与热修复的选型](https://www.cnblogs.com/weizhxa/p/7744721.html)
[技术方案分享](https://cloud.tencent.com/developer/article/1751965)

插件(Plug-in,又称addin、add-in、addon或add-on,又译外挂)是一种遵循一定规范的应用程序接口编写出来的**程序**。其只能运行在程序规定的系统平台下（可能同时支持多个平台），而不能脱离指定的平台单独运行。因为插件需要调用原纯净系统提供的函数库或者数据。

### 3.1.  插件化相关概念

1. Android App中插件具体表示什么？一个插件其实就是一个apk，二者没有任何不同之处。
2. 什么是插件工程？能够编译生成插件apk的工程，就是插件工程；
3. 什么是插件化(What)?  不依赖安装，动态加载Native功能。将一个大apk拆分成多个小apk的过程；
4. 为什么要插件化(Why)?  动态发版、包大小精简、逻辑解耦、编译提速？
5. 带来这些好处的背后，有哪些挑战？ 客户端稳定性，开发体验。
6. 什么时候出现插件化(When)?  2012年。
7. 插件化在哪些地方盛行(Where)?  国内，手淘、携程、滴滴等大型App
8. 谁在做插件化(Who)?  前期个人开发者，后期大公司。
9. 怎么做插件化(How)?  总的来说，插件化分**Hook派和静态代理派**，Hook派涉及的基础知识会更多一些。

### 3.2. 插件化的历史与发展现状

插件化框架历史：

![](%E6%88%AA%E5%B1%8F2022-03-06%2021.13.58.png)

1. 各插件框架，都是基于自身App的业务来开发的，目标或多或少都有区别，所以很难有一个插件框架能一统江湖解决所有问题。
2. Android每次版本升级都会给各个插件化框架带来不少冲击，都要**费劲心思适配一番**，更别提**国内各个厂商对在ROM上做的定制**了，正如VirtualAPK的作者任玉刚所说：完成一个插件化框架的 Demo 并不是多难的事儿，然而要开发一款完善的插件化框架却并非易事。
3. 在2020年，Android插件化依旧是一个**高风险的技术**，涉及到**各个Android SDK版本、各个OEM厂商的兼容性适配**问题。

### 3.3. 插件化要解决的问题

实践中问题：发布不灵活、软件包过大、编译慢、模块不够独立、数亿用户级别（每一处改动不能对用户有影响）。

动态发版、包大小精简、逻辑解耦、编译提速。插件化无非是为了解决类加载和资源加载的问题,资源加载一般是通过反射 AssertManager , 按照类加载划分,插件化一般分为静态代理和 Hook 的方式,使用插件化一般为了解决应用新版本覆盖慢的问题。

四大组件可动态加载，意味着用户不需要手动安装新版本的应用，我们也可以给用户提供新的功能和页面，或者在用户无感的情况下修复 bug。

![](DraggedImage-10.tiff)

插件化为了解决如下问题：

1. 解决的是线上运行阶段业务方发布时间的问题。不需要在一起集成，一起发布了，每个业务方自行管理版本。
2. 包体积的问题，方法数或变量数也能随之解决。现在谷歌开放的app-bundles，他的大体作用也是将一个巨大的APP，拆分成不同的budle，用户在首次安装的时候，可能只需要下载2m，其他模块，按需下载；

优点：每个业务组件都变为了一个独立的App，这个App可以独立编译运行，因此可以大大提高编译时间，降低调试成本；每个插件App可以插入到不同的项目去使用；

![](DraggedImage-11.tiff)

![](DraggedImage-12.tiff)
![](%E6%88%AA%E5%B1%8F2022-03-06%2020.33.27.png)

### 3.4. 插件化工程结构

![](DraggedImage-2.png)

插件化工程结构与组件化的主要区别，就是业务组件变为了一个个插件化工程，它们都可以独立编译为一个apk。

### 3.5. 插件化框架会遇到的技术问题

[https://www.51cto.com/article/687386.html](https://www.51cto.com/article/687386.html)
[https://www.kindem.xyz/post/29/](https://www.kindem.xyz/post/29/)

![](DraggedImage-13.tiff)

不接入的三种担心：

1. 不够稳定：
2. 不够灵活：灵活与稳定之间还需要掌握一个平衡。
3. 大项目才用：“核心功能皆为插件”

### 3.6. 插件化框架需要储备的技术

![](DraggedImage-14.tiff)

1. Apk打包流程（Gradle、aapt）：有一些方案会在编译器修改插件的字节码，还有些方案在编译器做资源重排，所以要了解此部分内容。
2. Apk安装流程（PMS、Manifest解析、dexopt）:类似DroidPlugin、VirutalApp等方案，模拟了系统安装apk等流程，所以这部分需要了解。
3. 四大组件的管理（AMS，主要是启动流程、生命周期）:插件化的核心就是**四大组件的插件化**，所以这部分必须了解。
4. 资源、so库的加载机制：AssetManager
5. Context：应用上下文对象，环境Env；
6. IPC/Binder：Hook派插件化要和`system_process`进程打交道，而比较成熟的方案都会**把插件作为一个单独的进程**，所以这部分也要了解。
   7. 类加载机制：**加载插件的重要环节就是加载插件类**，这个是任何方案都无法避免的。
8. 反射与动态代理：Hook派插件必须用到的Java技术。

![](DraggedImage-15.tiff)![](%E6%88%AA%E5%B1%8F2022-03-05%2013.35.54.png)

### 3.7. 插件化缺点

1. 接入难度大，改造成本高；对代码的改动很大；
2. 受插件化框架的局限；
	- 比如不支持启动前台service，那么插件化改造就无法支持。但是基本可以满足常见的需求；

### 3.8. 插件化技术选型

[ 插件化技术选型 ](https://huanle19891345.github.io/en/android/%E6%8F%92%E4%BB%B6%E5%8C%96/%E6%8F%92%E4%BB%B6%E5%8C%96%E6%8A%80%E6%9C%AF%E9%80%89%E5%9E%8B/)
[Atlas、replugin、VirtualApk对比](https://dim.red/2018/03/31/plugin_framework_%20exploration/)

1. [Atlas](https://github.com/alibaba/atlas): 阿里巴巴。只支持到Android 9.0版本。功能最齐全，学习和接入最困难，适合超大型App，否则有点杀鸡用牛刀； 2 years go last update ；8k star 
2. [Replugin](https://github.com/Qihoo360/RePlugin)：奇虎360。Api最贴近原生Api，使用简单，只有一个hook点，因此兼容性非常好；适合中小型App；10months ago last update； 6.9k star；虽然唯一Hook点为宿主Application#LoadedApk中的classLoader对象，但源码中依然存在着众多的invoke反射，和**google禁止使用非公开api**的策略相违背；
3. [VirtualApk](https://github.com/didi/VirtualAPK): 4 years ago last update。8.6k star 滴滴
4. [DynamicApk](https://github.com/CtripMobile/DynamicAPK): 6 years ago last update。3k star。
5. [Shadow](https://github.com/Tencent/Shadow): 19days ago last update。6.2k star 腾讯。经过线上亿级用户量检验。Shadow不仅开源分享了插件技术的关键代码，还完整的分享了上线部署所需要的所有设计。**零反射实现插件化，符合google禁止使用非公开api的策略**；
6. [DroidPlugin](https://github.com/DroidPluginTeam/DroidPlugin)：2 years ago last update；6.7k star。
7. [Small](https://github.com/wequick/Small)：4 years ago last update；6k star。
8. [Neptune](https://github.com/iqiyi/Neptune): 爱奇艺 3years ago last update；730 star；

### 3.9. 谷歌官方App Bundle

AppBundle是谷歌官方提供的动态化框架，它支持资源依据设备类型按需下载，支持功能模块的动态交付，与插件化有相似之处。

## 四、换个思路实现动态化

![](DraggedImage-16.tiff)

## 五、容器化

[VLayout：阿里项目已停止维护](https://github.com/alibaba/vlayout/blob/master/README-ch.md) ：为了解决页面形态灵活性的问题。框架内置提供了几种常用的布局类型，包括：网格布局、线性布局、瀑布流布局、悬浮布局、吸边布局等，一拖N布局，浮动布局，固定布局…。这样**实现了混合布局的能力**，并且支持扩展外部，实现特殊的布局方式。

[VLayout介绍：万能布局器？与容器化什么关系？就像一个大容器，啥都有？然后动态切换？](https://blog.css8.cn/post/7968273.html)

![](DraggedImage-17.tiff)

[VirtualView 阿里已停止维护](https://github.com/alibaba/Virtualview-Android): 解决页面组件动态发布更新的问题。它提供了一系列基础 UI 组件和布局组件能力，通过 XML 来搭建业务组件，并将 XML 模板编译成二进制数据，然后主体框架解析二进制数据并渲染出视图。当 XML 模板数据能动态下发的时候，客户端上的业务组件视图也就能动态更新了。编写 XML 模板的方式、序列化成二进制数据的协议，VirtualView 都很大程度上吸取了 Android 原生开发的开发方式和原理，但增加了数据绑定、表达式相关的能力。

[VirtualView](https://www.jianshu.com/p/b013357f6aaa)
[滴滴跨端解决方案 Chameleon](https://github.com/didi/chameleon)
[CML与其他框架区别](https://github.com/didi/chameleon/issues/12)
[CML介绍](https://blog.csdn.net/DiDi_Tech/article/details/106581071)








