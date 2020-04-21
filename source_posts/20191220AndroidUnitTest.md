---
title: Android 开发如何进行单元测试
date: 2019-12-23 17:08:54
tags:
- Android UnitTest
categories:
- Android开发笔记
---

# 什么是单元测试

单元测试是由一组独立的测试构成，每个测试针对软件中的一个单独的程序单元。单元测试并非检查程序单元直接是否能够合作良好，而是检查单个程序单元的行为是否正确。

事实上，单元测试是一种**验证行为**，测试和验证程序中的每一项的正确性。


<!--more-->

# 为什么要进行单元测试

对于单元测试，人们往往存在很多误解：

1. 浪费时间太多，本身项目的时间就很紧张，没有时间去写单元测试。
2. 过度的依赖测试人员，认为软件开发人员不应该参与单元测试。
3. 认为单元测试不必要，代码写得很好了，no bug，no warning。
4. 老代码结构混乱，耦合度高，为了写单元测试修改代码结构，意义不大，投入跟产出不成比例。

> 单元测试真的这么鸡肋么？No，No，No！！！

试想

1. 测试人员给你报了一个bug，但是由于之后的merge失误导致代码丢失，或者别人修改代码导致这个bug再次复现。
2. 重构代码的时候，被bug淹没。造成你持续不断的改bug，持续不断的加班。
3. 明明很正常的功能，怎么现在突然不能用了？是接口的问题，还是有人修改了这个功能的逻辑？

    。。。

如果你也经常遇到这些困惑，那么你就需要对项目进行**单元测试**了。

因为**单元测试**具有以下优势：

1. **帮助理解需求**

    > 单元测试应该反映Use Case，把被测单元当成黑盒测试其外部行为。

2. **提高实现质量** 
    
    > 单元测试不保证程序做正确的事，但能帮助保证程序正确地做事，从而提高实现质量。
    
3. **测试成本低**

    > 相比集成测试、验收测试，单元测试所依赖的外部环境少，自动化程度高，时间短，节约了测试成本。  
    
4. **反馈速度快**

    > 单元测试提供快速反馈，把bug消灭在开发阶段，减少问题流到集成测试、验收测试和用户，降低了软件质量控制的成本。
    
5. **利于重构** 

    > 由于有单元测试作为回归测试用例，有助于预防在重构过程中引入bug。
    
6. **文档作用** 

    > 单元测试提供了被测单元的使用场景，起到了使用文档的作用。
    
7. **对设计的反馈** 

    > 一个模块很难进行单元测试通常是不良设计的信号，单元测试可以反过来指导设计出高内聚、低耦合的模块。
    
[为什么要做单元测试？](https://www.cnblogs.com/weidagang2046/articles/1753417.html)

# 怎么进行单元测试

## Android 单元测试分类

Android 单元测试分为两大类： 

```
app/src
     ├── androidTestjava (Instrumented 单元测试、UI测试)
     ├── main/java (业务代码)
     └── test/java  (Local 单元测试)
```

1. Local test:

    > 运行在本地的JVM虚拟机上，不依赖Android框架。
2. Instrumented tests:
    
    > 通过Android系统的Instrumented测试框架，运行测试代码在真实手机上。
    
## Android Junit + Mockito + Powermock单元测试方案
### Junit + Mockito + Powermock简介 
**Junit** 是一个Java语言的单元测试框架
**Mockito** 是一个Mock框架，我们可以通过Mockito框架创建配置mock对象。
**Powermock** 可以针对static，final，private方法进行mock

### Junit + Mockito + Powermock使用

强烈建议你熟读以下内容，来熟悉Junit + Mockito + Powermock的使用。

1. [Mockito 中文文档 ( 2.0.26 beta )](https://yq.aliyun.com/go/articleRenderRedirect?spm=a2c4e.11153940.0.0.43bf26detjrS1T&url=https%3A%2F%2Fgithub.com%2Fhehonghui%2Fmockito-doc-zh)
2. [Mockito reference documentation](https://yq.aliyun.com/go/articleRenderRedirect?spm=a2c4e.11153940.0.0.43bf26detjrS1T&url=http%3A%2F%2Fsite.mockito.org%2Fmockito%2Fdocs%2Fcurrent%2Forg%2Fmockito%2FMockito.html)
3. [powermock wiki](https://yq.aliyun.com/go/articleRenderRedirect?spm=a2c4e.11153940.0.0.43bf26detjrS1T&url=https%3A%2F%2Fgithub.com%2Fjayway%2Fpowermock%2Fwiki)
4. [Unit tests with Mockito - Tutorial](https://yq.aliyun.com/go/articleRenderRedirect?spm=a2c4e.11153940.0.0.43bf26detjrS1T&url=https%3A%2F%2Fgithub.com%2Fjayway%2Fpowermock%2Fwiki)

#### 比如说我们要对Calculate类进行单元测试

```java
public class Calculate {

    private int mPrivate;

    private final int mPrivateFinal = 0;

    private static int mPrivateStatic = 0;

    private static final int mPrivateStaticFinal = 0;

    public int mPublic;

    public final int mPublicFinal = 0;

    public static int mPublicStatic = 0;

    public static final int mPublicStaticFinal = 0;

    public void voidPublicMethod(int a, int b) {
        return;
    }

    public int addPublicMethod(int a, int b) {
        return a + b;
    }

    private int addPrivateMethod(int a, int b) {
        return a + b;
    }

    public static int addPublicStaticMethod(int a, int b) {
        return a + b;
    }

    private static int addPrivateStaticMethod(int a, int b) {
        return a + b;
    }

}
```

1. 测试Public 变量

    ```java
     @Test
    public void testPublicField() {
        assertEquals(mCalculate.mPublic, 0);
        assertEquals(mCalculate.mPublicFinal, 0);
        assertEquals(Calculate.mPublicStatic, 0);
        assertEquals(Calculate.mPublicStaticFinal, 0);

        mCalculate.mPublic = 1;
        Calculate.mPublicStatic = 2;

        assertEquals(mCalculate.mPublic, 1);
        assertEquals(mCalculate.mPublicFinal, 0);
        assertEquals(Calculate.mPublicStatic, 2);
    }
    ```
2. 测试Public 方法

    ```
      @Test
    public void testAddPublicMethod() {
        //when
        when(mCalculate.addPublicMethod(anyInt(), anyInt()))
                .thenReturn(0)
                .thenReturn(1);

        //call method
        for (int i = 0; i < 2; i++) {

            //verify
            assertEquals(mCalculate.addPublicMethod(i, i), i);
        }

        //verify
        verify(mCalculate, times(2)).addPublicMethod(anyInt(), anyInt());
        verify(mCalculate, atLeast(1)).addPublicMethod(anyInt(), anyInt());
        verify(mCalculate, atLeastOnce()).addPublicMethod(anyInt(), anyInt());
        verify(mCalculate, atMost(2)).addPublicMethod(anyInt(), anyInt());
    }
    ```
    
3. 测试Public 返回Void 方法

    ```
    @Test
    public void testAddPublicVoidMethod() {
        //when
        doNothing().when(mCalculate).voidPublicMethod(anyInt(), anyInt());

    }
    ```
    
4. 测试Public Static 方法

    ```java
    @Test
    public void testAddPublicStaicMethod() throws Exception {
        PowerMockito.mockStatic(Calculate.class);

        PowerMockito.when(Calculate.class, "addPublicStaticMethod", anyInt(), anyInt())
                .thenReturn(0)
                .thenReturn(1);
    }
    ```
    
5. 测试Private Static 变量

    ```java
     @Test
    public void testPrivate() throws IllegalAccessException {
        PowerMockito.mockStatic(Calculate.class);

        assertEquals(Whitebox.getField(Calculate.class, "mPrivate").getInt(mCalculate), 0);
        assertEquals(Whitebox.getField(Calculate.class, "mPrivateFinal").getInt(mCalculate), 0);
        assertEquals(Whitebox.getField(Calculate.class, "mPrivateStatic").getInt(null), 0);
        assertEquals(Whitebox.getField(Calculate.class, "mPrivateStaticFinal").getInt(null), 0);


        Whitebox.setInternalState(mCalculate, "mPrivate", 1);
        Whitebox.setInternalState(Calculate.class, "mPrivateStatic", 1, Calculate.class);

        assertEquals(Whitebox.getField(Calculate.class, "mPrivate").getInt(mCalculate), 1);
        assertEquals(Whitebox.getField(Calculate.class, "mPrivateFinal").getInt(mCalculate), 0);
        assertEquals(Whitebox.getField(Calculate.class, "mPrivateStatic").getInt(null), 1);
        assertEquals(Whitebox.getField(Calculate.class, "mPrivateStaticFinal").getInt(null), 0);
    }
    ```
    
6. 测试Private 方法

    ```java
     @Test
    public void testAddPrivateMethod() throws Exception {
        PowerMockito.mockStatic(Calc.class);

        //when
        PowerMockito.when(mCalculate,"addPrivateMethod",anyInt(),anyInt())
                .thenReturn(0)
                .thenReturn(1);
        
    }
    ```
    
7. 测试Private static 方法

    ```java
    @Test
    public void testAddPrivateStaicMethod() throws Exception {
        PowerMockito.mockStatic(Calculate.class);

        PowerMockito.when(Calculate.class, "addPrivateStaticMethod", anyInt(), anyInt())
                .thenReturn(0)
                .thenReturn(1);
    }
    ```
    
#### 对单例进行mock

比如对以下代码进行mock

```java
public class Singleton {

    public String PUBLIC_STR = "public_str";

    private static class ServiceSingleton {
        private static final Singleton SINGLE = new Singleton();
    }

    public static Singleton get() {
        return ServiceSingleton.SINGLE;
    }

    public String getPublicStr() {
        return PUBLIC_STR;
    }

}
```

```java
public class SingletonUse {
    private static final String PUBLIC_STR = "public_str";

    private static class ServiceSingleton {
        private static final SingletonUse SINGLE = new SingletonUse();
    }

    public static SingletonUse get() {
        return ServiceSingleton.SINGLE;
    }

    public String getUsePublicStr() {
        return Singleton.get().getPublicStr();
    }
}
```

测试类

```java
@RunWith(PowerMockRunner.class)
@PrepareForTest({Singleton.class, SingletonUse.class})
public class SingletonUseTest {

    private SingletonUse singletonUse;

    @Before
    public void setUp() throws Exception {
        Singleton singleton = PowerMockito.mock(Singleton.class);
        PowerMockito.mockStatic(Singleton.class);
        PowerMockito.doReturn(singleton).when(Singleton.class, "get");

        singletonUse = PowerMockito.mock(SingletonUse.class);
        PowerMockito.mockStatic(SingletonUse.class);
        PowerMockito.doReturn(singletonUse).when(SingletonUse.class, "get");
    }

    @Test
    public void getUsePublicStr() throws Exception {
        PowerMockito.doReturn("123456").when(singletonUse,"getUsePublicStr");
    }
}
```


    
# 参考资料

4. [Unit testing support](https://yq.aliyun.com/go/articleRenderRedirect?spm=a2c4e.11153940.0.0.43bf26detjrS1T&url=http%3A%2F%2Ftools.android.com%2Ftech-docs%2Funit-testing-support)
5. [junit4](https://yq.aliyun.com/go/articleRenderRedirect?spm=a2c4e.11153940.0.0.43bf26detjrS1T&url=http%3A%2F%2Fjunit.org%2Fjunit4%2F)
6. [mockito](https://yq.aliyun.com/go/articleRenderRedirect?spm=a2c4e.11153940.0.0.43bf26detjrS1T&url=http%3A%2F%2Fmockito.org%2F)
7. [powermock](https://yq.aliyun.com/go/articleRenderRedirect?spm=a2c4e.11153940.0.0.43bf26detjrS1T&url=https%3A%2F%2Fgithub.com%2Fjayway%2Fpowermock)
9. [在Android Studio中进行单元测试和UI测试](https://yq.aliyun.com/go/articleRenderRedirect?spm=a2c4e.11153940.0.0.43bf26detjrS1T&url=http%3A%2F%2Fwww.jianshu.com%2Fp%2F03118c11c199)
10. [Android单元测试只看这一篇就够了](https://www.jianshu.com/p/aa51a3e007e2)
11. [代码覆盖率浅谈](https://www.cnblogs.com/coderzh/archive/2009/03/29/1424344.html?tt_from=copy_link&utm_source=copy_link&utm_medium=toutiao_ios&utm_campaign=client_share)
12. [PowerMock单元测试踩坑与总结](https://cloud.tencent.com/developer/article/1381404)






