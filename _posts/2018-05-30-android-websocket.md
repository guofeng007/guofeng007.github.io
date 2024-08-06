---
layout: post
title: Android Jetpack Navigation
categories: Blog
description:  Android Jetpack Navigation
keywords:      Android Jetpack Navigation

---

# Android Jetpack Navigation

搬运一篇好文章备用，[参见](https://www.jianshu.com/p/ad040aab0e66)

## 前言

我在项目中尽量避免 **使用** 和 **管理**  Fragment, 尤其是处理Fragment的 **多重嵌套** 和 **回退栈**的情况。所幸有Activity在，我绕过了很多 Fragment **复杂的使用场景**——必须承认，相比Activity的简单易上手，Fragment的古灵精怪令我头痛不已。

当然，Github上也有很多前辈对于Fragment的管理框架，这是 **最简单** 的解决方案，比如目前比较火的 **Fragmentation**，以及我司低调的 [Yumenokanata](https://github.com/Yumenokanata)大神 **函数式架构** 的 **FragmentManager**。它们都经历过若干项目的检验，框架 **成熟** 且 **稳定**，其中的设计思想我甚至觉得 **整个职业生涯都难以企及**。

但我一直没有尝试使用它们，原因是因为Activity的存在，我觉得没有足够的必要在 **复杂的场景**使用多Fragment去实现，简单的回退栈管理通过Android原生的API也足以实现——可以说，Fragment复杂的管理应用一直是我的 **技术盲点**。

## 学习契机

在不久前的Google 2018 I/O大会上，Google正式推出了**AndroidJetpack**  ——这是一套组件、工具和指导，可以帮助开发者构建出色的 Android 应用，这其中就包含了去年推出的 Lifecycle, ViewModel, LiveData 以及 Room。除此之外，**AndroidJetpack** 还隆重推出了一个新的架构组件：**Navigation**。

从名字来看，我翻译它叫**导航**, 我们来看看Google官方对它的描述：

> 今天，我们宣布推出Navigation组件，**作为构建您的应用内界面的框架，重点是让单 Activity 应用成为首选架构**。利用Navigation组件对 Fragment 的原生支持，您可以获得架构组件的所有好处（例如生命周期和 ViewModel），同时让此组件为您处理 FragmentTransaction 的复杂性。此外，Navigation组件还可以让您声明我们为您处理的转场。它可以自动构建正确的“向上”和“返回”行为，包含对深层链接的完整支持，并提供了帮助程序，用于将导航关联到合适的 UI 小部件，例如抽屉式导航栏和底部导航。

抛开比较性的话题不谈（StoryBoard VS Navigation?），Navigation的发布让我意识到 **这是一个契机**，我觉得我有必要花时间去深入了解它——既能 **学习新的技术及理念** ，同时又能 **查漏补缺，完善自己的Android知识体系（Fragment的管理）**。

![img](http://upload-images.jianshu.io/upload_images/7293029-aefa8cf49b8820ce.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/443)

这件事立即被我列上日程，过去的一周，我闲暇之际仔细研究了 **Navigation**, 并略有心得，我尝试写下本文，在总结的同时，希望能够给后来的朋友们一些 **系统性的指导建议** 。如果可能，我甚至希望这篇文章能够做到:

> 本文不是详细的API说明文档，但仅通过阅读本文，能够对 **Navigation** 有一个系统性地学习—— **了解它，理解它，最后搞懂它**。

这对读写双方都是 **一次挑战**。完成它的第一步是做到：知道Navigation这个导航组件 **怎么用**。

## 了解Navigation

### 1.官方文档

**官方文档** 永远是最接近 **正确** 和 **核心理念** 的参考资料 —— 在不久之后，本文可能会因为框架本身API的迭代更新而 **毫无意义**，但官方文档不会，即使在最恶劣的风暴中，它依然是最可靠的 **指明灯**：

<https://developer.android.com/topic/libraries/architecture/navigation/>

其次，一个好的Demo能够起到重要的启发作用， 这里我推荐 [Google实验室](https://github.com/googlecodelabs) 的这个Sample:

项目地址：<https://github.com/googlecodelabs/android-navigation>
项目教程：<https://codelabs.developers.google.com/codelabs/android-navigation/#0>

这个教程Demo的优势在于，官方为这个Demo提供了 **一系列详细的教程**，通过一步步，引导学习每一个类或者组件的应用场景，最终完全上手 **Navigation**。

> 因为刚刚发布的原因，目前Navigation的中文教程 **极其匮乏**，许多资料的查阅可能需要开发者 **自备梯子**。不过请不必担心，本文会力争做到比其它同类文章讲解的 **更加全面**。

### 2.Sample展示

我写了一个Navigation的sample，它最终的效果是这样：

![img](http://upload-images.jianshu.io/upload_images/7293029-a0439fd823c7baec.gif?imageMogr2/auto-orient/strip%7CimageView2/2/w/498)

sample.gif

这是3个简单的Fragment之间跳转的情景，经过 **转场动画** 的修饰，它们之前的切换非常 **流畅** 且 **自然**。在展示的最后，我们可以看到，Fragment2 -> Fragment1的时候，实际上是由 用户 **点击手机Back键** 触发的。

项目结构图如下，这可以帮你尽快了解sample的结构：

![img](http://upload-images.jianshu.io/upload_images/7293029-88a837918a4c43f3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/338)

> 我把这个sample的源码托管在了我的github上，你可以通过 [点我查看源码](https://github.com/qingmei2/SampleNavigation) 。

### 3.尝试使用Navigation

> #### **Navigation目前仅AndroidStudio 3.2以上版本支持，如果您的版本不足3.2，请点此下载预览版AndroidStudio**

首先介绍Navigation的使用：

> 无论是否认可，我们都必须承认，Google已经在尝试让Kotlin上位，无论是今年IO大会的 **数据展示**，还是官方文档上的 **代码示例片段**，亦或是Google最新 **开源Demo的源码**，使用语言清一色 Kotlin，本文亦然。

![img](http://upload-images.jianshu.io/upload_images/7293029-69da8c7c81cd8c9f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/311)

#### ① 在Module下的build.gradle中添加以下依赖：

```
dependencies {
    def nav_version = '1.0.0-alpha01'
    implementation "android.arch.navigation:navigation-fragment:$nav_version"
    implementation "android.arch.navigation:navigation-ui:$nav_version"
}

```

#### ② 新建三个Fragment：

```
//3个Fragment，它们除了layout不同，没有其它区别
class MainPage1Fragment : Fragment() {

    override fun onCreateView(inflater: LayoutInflater, container: ViewGroup?,
                              savedInstanceState: Bundle?): View {
        return inflater.inflate(R.layout.fragment_main_page1, container, false)
    }
}

class MainPage2Fragment : Fragment() {

    override fun onCreateView(inflater: LayoutInflater, container: ViewGroup?,
                              savedInstanceState: Bundle?): View? {
        return inflater.inflate(R.layout.fragment_main_page2, container, false)
    }
}

class MainPage3Fragment : Fragment() {

    override fun onCreateView(inflater: LayoutInflater, container: ViewGroup?,
                              savedInstanceState: Bundle?): View? {
        return inflater.inflate(R.layout.fragment_main_page3, container, false)
    }
}

```

#### ③ 新建导航视图文件（nav_graph）

在res目录下新建navigation文件夹，然后新建一个navigation的resource文件，我叫它 **nav_graph_main.xml** ：

![img](http://upload-images.jianshu.io/upload_images/7293029-494482b3bdba6f6b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/700)

打开导航视图文件，我们可以在AndroidStudio 3.2版本上，进行可视化编辑，包括选择新增Fragment，或者拖拽，连接Fragment：

![img](http://upload-images.jianshu.io/upload_images/7293029-a79468b529eec3e9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/700)

#### ④ 编辑导航视图文件

我们打开Text标签，进入xml编辑的页面，并这样配置:

```
<?xml version="1.0" encoding="utf-8"?>
<navigation xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    app:startDestination="@id/page1Fragment">

    <fragment
        android:id="@+id/page1Fragment"
        android:name="com.qingmei2.samplejetpack.ui.main.MainPage1Fragment"
        android:label="fragment_page1"
        tools:layout="@layout/fragment_main_page1">
        <action
            android:id="@+id/action_page2"
            app:destination="@id/page2Fragment" />
    </fragment>

    <fragment
        android:id="@+id/page2Fragment"
        android:name="com.qingmei2.samplejetpack.ui.main.MainPage2Fragment"
        android:label="fragment_page2"
        tools:layout="@layout/fragment_main_page2">
        <action
            android:id="@+id/action_page1"
            app:popUpTo="@id/page1Fragment" />
        <action
            android:id="@+id/action_page3"
            app:destination="@id/nav_graph_page3" />
    </fragment>

    <navigation
        android:id="@+id/nav_graph_page3"
        app:startDestination="@id/page3Fragment">
        <fragment
            android:id="@+id/page3Fragment"
            android:name="com.qingmei2.samplejetpack.ui.main.MainPage3Fragment"
            android:label="fragment_page3"
            tools:layout="@layout/fragment_main_page3" />
    </navigation>

</navigation>

```

**注意**：请保证fragment标签下，android:name属性内包名的正确声明。

#### ⑤ 编辑MainActivity

在Activity中配置 Navigation **非常简单**，我们首先编辑Activity的布局文件，并在布局文件中添加一个 **NavHostFragment** :

```
<?xml version="1.0" encoding="utf-8"?>
<android.support.constraint.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:id="@+id/container"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".MainActivity">

    <fragment
        android:id="@+id/my_nav_host_fragment"
        android:name="androidx.navigation.fragment.NavHostFragment"
        android:layout_width="0dp"
        android:layout_height="0dp"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintLeft_toLeftOf="parent"
        app:layout_constraintRight_toRightOf="parent"
        app:layout_constraintTop_toTopOf="parent"
        app:defaultNavHost="true"
        app:navGraph="@navigation/nav_graph_main" />

</android.support.constraint.ConstraintLayout>

```

这是一个宽和高都 match_parent 的Fragment，它的作用就是 **导航界面的容器**。

这并不难以理解，我们需要在Activity中通过 Navigation **展示一系列的Fragment**，但是我们需要告诉Navigation 和Activity，这一系列的 Fragment **展示在哪**——NavHostFragment应运而生，我把它的作用归纳为 **导航界面的容器**。

这之后，在Activity中添加如下代码：

```
class MainActivity : AppCompatActivity() {

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)
    }

    override fun onSupportNavigateUp() =
            findNavController(this, R.id.my_nav_host_fragment).navigateUp()
}

```

onSupportNavigateUp()方法的重写，意味着Activity将它的 **back键点击事件的委托出去**，如果当前并非栈中顶部的Fragment, 那么点击back键，返回上一个Fragment。

#### ⑥ 最后，配置不同Fragment对应的跳转事件

```
class MainPage1Fragment : Fragment() {
     //隐藏了onCreateView()方法的实现，下同
    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        super.onViewCreated(view, savedInstanceState)
        btn.setOnClickListener {
            //点击跳转page2
            Navigation.findNavController(it).navigate(R.id.action_page2)
        }
    }
}

class MainPage2Fragment : Fragment() {
    
    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        super.onViewCreated(view, savedInstanceState)
        btn.setOnClickListener {
           //点击返回page1
            Navigation.findNavController(it).navigateUp()
        }
        btn2.setOnClickListener {
            //点击跳转page3
            Navigation.findNavController(it).navigate(R.id.action_page3)
        }
    }
}

class MainPage3Fragment : Fragment() {

    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        super.onViewCreated(view, savedInstanceState)
        //点击返回page2
        btn.setOnClickListener { Navigation.findNavController(it).navigateUp() }
    }
}

```

可以看到，我们对于Fragment 并非是通过原生的 **FragmentManager** 和 **FragmentTransaction** 进行控制的。而是通过以下API进行的控制：

- **Navigation.findNavController(params).navigateUp()**
- **Navigation.findNavController(params).navigate(actionId)**

> 到这里，Navigation最基本的使用就已经讲解完毕了。您可以通过运行预览和示例 **基本一致** 的效果，如果遇到问题，或者有疑问，可以[点我查看源码](https://github.com/qingmei2/SampleNavigation) 。

## 理解Navigation

我对于 **通过博客归纳总结** 的学习方式已近两年，我不断反思，一篇优秀的文章不仅是做到 **完整叙述**，同时，它更应该体现的是 **对思路的整理** 并 **简洁干净地阐述它们**。

做到这点并不容易，首先需要做到的就是 **不要仅局限于API的使用**——最初的学习中，通过上面的代码，我已经 **实现了Fragment的导航**。但是，上面的代码中，除了Activity 和 Fragment，其它的东西我一个都不认识。

我感觉很难受， 所谓 **行百里路半九十**，别说九十，这个Navigation，**我一窍不通**。

![img](http://upload-images.jianshu.io/upload_images/7293029-19778f7793d404c2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/457)

**仅有上述示例代码毫无意义**，通过它们，更应该将其理解为 **入门**；接下来我们需要做到 **了解每一个类的职责，理解框架设计者的思想**。

我们先思考这样一个问题：**如果让我们实现一个Fragment的导航库，首先要实现什么？**

### 1.NavGraphFragment：导航界面的容器

答案近在眼前。

即使我们使用原生的API，想展示一个Fragment，我们首先也需要 **定义一个容器承载它**。以往，它可能是一个 **RelativeLayout** 或者 **FrameLayout**，而现在，它被替换成了 **NavGraphFragment**。

这也就说明了，我们为什么要往Activity的layout文件中提前扔进去一个NavGraphFragment，因为我们需要导航的这些Fragment都展示在NavGraphFragment上面。

实际上它做了什么呢？来看一下NavGraphFragment的onCreateView()方法：

```
    public View onCreateView(@NonNull LayoutInflater inflater, @Nullable ViewGroup container,
                             @Nullable Bundle savedInstanceState) {
        FrameLayout frameLayout = new FrameLayout(inflater.getContext());
        frameLayout.setId(getId());
        return frameLayout;
    }

```

NavGraphFragment内部实例化了一个FrameLayout, **作为ViewGroup的载体，导航并展示其它Fragment**。

除此之外，你 **应当注意** 到在layout文件中，它还声明了另外两个属性：

> app:defaultNavHost="true"
> app:navGraph="@navigation/nav_graph_main"

app:defaultNavHost="true"这个属性意味着你的NavGraphFragment将会 **拦截系统Back键的点击事件**（因为系统的back键会直接关闭Activity而非切换Fragment），你同时 **必须重写** Activity的 **onSupportNavigateUp()** 方法，类似这样：

```
override fun onSupportNavigateUp()
        = findNavController(R.id.nav_host_fragment).navigateUp()

```

app:navGraph="@navigation/nav_graph_main"这个属性就很好理解了，它会指向一个navigation_graph的xml文件,这之后，NavGraphFragment就会 **导航并展示对应的Fragment**。

在我们使用Navigation的第一步，我们需要：

> **在Activity的布局文件中显示声明NavGraphFragment，并配置 app:defaultNavHost 和 app:navGraph属性**。

### 2.nav_graph.xml：声明导航结构图

NavGraphFragment作为Activity导航的 **容器** ，然后，其 **app:navGraph** 属性指向一个navigation_graph的xml文件，以声明其 **导航的结构**。

NavGraphFragment在 **获取** 并 **解析** 完这个xml资源文件后，它首先需要知道的是：

> 类似APP的home界面，NavGraphFragment首先要导航到哪里?

```
<?xml version="1.0" encoding="utf-8"?>
<navigation xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    app:startDestination="@id/page1Fragment">

    <fragment
        android:id="@+id/page1Fragment"
        android:name="com.qingmei2.samplejetpack.ui.main.MainPage1Fragment"
        android:label="fragment_page1"
        tools:layout="@layout/fragment_main_page1">
        <action
            android:id="@+id/action_page2"
            app:destination="@id/page2Fragment" />
    </fragment>
    //省略...
</navigation>

```

在navigation的根节点下，我们需要处理这样一个属性：

> app:startDestination="@id/page1Fragment"

**Destination** 是一个很关键的单词，它的直译是 **目的地**。**app:startDestination**属性便是声明这个id对应的 **Destination** 会被作为 **默认布局** 加载到Activity中。这也就说明了，为什么我们的sample，默认会显示 **MainPage1Fragment**。

现在，我们的app默认展示了MainPage1Fragment, 那么接下来，我们如何实现跳转逻辑的处理呢？

### 3.Action标签：声明导航的行为

我们声明了这样一个Action标签，这是一个 **导航的行为**：

```
<action
    android:id="@+id/action_page2"
    app:destination="@id/page2Fragment" />

```

**app:destination**的属性，声明了这个行为导航的 **destination（目的地）**，我们可以看到，它会指印跳转到 id 为 **page2Fragment** 的Fragment（也就是 **MainPage2Fragment**）。

**android:id** 这个id作为Action唯一的 **标识**，在Fragment的某个点击事件中，我们通过id指向**对应的行为**，就像这样：

```
btn.setOnClickListener {
       //点击跳转page2Fragment
       Navigation.findNavController(it).navigate(R.id.action_page2)
}

```

此外，Navigation还提供了一个 **app:popUpTo** 属性，它的作用是声明**导航行为** 将 **返回到** id对应的Fragment，比如，直接从Page3 返回到 Page1。

此外，Navigation 对导航行为还提供了 **转场动画** 的支持，它可以通过代码这样实现：

```
<action
        android:id="@+id/confirmationAction"
        app:destination="@id/confirmationFragment"
        app:enterAnim="@anim/slide_in_right"
        app:exitAnim="@anim/slide_out_left"
        app:popEnterAnim="@anim/slide_in_left"
        app:popExitAnim="@anim/slide_out_right" />

```

> 篇幅原因，这些anim的xml文件我并未展示在文中，如有需求，请参考[Sample代码](https://github.com/qingmei2/SampleNavigation)。

其实Navigation 还提供了对Destination之间 **参数传递** 的支持，以及对SubNavigation标签的支持，以方便开发者在xml文件中 **复用fragment标签** ——甚至是对 **Deep Link** 的支持，但这些拓展功能本文不再叙述。

### 4.Fragment：通过代码声明导航

其实在3中我们已经讲解了导航代码的使用,我们以Page2为例，它包含了2个按钮，分别对应 **返回Page1** 和 **进入Page3** 两个事件：

```
btn.setOnClickListener {
      Navigation.findNavController(it).navigateUp()
}
btn2.setOnClickListener {
      Navigation.findNavController(it).navigate(R.id.action_page3)
}

```

Navigation.findNavController(View) 返回了一个 **NavController** ,它是整个 **Navigation** 架构中 **最重要的核心类**，我们所有的导航行为都由 **NavController** 处理，这个我们后面再讲。

我们通过获取 **NavController**，然后调用  NavController.navigate()方法进行导航。

![img](http://upload-images.jianshu.io/upload_images/7293029-e2770edac9c83820.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/486)

我们更多情况下通过传入ActionId，指定对应的 **导航行为** ；同时可以通过传入Bundle以 **数据传递**；或者是再传入一个 **NavOptions**配置更多（比如 **转场动画**，它也可以通过这种方式进行代码的动态配置）。

NavController.navigate()方法更多时候应用在 **向下导航** 或者 **指定向上导航**（比如Page3 直接返回 Page1，跳过返回Page2的这一步）；如果我们处理back事件，我们应该使用 **NavController.
navigateUp()**。

### 恭喜您，已经能够游刃有余的使用Navigation!

恭喜您，您已对 **Navigation** 十分熟悉，并能通过熟练使用其 **暴露的API**，灵活地处理您应用中的 **页面导航** 行为。

我美滋滋的在个人履历上填上了这样一条：

- 熟练使用Google官方组件Navigation实现Fragment的管理，并掌握其原理

面试官对此十分感动，然后让我谈谈 **对它架构设计的一些个人观点**。

![img](http://upload-images.jianshu.io/upload_images/7293029-202e118f6bb5596f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/189)

到了这一步，我们算得上是 **API的搬运工** ，我们已经 **了解每一个类的职责**，还没有完全 **理解框架设计者的思想**。

## 彻底搞懂Navigation

在我们熟悉Navigation的API之后，我们整装待发，准备 **源码级攻克** Navigation。

> 正如我所说的，在这之前，您首先需要达到 **熟练使用Navigation**，本文地初衷并非是 **一步到位**，而是尝试 **循序渐进**。

### 1.对源码分析说NO

**声明** —— 我拒绝 **大段大段地源码分析**，我认为这种行为 **严重降低** 了文章的 **质量** 和 **深度**。

我花了一些时间绘制了 **Navigation的UML类图**，我坚信，这种方式能帮助你我 **更深刻的理解** Navigation的整体架构：

![img](http://upload-images.jianshu.io/upload_images/7293029-da074ee4ca0484ec.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/700)

UML类图

让我们换个角度，我们的身份不再是 **源码的观众**，而是 **架构的设计者**。

### 2.  设计 NavHostFragment

NavHostFragment 应当有两个作用：

- 作为Activity导航界面的载体
- **管理并控制导航的行为**

前者的作用我们已经说过了，我们通过在NavHostFragment的创建时，为它创建一个对应的FrameLayout作为 **导航界面的载体**：

```
    public View onCreateView(@NonNull LayoutInflater inflater, @Nullable ViewGroup container,
                             @Nullable Bundle savedInstanceState) {
        FrameLayout frameLayout = new FrameLayout(inflater.getContext());
        frameLayout.setId(getId());
        return frameLayout;
    }

```

我们都知道代码设计应该遵循 **单一职责原则**，因此，我们应该将 **管理并控制导航的行为** 交给另外一个类，这个类的作用应该仅是 **控制导航行为**，因此我们命名为 **NavController**。

Fragment理应持有这个**NavController**的实例，并将导航行为 **委托** 给它，这里我们将 **NavController** 的持有者抽象为一个 **接口**，以便于以后的拓展。

于是我们创造了 **NavHost** 接口，并让NavHostFragment实现了这个接口：

```
public interface NavHost {

    NavController getNavController();
}

```

为了保证导航的 **安全**，NavHostFragment 在其 **作用域** 内，理应 **有且仅有一个NavController 的实例**。

这里我们驻足一下，请注意API的设计，似乎 Navigation.findNavController(View)，参数中传递任意一个 view的引用似乎都可以获取 **NavController**——如何保证 **NavController 的局部单例**呢？

事实上，findNavController(View)内部实现是通过 **遍历** View树，直到找到最底部 **NavHostFragment** 中的**NavController**对象，并将其返回的：

```
private static NavController findViewNavController(@NonNull View view) {
        while (view != null) {
            NavController controller = getViewNavController(view);
            if (controller != null) {
                return controller;
            }
            ViewParent parent = view.getParent();
            view = parent instanceof View ? (View) parent : null;
        }
        return null;
  }

```

### 3.设计 NavController

站在 **设计者** 的角度，**NavController** 的职责是：

- 1.对navigation资源文件夹下nav_graph.xml的 **解析**
- 2.通过解析xml，获取所有 **Destination**（目标点）的 **引用** 或者 **Class的引用**
- 3.记录当前栈中 **Fragment的顺序**
- 3.管理控制 **导航行为**

**NavController** 持有了一个 **NavInflater** ,并通过 **NavInflater** 解析xml文件。

这之后，获取了所有 **Destination**（在本文中即**Page1Fragment** , **Page2Fragment** , **Page3Fragment** ） 的 Class对象，并通过反射的方式，实例化对应的 **Destination**，通过一个队列保存：

```
    private NavInflater mInflater;  //NavInflater 
    private NavGraph mGraph;        //解析xml，得到NavGraph
    private int mGraphId;           //xml对应的id，比如 nav_graph_main
    //所有Destination的队列,用来处理回退栈
    private final Deque<NavDestination> mBackStack = new ArrayDeque<>();   

```

这看起来没有任何问题，但是站在 **设计者** 的角度上，还略有不足，那就是，**Navigation并非只为Fragment服务**。

先不去吐槽Google工程师的野心，因为现在我们就是他，从拓展性的角度考虑，Navigation是一个导航框架，今后可能 **并非只为Fragment导航**。

我们应该为要将导航的 **Destination**  抽象出来，这个类叫做 **NavDestination** ——无论 **Fragment** 也好，**Activity** 也罢，只要实现了这个接口，对于**NavController** 来讲，他们都是 **Destination（目标点）**而已。

对于不同的 **NavDestination** 来讲，它们之间的导航方式是不同的，这完全有可能（比如Activity 和 Fragment），如何根据不同的 **NavDestination** 进行不同的 **导航处理** 呢？

#### 4. NavDestination 和 Navigator

有同学说，我可以这样设计，通过 instanceof 关键字，对 **NavDestination** 的类型进行判断，并分别做出处理，比如这样：

```
if (destination instanceof Fragment) {
  //对应Fragment的导航
} else if (destination instanceof Activity) {
  //对应Activity的导航
}

```

这是OK的，但是不够优雅，Google的方式是通过抽象出一个类，这个类叫做 **Navigator** ：

```
public abstract class Navigator<D extends NavDestination> {
    //省略很多代码,包括部分抽象方法，这里仅阐述设计的思路！

    //导航
    public abstract void navigate(@NonNull D destination, @Nullable Bundle args,
                                     @Nullable NavOptions navOptions);
    //实例化NavDestination（就是Fragment）
    public abstract D createDestination();

    //后退导航
    public abstract boolean popBackStack();
}

```

**Navigator**(导航者) 的职责很单纯：

- 1.能够实例化对应的 **NavDestination**
- 2.能够指定导航
- 3.能够后退导航

你看，我的 **NavController** 获取了所有 **NavDestination** 的Class对象，但是我不负责它 **如何实例化** ，也不负责 **如何导航** ，也不负责
**如何后退** ——我仅仅持有向上的引用，然后调用它的接口方法，它的实现我不关心。

以 **FragmentNavigator**为例，我们来看看它是如何执行的职责：

```
public class FragmentNavigator extends Navigator<FragmentNavigator.Destination> {
    //省略大量非关键代码，请以实际代码为主！
  
    @Override
    public boolean popBackStack() {
        return mFragmentManager.popBackStackImmediate();
    }

    @NonNull
    @Override
    public Destination createDestination() {
        // 实际执行了好几层，但核心代码如下，通过反射实例化Fragment
        Class<? extends Fragment> clazz = getFragmentClass();
        return  clazz.newInstance();
    }

    @Override
    public void navigate(@NonNull Destination destination, @Nullable Bundle args,
                            @Nullable NavOptions navOptions) {
        // 实际上还是通过FragmentTransaction进行的跳转处理
        final Fragment frag = destination.createFragment(args);
        final FragmentTransaction ft = mFragmentManager.beginTransaction();
        ft.replace(mContainerId, frag);
        ft.commit();
        mFragmentManager.executePendingTransactions();
    }
}

```

不同的 **Navigator** 对应不同的 **NavDestination**，**FragmentNavigator** 对应的是 **FragmentNavigator.Destination**，你可以把他理解为案例中的 **Fragment** ，有兴趣的朋友可以自己研究一下。

#### 5.至此

至此，Navigation 整体的架构设计 也已经通过 **UML类图** + **设计的角度分析** 的方式学习完了。

当然，Navigation 还有很多其它的类我没有去阐述，它们已经无法阻拦你我的脚步。

我更建议 读者在这之后，能够尝试自己阅读源码，通过借鉴上文中的 **UML类图**，当然，自己通过思路的整理，自己绘制出一份，会对理解它更有帮助。

## 总结

**Navigation** 是一个优秀的库，这从API上无法体现，因为它和其它优秀的三方 Fragment 管理库 都能达到 **固定的目标**。

并且，随着技术的不断发展，它们也早晚会被历史所淹没，我们能够做到的，就是使用API的同时，**学习它的思想，并收为己用**。