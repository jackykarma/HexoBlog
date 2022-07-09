---
title: Android单元测试
date: 2022-03-27 14:09:06
tags: Android单元测试
categories: 测试
---


学习前问题：

1. 单元测试是什么？类型有哪些？
2. 为何要做单元测试？它有什么作用？
3. 单元测试要测试的内容是什么？
4. 单元测试与其他测试类型的关系是什么？
5. 单元测试相关的测试框架有哪些？

## 一、概述

Android中主要有三类测试：

1. 单元测试（Junit4、Mockito、PowerMockito、Robolectric）：test
2. 集成测试：androidTest
3. UI测试（Espresso、UI Automator）：针对性测试

当然还有压力测试（Monkey）。

### 1.1. 单元测试是什么？

Android中有一个测试金字塔说明了应用应如何包含三类测试（即小型、中型和大型测试）：

1. 小型测试是指单元测试，用于验证应用的行为，一次验证一个类。
2. 中型测试是指集成测试，用于验证模块内堆栈级别之间的互动或相关模块之间的互动。
3. 大型测试是指端到端测试，用于验证跨越了应用的多个模块的用户操作流程。

![](%E6%88%AA%E5%B1%8F2022-03-27%2014.11.37.png)

沿着金字塔逐级向上，从小型测试到大型测试，各类测试的保真度逐级提高，但维护和调试工作所需的执行时间和工作量也逐级增加。因此，您编写的单元测试应多于集成测试，集成测试应多于端到端测试。虽然各类测试的比例可能会因应用的用例不同而异，但我们通常建议各类测试所占比例如下：**小型测试占 70%**，中型测试占 20%，大型测试占 10%。

单元测试是应用测试策略中的基本测试。通过针对代码创建和运行单元测试，您可以轻松验证各个单元的逻辑是否正确。

单元测试通常以可重复的方式运用尽可能小的代码单元（可能是方法、类或组件）的功能。当您需要验证应用中特定代码的逻辑时，应构建单元测试。例如，如果您正在对某个类进行单元测试，测试可能会检查该类是否处于正确状态。通常，代码单元在隔离的环境中进行测试；您的测试仅影响和监控对该单元的更改。您可以使用依赖项提供器（如 Robolectric）或模拟框架将您的单元与其依赖项隔离开来。

### 1.2. 单元测试的分类

1. 本地单元测试（Local Tests）：仅在本地计算机上运行的单元测试。这些测试编译为在 Java 虚拟机 (JVM) 本地运行，以最大限度地缩短执行时间，提升测试效率。这种单元测试不依赖于Android框架，或者即使有依赖，也很方便使用模拟框架来模拟依赖，以达到隔离Android依赖的目的，模拟框架如google推荐的Mockito或是Robolectric。速度快保真度低。
2. 仪器化单元测试（Instrumented Tests）：在真机或模拟器上运行的单元测试，由于需要跑到设备上，比较慢，这些测试可以访问仪器（Android系统）信息，比如被测应用程序的上下文，一般地，**依赖不太方便通过模拟框架模拟时采用这种方式**。速度慢保真度高。

### 1.3. 单元测试要测什么？

1. 列出想要测试覆盖的正常、异常情况，进行测试验证;
2. 性能测试，例如某个算法的耗时等等。

### 1.4. 单元测试的意义

1. 提高稳定性，能够明确地了解是否正确的完成开发；
2. 在每次构建后运行单元测试可帮助您快速捕捉和修复由应用的代码更改导致的软件回归。
3. 单元测试可以确保单个方法按照正确预期运行。如果修改了某个方法的代码，只需确保其对应的单元测试通过，即可认为改动正确。
4. 测试代码本身就可以作为示例代码，用来演示如何调用该方法；
5. 快速反馈bug，跑一遍单元测试用例，定位bug；
6. 在开发周期中尽早通过单元测试检查bug，最小化技术债，越往后可能修复bug的代价会越大，严重的情况下会影响项目进度；
7. 为代码重构提供安全保障，在优化代码时不用担心回归问题，在重构后跑一遍测试用例，没通过说明重构可能是有问题的，更加易于维护。

### 1.5. 单元测试的局限

单元测试不适用于测试复杂的界面互动事件。对于此类事件，您应改用**界面测试框架**。

### 1.6. 指标

1. 执行时间：速度、效率；
2. 置信度：有效性
3. 保真度

## 二、测试环境

使用 JUnit 4 框架提供的标准 API。如果您的测试需要与 Android 依赖项互动，请添加 Robolectric 或 Mockito 库以简化您的本地单元测试。

### 2.1. 测试环境依赖

```c
dependencies {
    // Required -- JUnit 4 framework
    testImplementation 'junit:junit:4.13.2'
    // 可选的模拟框架 -- Robolectric environment
    testImplementation 'androidx.test:core:1.4.0'
    // 核心库Robolectric:使用Robolectric必须添加 Android官方文档不正确
    testImplementation 'org.robolectric:robolectric:4.8.1'
    // 可选的模拟框架 -- Mockito framework
    testImplementation 'org.mockito:mockito-core:4.4.0'
}
```

### 2.2. 创建单元测试类

![](%E6%88%AA%E5%B1%8F2022-03-26%2019.21.00.png)

在要所测试的类中，通过Comman + N(Mac)、Alt+Insert(Windows) 按键选择创建Test测试类。

![](%E6%88%AA%E5%B1%8F2022-03-26%2019.21.30.png)

此步可以选择测试框架、是否生成setUp方法、tearDown方法（JUnit中会解释）、是否要显示继承方法，以及选择要测试的方法列表。所以单元测试主要就是测试类中的方法。

![](%E6%88%AA%E5%B1%8F2022-03-26%2019.25.43.png)

此步选择要将测试类生成在test下还是androidTest下。当然对一个文件可以生成两次，一次生成纯Java方法的测试，比如使用JUnit框架测试纯Java方法，生成的Test类放入到test目录下；一次将需要android依赖的方法测试生成的类，选择放入到androidTest中。

位于androidTest中的测试类：

```c
public class ConfigAbilityTest extends TestCase {

    public void testGetAppName() {
    }
}
```

位于test中的测试类：

```c
public class ConfigAbilityTest extends TestCase {

    public void testSum() {
    }
}
```

虽然是可以这样，但是并不建议将一个类的接口拆分放入到两个不同目录。可以按照AS默认的方式处理即可：

```js
app/src
     ├── androidTest/java (仪器化单元测试/UI测试/Android集成测试)
     ├── main/java (业务代码)
     └── test/java  (本地单元测试)
```

选择JUnit4（**与引入的junit库版本保持一致，否则生成的类不正确**）后生成的测试类如下，放入到test/java下：

```js
public class ConfigAbilityTest {

    @Before
    public void setUp() throws Exception {
    }

    @After
    public void tearDown() throws Exception {
    }

    @Test
    public void getAndroidVersion() {
    }

    @Test
    public void getAppName() {
    }

    @Test
    public void sum() {
    }
}
```

> > 注意，生成的测试方法都是没有方法参数的。测试方法是没有参数，方法的输入值都是在测试方法中写入的。

## 三、构建本地单元测试

### 3.1. JUnit 测试Java方法

Junit4是事实上的Java标准测试库，并且它是JUnit框架有史以来的最大改进，其主要目标便是利用Java5的Annotation特性简化测试用例的编写。

![](%E6%88%AA%E5%B1%8F2022-03-26%2019.30.04.png)

单元测试类创建完成后，就需要为方法测试编写测试用例了。主要的测试方法——断言。

#### 添加测试用例

```js
@Test
public void sum() {
    // assertEquals重载了多个类型的实现，只是这里是比较int值而已。验证求和的值正确与否
    // 可以添加多组测试用例, 测试多组数据
    assertEquals(30, mConfigAbility.sum(10, 20));
    assertEquals(30, mConfigAbility.sum(1000, 20));
    assertNotEquals(30, mConfigAbility.sum(10, 20000));
    assertEquals(300, mConfigAbility.sum(100, 200));
    assertTrue("Sum result not right", mConfigAbility.sum(30, 40) == 70);
}
```

#### 异步测试

```js
public class ConfigAbilityTest {

    private ConfigAbility mConfigAbility;

    private ExecutorService sSingleExecutorService = Executors.newSingleThreadExecutor();

	...

    @Test
    public void sum() {
        // assertEquals重载了多个类型的实现，只是这里是比较int值而已。验证求和的值正确与否
        // 可以添加多组测试用例, 测试多组数据
        sSingleExecutorService.execute(new Runnable() {
            @Override
            public void run() {
                assertEquals(30, mConfigAbility.sum(10, 20));
                assertEquals(30, mConfigAbility.sum(1000, 20));
                assertNotEquals(30, mConfigAbility.sum(10, 20000));
                assertEquals(300, mConfigAbility.sum(100, 200));
                assertTrue("Sum result not right", mConfigAbility.sum(30, 40) == 70);
            }
        });
    }
}
```

从上面可以知道，你**可以在测试类中引入其他各种类来辅助你测试**。比如可以自己写一个数据生成器来生成模拟数据，然后输入这些数据对接口的输入输出进行测试。

#### 自定义Junit Rule 规则

实现TestRule接口并重写apply方法。

```js
public class JsonChaoRule implements TestRule {

    @Override
    public Statement apply(final Statement base, final Description description) {
        Statement repeatStatement =  new Statement() {
            @Override
            public void evaluate() throws Throwable {
                    //测试前的初始化工作
                    //执行测试方法
                    base.evaluate();
                    //测试后的释放资源等工作
            }
        };
        return repeatStatement;
    }
}
```

然后在想要的测试类中使用@Rule注解声明使用JsonChaoRule即可（注意被@Rule注解的变量必须是final的）：

```js
@Rule
public final JsonChaoRule repeatRule = new JsonChaoRule();
```

#### 手动、自动执行测试

1. 手动：右键测试类，点击run
2. 执行gradlew test即可同时测试module下所有的测试类，并在module下的build/reports/tests/下生成对应的index.html测试报告。

#### JUnit4 优缺点

优点：速度快，支持**代码覆盖率**等代码质量的检测工具，
缺点：无法单独对Android UI，一些类进行操作，与原生JAVA有一些差异。

#### JUnit 测试流程总结

1. 编写测试类。
2. 选择要测试的类，然后鼠标右键点击测试类，选择选择Go To-\>Test（或者使用快捷键Ctrl+Shift+T，此快捷键可以在方法和测试方法之间来回切换）在Test/java/项目测试文件夹/下自动生成测试模板。
3. 使用断言（assertEqual、assertEqualArrayEquals等等)进行单元测试。
4. 右键点击测试类，Run编写好的测试类。

### 3.2. Mock 测试

有些时候我们不免会引用Android框架的对象，但是我们单元测试又不是运行在真实设备上的，在运行时是没有构建出真实的Android对象的，不过我们可以通过mock程序来模拟一个假的对象，并且强制让该对象的接口返回我们预期的结果。

![](%E6%88%AA%E5%B1%8F2022-03-26%2021.37.54.png)

注意：多个模拟框架不可同时运行，因此要依据具体情况来选择合适的测试框架。

#### Robolectric 模拟框架

[官网](http://robolectric.org/) 

如果您的测试与多个 Android 框架依赖项互动，特别是以复杂的方式与这些依赖项互动，请使用 **AndroidX Test** 提供的 Robolectric 模拟框架，而不是使用Mockito。使用使用 Robolectric 添加框架依赖项，达到模拟Android环境目的。 Robolectric 在本地 JVM 或真实设备上执行真实的 Android 框架代码和原生框架代码的虚假对象。Robolectric通过一套能运行在JVM上的Android代码，解决了在Java单元测试中很难进行Android单元测试的痛点。

主要是解决仪器化测试中耗时的缺陷，仪器化测试需要安装以及跑在Android系统上，也就是需要在Android虚拟机或真机上面，所以十分的耗时，基本上每次来来回回都需要几分钟时间。针对这类问题，业界其实已经有了一个现成的解决方案: Pivotal实验室推出的Robolectric，通过使用Robolectrict模拟Android系统核心库的Shadow Classes的方式，我们可以像写本地测试一样写这类测试，并且直接运行在工作环境的JVM上，十分方便。

```js
dependencies {
    // 可选的模拟框架 -- Robolectric environment
    testImplementation 'androidx.test:core:1.4.0'
    // 核心库Robolectric:使用Robolectric必须添加 Android官方文档不正确
    testImplementation 'org.robolectric:robolectric:4.6'
}
```

```js
android {
   // ...
   testOptions {
       unitTests.includeAndroidResources = true
   }
}
```

首先给指定的测试类上面进行配置：

```js
@RunWith(RobolectricTestRunner.class)
```

目前4.6的Robolectric最高支持sdk版本为30。否则会报错如下：修改方法就是targetSdkVersion改为30即可。

```js
failed to configure com.jacky.unittest.ConfigAbilityTest.sum: Package targetSdkVersion=31 > maxSdkVersion=30
java.lang.IllegalArgumentException: failed to configure com.jacky.unittest.ConfigAbilityTest.sum: Package targetSdkVersion=31 > maxSdkVersion=30
	at org.robolectric.RobolectricTestRunner.getChildren(RobolectricTestRunner.java:256)
	at org.junit.runners.ParentRunner.getFilteredChildren(ParentRunner.java:534)
	at org.junit.runners.ParentRunner.getDescription(ParentRunner.java:400)
	at org.junit.runners.model.RunnerBuilder.configureRunner(RunnerBuilder.java:81)
	at org.junit.runners.model.RunnerBuilder.safeRunnerForClass(RunnerBuilder.java:72)
```

下面是一些常用用法：

```js
//当Robolectric.setupActivity()方法返回的时候，
//默认会调用Activity的onCreate()、onStart()、onResume()
//获取到Activity之后就可以调用其所有公开的方法，比如获取view
mTestActivity = Robolectric.setupActivity(TestActivity.class);

//获取TestActivity对应的影子类，从而能获取其相应的动作或行为
ShadowActivity shadowActivity = Shadows.shadowOf(mTestActivity);
Intent intent = shadowActivity.getNextStartedActivity();

//使用ShadowToast类获取展示toast时相应的动作或行为
Toast latestToast = ShadowToast.getLatestToast();
Assert.assertNull(latestToast);
//直接通过ShadowToast简单工厂类获取Toast中的文本
Assert.assertEquals("hahaha", ShadowToast.getTextOfLatestToast());

//使用ShadowAlertDialog类获取展示AlertDialog时相应的
//动作或行为（暂时只支持app包下的，不支持v7。。。）
latestAlertDialog = ShadowAlertDialog.getLatestAlertDialog();
AlertDialog latestAlertDialog = ShadowAlertDialog.getLatestAlertDialog();
Assert.assertNull(latestAlertDialog);

//使用RuntimeEnvironment.application可以获取到
//Application，方便我们使用。比如访问资源文件。
Application application = RuntimeEnvironment.application;
String appName = application.getString(R.string.app_name);
Assert.assertEquals("WanAndroid", appName);

//也可以直接通过ShadowApplication获取application
ShadowApplication application = ShadowApplication.getInstance();
Assert.assertNotNull(application.hasReceiverForIntent(intent));
```

自定义Shadow类：

```js
@Implements(Person.class)
public class ShadowPerson {

    @Implementation
    public String getName() {
        return "AndroidUT";
    }

}

@RunWith(RobolectricTestRunner.class)
@Config(constants = BuildConfig.class,
        sdk = 23,
        shadows = {ShadowPerson.class})

    Person person = new Person();
    //实际上调用的是ShadowPerson的方法，输出JsonChao
    Log.d("test", person.getName());

    ShadowPerson shadowPerson = Shadow.extract(person);
    //测试通过
    Assert.assertEquals("JsonChao", shadowPerson.getName());

}
```

```js
@RunWith(RobolectricTestRunner.class)
public class ConfigAbilityTest {

    private ConfigAbility mConfigAbility;

    // ApplicationProvider来自androidX Test的Robolectric框架
    private final Context mContext = ApplicationProvider.getApplicationContext();

    private final ExecutorService sSingleExecutorService = Executors.newSingleThreadExecutor();

    @Before
    public void setUp() throws Exception {
        mConfigAbility = new ConfigAbility(mContext);
    }

	.....

    @Test
    public void getAppName() {
        assertEquals(mConfigAbility.getAppName(), "UnitTest");
    }

    @Test
    public void sum() {
        // assertEquals重载了多个类型的实现，只是这里是比较int值而已。验证求和的值正确与否
        // 可以添加多组测试用例, 测试多组数据
        sSingleExecutorService.execute(new Runnable() {
            @Override
            public void run() {
                assertEquals(30, mConfigAbility.sum(10, 20));
                assertEquals(30, mConfigAbility.sum(1000, 20));
                assertNotEquals(30, mConfigAbility.sum(10, 20000));
                assertEquals(300, mConfigAbility.sum(100, 200));
                assertTrue("Sum result not right", mConfigAbility.sum(30, 40) == 70);
            }
        });
    }
}
```

执行结果：

![](%E6%88%AA%E5%B1%8F2022-03-26%2021.14.07.png)

异步测试出现一些问题（比如改变一些编码习惯，比如回调函数不能写成匿名内部类对象，需要定义一个全局变量，并破坏其封装性，即提供一个get方法，供UT调用），解决方案使用Mockito来结合进行测试，将异步转为同步。

Robolectric的优缺点：

1. 优点：支持大部分Android平台依赖类底层的引用与模拟。
2. 缺点：异步测试有些问题，需要结合一些框架来配合完成更多功能。

#### Mockito 模拟框架

[官方](https://site.mockito.org/)

> > Mock对象，模拟控制其方法返回值，监控其方法的调用等。

如果您的测试对 Android 框架的依赖性极小，或者如果测试仅取决于您自己的对象，可以使用诸如 Mockito 之类的模拟框架添加模拟依赖项。Mockito 是美味的 Java 单元测试 Mock 框架，mock可以模拟各种各样的对象，从而代替真正的对象做出希望的响应。

如果您的测试对 Android 的依赖性极小，并且您需要在应用中测试组件与其依赖项之间的特定互动，请使用模拟框架对代码中的外部依赖项打桩。这样，您就可以轻松地测试组件是否按预期方式与依赖项互动。通过用模拟对象代替 Android 依赖项，您可以将单元测试与 Android 系统的其余部分隔离，同时验证是否调用了这些依赖项中的正确方法。适用于 Java（版本 1.9.5 及更高版本）的 Mockito 模拟框架提供了与 Android 单元测试的兼容性。通过 Mockito，您可以将模拟对象配置为在被调用时返回某个特定值。

如需使用此框架将模拟对象添加到本地单元测试，请遵循以下编程模型：

1. 在 build.gradle 文件中添加 Mockito 库依赖项，如设置测试环境中所述。
2. 在单元测试类定义的开头，添加 @RunWith(MockitoJUnitRunner.class) 注释。此注释可告知 Mockito 测试运行程序验证您对框架的使用是否正确无误，并简化了模拟对象的初始化。
3. 如需为 Android 依赖项创建模拟对象，请在字段声明前添加 @Mock 注释。
4. 如需模拟依赖项的行为，您可以使用 when() 和 thenReturn() 方法来指定某种条件以及满足该条件时的返回值。

```c
dependencies {
    // 可选的模拟框架 -- Mockito framework
    testImplementation 'org.mockito:mockito-core:4.4.0'
}
```

```c
import static org.hamcrest.core.Is.is;
import static org.junit.Assert.*;
import static org.mockito.ArgumentMatchers.anyInt;
import static org.mockito.Mockito.*; // 静态导入 方便
import static org.mockito.internal.verification.VerificationModeFactory.atLeast;

@RunWith(MockitoJUnitRunner.class)
public class MyClassTest {

    @Mock
    MyClass test;

    @Test
    public void mockitoTestExample() throws Exception {

        //可是使用注解@Mock替代
        //MyClass test = mock(MyClass.class);

        // 当调用test.getUniqueId()的时候返回43
        when(test.getUniqueId()).thenReturn(18);
        // 当调用test.compareTo()传入任意的Int值都返回43
        when(test.compareTo(anyInt())).thenReturn(18);

        // 当调用test.close()的时候，抛NullPointerException异常
        doThrow(new NullPointerException()).when(test).close();
        // 当调用test.execute()的时候，什么都不做
        doNothing().when(test).execute();

        assertThat(test.getUniqueId(), is(18));
        // 验证是否调用了1次test.getUniqueId()
        verify(test, times(1)).getUniqueId();
        // 验证是否没有调用过test.getUniqueId()
        verify(test, never()).getUniqueId();
        // 验证是否至少调用过2次test.getUniqueId()
        verify(test, atLeast(2)).getUniqueId();
        // 验证是否最多调用过3次test.getUniqueId()
        verify(test, atMost(3)).getUniqueId();
        // 验证是否这样调用过:test.query("test string")
        verify(test).query("test string");
        // 通过Mockito.spy() 封装List对象并返回将其mock的spy对象
        List list = new LinkedList();
        List spy = spy(list);
        //指定spy.get(0)返回"Jdqm"
        doReturn("Jdqm").when(spy).get(0);
        assertEquals("Jdqm", spy.get(0));
    }
}
```

使用mock方法模拟对象：

```c
Person mPerson = mock(Person.class); 
```

在JUnit框架下，case（即每一个测试点，带@Test注解的那个函数）也是个函数，直接调用这个函数就不是case，和case是无关的，两者并不会相互影响，可以直接调用以减少重复代码。单元测试不应该对某一个条件过度耦合，因此，需要用mock解除耦合，直接mock出网络请求得到的数据，单独验证页面对数据的响应。

验证方法的调用，指定方法的返回值，或者执行特定的动作：

```c
when(iMathUtils.sum(1, 1)).thenReturn(2); 
doReturn(3).when(iMathUtils).sum(1,1);   
//给方法设置桩可以设置多次，只会返回最后一次设置的值
doReturn(2).when(iMathUtils).sum(1,1);

//验证方法调用次数
//方法调用1次
Mockito.when(mockValidator.verifyPassword("xiaochuang_is_handsome")).thenReturn(true);
//方法调用3次
Mockito.when(mockValidator.verifyPassword("xiaochuang_is_handsome")
， Mockito.times(3).thenReturn(true);

//verify方法用于验证“模仿对象”的互动或验证发生的某些行为
verify(mPerson, atLeast(2)).getAge();

//参数匹配器,用于匹配特定的参数
any()
contains()
argThat()
when(mPerson.eat(any(String.class))).thenReturn("米饭");

//除了mock()外，spy()也可以模拟对象，spy与mock的
//唯一区别就是默认行为不一样：spy对象的方法默认调用
//真实的逻辑，mock对象的方法默认什么都不做，或直接
//返回默认值
//如果要保留原来对象的功能，而仅仅修改一个或几个
//方法的返回值，可以采用spy方法,无参构造的类初始
//化也使用spy方法
Person mPerson = spy(Person.class); 

//检查入参的mocks是否有任何未经验证的交互
verifyNoMoreInteractions(iMathUtils);
```

简单的测试会使整体的代码更简单，更可读、更可维护。如果你不能把测试写的很简单，那么请在测试时重构你的代码。

Mockito的优缺点：

1. 优点：丰富强大的方式验证“模仿对象”的互动或验证发生的某些行为
2. 缺点：Mockito框架不支持mock匿名类、final类、static方法、private方法。

虽然，static方法可以使用wrapper静态类的方式实现mockito的单元测试，但是，毕竟过于繁琐，因此，**PowerMockito**由此而来。

#### PowerMockito

[注意与mockito版本兼容](https://github.com/powermock/powermock/wiki/Mockito#supported-versions)

PowerMockito是一个扩展了Mockito的具有更强大功能的单元测试框架，它支持mock匿名类、final类、static方法、private方法。

```c
testImplementation 'org.powermock:powermock-module-junit4:1.6.5'
testImplementation 'org.powermock:powermock-api-mockito:1.6.5'
```

用PowerMockito来模拟对象：

```c
//使用PowerMock须加注解@PrepareForTest和@RunWith(PowerMockRunner.class)（@PrepareForTest()里写的
// 是对应方法所在的类    ，mockito支持的方法使用PowerMock的形式实现时，可以不加这两个注解）
@PrepareForTest(T.class)
@RunWith(PowerMockRunner.class)

//mock含静态方法或字段的类    
PowerMockito.mockStatic(Banana.class);

//Powermock提供了一个Whitebox的class，可以方便的绕开权限限制，可以get/set private属性，实现注入。
//也可以调用private方法。也可以处理static的属性/方法，根据不同需求选择不同参数的方法即可。
修改类里面静态字段的值
Whitebox.setInternalState(Banana.class, "COLOR", "蓝色");

//调用类中的真实方法
PowerMockito.when(banana.getBananaInfo()).thenCallRealMethod();

//验证私有方法是否被调用
PowerMockito.verifyPrivate(banana, times(1)).invoke("flavor");

//忽略调用私有方法
PowerMockito.suppress(PowerMockito.method(Banana.class, "flavor"));

//修改私有变量
MemberModifier.field(Banana.class, "fruit").set(banana, "西瓜");

//使用PowerMockito mock出来的对象可以直接调用final方法
Banana banana = PowerMockito.mock(Banana.class);

//whenNew 方法的意思是之后 new 这个对象时，返回某个被 Mock 的对象而不是让真的 new
//新的对象。如果构造方法有参数，可以在withNoArguments方法中传入。
PowerMockito.whenNew(Banana.class).withNoArguments().thenReturn(banana);
```

使用PowerMockRule来代替@RunWith(PowerMockRunner.class)的方式，需要多添加以下依赖：

```c
testImplementation "org.powermock:powermock-module-junit4-rule:1.7.4"
testImplementation "org.powermock:powermock-classloading-xstream:1.7.4"
```

```c
@Rule
public PowerMockRule mPowerMockRule = new PowerMockRule();
```

使用Parameterized来进行参数化测试：通过注解@Parameterized.parameters提供一系列数据给构造器中的构造参数或给被注解@Parameterized.parameter注解的public全局变量。

```c
RunWith(Parameterized.class)
public class ParameterizedTest {

    private int num;
    private boolean truth;

    public ParameterizedTest(int num, boolean truth) {
        this.num = num;
        this.truth = truth;
    }

    //被此注解注解的方法将把返回的列表数据中的元素对应注入到测试类
    //的构造函数ParameterizedTest(int num, boolean truth)中
    @Parameterized.Parameters
    public static Collection providerTruth() {
        return Arrays.asList(new Object[][]{
                {0, true},
                {1, false},
                {2, true},
                {3, false},
                {4, true},
                {5, false}
        });
    }

//    //也可不使用构造函数注入的方式，使用注解注入public变量的方式
//    @Parameterized.Parameter
//    public int num;
//    //value = 1指定括号里的第二个Boolean值
//    @Parameterized.Parameter(value = 1)
//    public boolean truth;

    @Test
    public void printTest() {
        Assert.assertEquals(truth, print(num));
        System.out.println(num);
    }

    private boolean print(int num) {
        return num % 2 == 0;
    }

}
```

#### 错误：“Method ... not mocked”

如果您运行的测试从并未模拟的 Android SDK 调用 API，您会收到一条错误，指出未模拟此方法。这是因为，用于运行单元测试的 android.jar 文件不包含任何实际代码（这些 API 仅由设备上的 Android 系统映像提供）。

默认情况下，所有方法都会抛出异常。这是为了确保单元测试仅测试代码，而不依赖于 Android 平台的任何特定行为，即您未明确模拟（如使用 Mockito 模拟）的行为。

如果抛出的异常会给测试带来问题，您可以通过在项目的顶级 build.gradle文件中添加以下配置来更改行为，以使方法返回null或0：

```js
android {
  ...
  testOptions {
    unitTests.returnDefaultValues = true
  }
}    
```

> > 注意：将 returnDefaultValues 属性设为 true 时应格外小心。null/0 返回值会在测试中引入回归，这很难调试，并且可能会允许失败的测试通过。只有在万不得已时，才能使用此配置。

## 四、构建仪器化单元测试

在某些情况下，虽然可以通过模拟的手段来隔离Android依赖，但代价很大，这种情况下可以考虑仪器化的单元测试，有助于减少编写和维护模拟代码所需的工作量。

仪器化单元测试（或插装单元测试）是在真机或模拟器上运行的测试，它们可以利用Android framework APIs 和 supporting APIs，如 AndroidX Test。如果测试用例需要访问仪器(instrumentation)信息(如应用程序的Context)，或者需要Android框架组件的真正实现(如Parcelable或SharedPreferences对象)，那么应该创建仪器化单元测试，插桩测试提供的保真度比本地单元测试要高，但运行速度要慢得多。因此，**建议只有在必须针对真实设备的行为进行测试时才使用插桩单元测试**。AndroidX Test 提供了几个库，可让您在必要时更轻松地编写插桩单元测试。例如，Android Builder 类可让您更轻松地创建本来难以构建的 Android 数据对象。

### 4.1. 设置测试环境

```c
dependencies {
    androidTestImplementation 'androidx.test:runner:1.1.0'
    androidTestImplementation 'androidx.test:rules:1.1.0'
    // Optional -- Hamcrest library
    androidTestImplementation 'org.hamcrest:hamcrest-library:1.3'
    // Optional -- UI testing with Espresso
    androidTestImplementation 'androidx.test.espresso:espresso-core:3.4.0'
    // Optional -- UI testing with UI Automator
    androidTestImplementation 'androidx.test.uiautomator:uiautomator:2.2.0'
}
```

如需使用 JUnit 4 测试类，请务必在您的项目中将 AndroidJUnitRunner 指定为默认插桩测试运行程序，方法是在应用的模块级 build.gradle 文件中添加以下设置：

```c
android {
    defaultConfig {
        testInstrumentationRunner "androidx.test.runner.AndroidJUnitRunner"
    }
}
```

### 4.2. 创建仪器化单元测试类

插桩单元测试类应该是一个 JUnit 4 测试类，它类似于有关如何创建本地单元测试类的部分中介绍的类。如需创建 JUnit 4 插桩测试类，请将 AndroidJUnit4 指定为默认测试运行程序。

> > 注意：如果您的测试套件依赖于 JUnit3 和 JUnit4 库的混合搭配，请在测试类定义的开头添加 @RunWith(AndroidJUnit4::class) 注释。

```c
package com.jacky.unittest;

import static org.junit.Assert.assertEquals; // 需要手动导入

import android.content.Context;

import androidx.test.ext.junit.runners.AndroidJUnit4;
import androidx.test.filters.SmallTest;
import androidx.test.platform.app.InstrumentationRegistry;

import org.junit.After;
import org.junit.Before;
import org.junit.Test;
import org.junit.runner.RunWith;

@RunWith(AndroidJUnit4.class)
@SmallTest
public class ConfigAbilityTest {

    private ConfigAbility mConfigAbility;

    @Before
    public void setUp() throws Exception {
        Context appContext = InstrumentationRegistry.getInstrumentation().getTargetContext();
        mConfigAbility = new ConfigAbility(appContext);
    }

    @After
    public void tearDown() throws Exception {
    }

    @Test
    public void getAndroidVersion() {
    }

    @Test
    public void getAppName() {
        assertEquals("UnitTest", mConfigAbility.getAppName());
    }

    @Test
    public void sum() {
        assertEquals(mConfigAbility.sum(10, 20), 30);
    }
}
```

### 4.3. 执行测试

方法：右键测试类，点击Run运行。仪器化单元测试运行的时候会启动AVD或是真机，因为它们需要在真机上运行。

![](%E6%88%AA%E5%B1%8F2022-03-27%2009.55.10.png)


## 五、单元测试覆盖率报告生成


### 5.1. AS直接生成

```c
package com.jacky.unittest;

import static org.junit.Assert.assertEquals;

import android.content.Context;
import android.os.Build;

import androidx.test.core.app.ApplicationProvider;

import org.junit.Before;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.mockito.MockitoAnnotations;
import org.robolectric.RobolectricTestRunner;

@RunWith(RobolectricTestRunner.class)
public class ConfigAbility2Test {

    private ConfigAbility mConfigAbility;

    private Context mContext = ApplicationProvider.getApplicationContext();

    @Before
    public void setUp() throws Exception {
        MockitoAnnotations.openMocks(this);
        mConfigAbility = new ConfigAbility(mContext);
    }

    @Test
    public void getAndroidVersion() {
        assertEquals(Build.VERSION.SDK_INT, 30);
    }

    @Test
    public void getAppName() {
        assertEquals(mConfigAbility.getAppName(), "UnitTest");
    }

    @Test
    public void sum() {
        assertEquals(30, mConfigAbility.sum(10, 20));
    }
}
```

![](%E6%88%AA%E5%B1%8F2022-03-27%2010.18.17.png)

AS无法为仪器化单元测试生成覆盖率报告。右键可以看到无此选项。**报告经常生成失败，且查看不够直观。**

### 5.2. Jacoco

[Jacoco](https://www.jacoco.org/jacoco/)
[ The JaCoCo Plugin ](https://docs.gradle.org/current/userguide/jacoco_plugin.html)
[最新Jacoco接入方法](https://juejin.cn/post/6994772628615462920)

1. 运行test、androidTest下的类，会生成test数据
2. 开启jacoco，那么在运行时会生成.exec文件，此文件是jacoco报告生成的数据文件
3. 执行jacocoTestReport task即可生成jacoco报告，html、xml形式展示。此task主要是配置sourceDirectories、classDirectories、executionData路径。

Jacoco的全称为Java Code Coverage（Java代码覆盖率），可以生成java的单元测试代码覆盖率报告。

```c
apply plugin: 'jacoco'

android {
    buildTypes {
        debug {
            /**打开覆盖率统计开关**/
            testCoverageEnabled = true
        }
    }
}

//源代码路径，有多少个module，就在这里写多少个路径，如果你只有app一个module，那么就写一个就可以
def coverageSourceDirs = [
        '../app/src/main/java',
]

//class文件路径，如果你只有app一个module，那么就写一个就可以
def coverageClassDirs = [
        '/app/build/intermediates/javac/releaseUnitTest/classes',
]

//Jacoco 版本
jacoco {
    toolVersion = "0.8.7"
}

//生成报告task

task jacocoTestReport(type: JacocoReport) {
    dependsOn ['jacocoInit']
    group = "JacocoReport"
    description = "Generate Jacoco coverage reports after running tests."
    reports {
        xml.enabled = true
        html.enabled = true
    }
    classDirectories.from = files(files(coverageClassDirs).files.collect {
        println("rootDir: $rootDir" + it)
        fileTree(dir: "$rootDir" + it,
                // 过滤不需要统计的class文件
                excludes: ['**/R*.class',
                           '**/*$InjectAdapter.class',
                           '**/*$ModuleAdapter.class',
                           '**/*$ViewInjector*.class'
                ])
    })
    sourceDirectories.from = files(coverageSourceDirs)
    executionData.from = files("$buildDir/jacoco/testReleaseUnitTest.exec") // 注意exec文件

    doFirst {
        println "classDirs = $coverageClassDirs"
        coverageClassDirs.each { path ->
            println("$rootDir" + path)
            new File("$rootDir" + path).eachFileRecurse { file ->
                if (file.name.contains('$$')) {
                    file.renameTo(file.path.replace('$$', '$'))
                }
            }
        }
    }
}

//初始化Jacoco Task
task jacocoInit() {
    group = "JacocoReport"
    doFirst {
        File file = new File("$buildDir/outputs/code-coverage/")
        if (!file.exists()) {
            file.mkdir();
        }
    }
}
```

在app/build.gradle中引入jacoco.gradle。`apply from: 'jacoco.gradle'`

第一步：执行单元测试，生成exec文件

1. ./gradlew app:testReleaseUnitTest 生成exec在jacoco目录下
2. ./gradlew app:testDebugUnitTest 生成exec在debug目录下

![](%E6%88%AA%E5%B1%8F2022-03-27%2012.11.07.png)

第二步：将第一步任务执行生成的exec拷贝到jacoco配置指定的路径下，然后执行配置的jacoco任务，生成易读的html、xml文件。

第三步：使用浏览器打开html文件。

![](%E6%88%AA%E5%B1%8F2022-03-27%2012.53.46.png)

## 六、创建更容易读懂的断言

用例测试的方法一般有：

1. 一般可以使用junit.Assert提供的[系列方法](https://junit.org/junit4/javadoc/latest/org/junit/Assert.html)
2. [Hamcrest匹配器](https://github.com/hamcrest) 的is()和equalTo() 来比较预期结果与实际结果。
3. 如需创建容易读懂的测试来评估应用中的组件是否返回预期的结果，我们建议使用 Truth 库和 Android Assertions 中的类。

Guava 团队提供了一个名为 [Truth](https://truth.dev/) 的流利断言库。在构建测试的验证步骤（或 then 步骤）时，您可以使用此库来代替基于 JUnit 或 Hamcrest 的断言。通常，您可以使用 Truth 来表达某个对象具有特定属性，使用的短语包含您要测试的条件，例如：

```c
assertThat(object).hasFlags(FLAGS)
assertThat(object).doesNotHaveFlags(FLAGS)
assertThat(intent).hasData(URI)
assertThat(extras).string(string_key).equals(EXPECTED)
```

AndroidX Test 支持 Android 的其他几个主题，以使基于 Truth 的断言更易于构建：

1. BundleSubject
2. IntentSubject
3. MotionEventSubject
4. NotificationActionSubject
5. NotificationSubject
6. PendingIntentSubject
7. PointerCoordsSubject
8. PointerPropertiesSubject

## 六、编写单元测试注意事项

1. 单元测试代码本身必须非常简单，能一下看明白，决不能再为测试代码编写测试；
2. 每个单元测试应当互相独立，不依赖运行的顺序；
3. 测试时不但要覆盖常用测试用例，还要特别注意测试边界条件，例如输入为0，null，空字符串""等情况。
4. 要验证程序正确性，必然要给出所有可能的条件（极限编程），并验证其行为或结果，才算是100%覆盖条件。实际项目中，验证一般条件和边界条件就OK了。
5. 在实际项目中，单元测试对象与页面是一对一的，并不建议跨页面，这样的单元测试耦合太大，维护困难。
6. 需要写完后，看覆盖率，找出单元测试中没有覆盖到的函数分支条件等，然后继续补充单元测试case列表，并在单元测试工程代码中补上case。
7. 直到规划的页面中所有逻辑的重要分支、边界条件都被覆盖，该项目的单元测试结束。
8. 考虑可读性，对于方法名使用表达能力强的方法名，对于测试范式可以考虑使用一种规范, 如 RSpec-style。方法名可以采用一种格式，如: [测试的方法](#)[测试的条件](#)[符合预期的结果](#)。
9. 不要使用逻辑流关键字(If/else、for、do/while、switch/case)，在一个测试方法中，如果需要有这些，拆分到单独的每个测试方法里。
10. 测试真正需要测试的内容，需要覆盖的情况，一般情况只考虑验证输出（如某操作后，显示什么，值是什么）。
11. 不需要考虑测试private的方法，将private方法当做黑盒内部组件，测试对其引用的public方法即可；不考虑测试琐碎的代码，如getter或者setter。
12. 每个单元测试方法，应没有先后顺序；尽可能的解耦对于不同的测试方法，不应该存在Test A与Test B存在时序性的情况。


















