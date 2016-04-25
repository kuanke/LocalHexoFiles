---
title: Activity自动化测试中使用Intent传值
date: 2016-04-21 16:33:06
categories: blog
tags:
---
Google Android官方网站给出了详细教程教你如何进行配置，编写测试代码，包括了单元测试，UI测试，App组件测试以及显示性能测试。
详细内容请见：[Getting Started with Testing](http://developer.android.com/intl/zh-cn/training/testing/start/index.html)

本篇博客内容只补充介绍UI测试中单个App测试部分。

android-test-support library 已经废弃了ActivityInstrumentationTestCase2类，改为使用JUnit4 rules,使用起来更为简洁方便。但是官方文档中缺少测试Activity时需要使用Intent传值的用法，本篇博客补充了如何在UI测试中使用
Intent传值。

先看最简单的例子，所有的Activity都是通过"android.intent.action.MAIN"的action来启动：

```
@SmallTest
@RunWith(AndroidJUnit4.class)
public class MainActivityTest {
 
    @Rule
    public ActivityTestRule<MainActivity> mActivityRule =
            new ActivityTestRule<>(MainActivity.class);
 
    @Test
    public void someTest() {
        /* Your activity is initialized and ready to go. */
    }
}
```
现在来看如何在当前类所有的测试方法中使用同一个intent，同时又可传递参数：

```
@SmallTest
@RunWith(AndroidJUnit4.class)
public class MainActivityTest {
 
    @Rule
    public ActivityTestRule<MainActivity> mActivityRule =
            new ActivityTestRule<MainActivity>(MainActivity.class) {
                @Override
                protected Intent getActivityIntent() {
                    Context targetContext = InstrumentationRegistry.getInstrumentation()
                        .getTargetContext();
                    Intent result = new Intent(targetContext, MainActivity.class);
                    result.putExtra("Name", "Value");
                    return result;
                }
            };
 
    @Test
    public void someTest() {
        /* Your activity is initialized and ready to go. */
    }
}
```
如果你想让每个@Test方法都使用自己的Intent，那你可以这么做：

```
@SmallTest
@RunWith(AndroidJUnit4.class)
public class MainActivityTest {
 
    @Rule
    public ActivityTestRule<MainActivity> mActivityRule =
            new ActivityTestRule<>(MainActivity.class, true, false);
 
    @Test
    public void someTest() {
        Context targetContext = InstrumentationRegistry.getInstrumentation()
            .getTargetContext();
        Intent intent = new Intent(targetContext, MainActivity.class);
        intent.putExtra("Name", "Value");
 
        mActivityRule.launchActivity(intent);
 
        /* Your activity is initialized and ready to go. */
    }
}
```


