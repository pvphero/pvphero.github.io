---
title: 重学Java--Java泛型
date: 2020-05-18 17:08:54
tags:
- Java
categories:
- Java学习笔记
---

# 1.为什么我们需要泛型？


通过两端代码我们就可以知道为什么需要泛型


```java
 public int addInt(int x,int y){
        return x+y;
    }

    public float addFloat(float x,float y){
        return x+y;
    }
```

<!--more-->

以上例子，求两个数的和。现在已经有int类型的求和跟float类型的求和，但是如果要实现一个double类型的和，就需要重新写一个**double**的**add**方法。如下：


```java
    //double
    public double addDouble(double x,double y){
        return x+y;
    }
```


其实对于开发者来说，逻辑是一样的，只是参数不同，如果没有泛型，就需要重写类似的方法。




所以泛型的好处：
**适用于多种数据类型执行相同的代码。可以简化代码。**


# 2.泛型类，泛型接口，泛型方法


定义一个自己的泛型:


## 1.泛型类


```java
public class NormalGeneric<K> {
    private K data;

    public NormalGeneric() {
    }

    public NormalGeneric(K data) {
        this.data = data;
    }

    public K getData() {
        return data;
    }

    public void setData(K data) {
        this.data = data;
    }

    public static void main(String[] args) {
        NormalGeneric<String> normalGeneric = new NormalGeneric<>();
        normalGeneric.setData("OK");
        //normalGeneric.setData(1);
        System.out.println(normalGeneric.getData());
        NormalGeneric normalGeneric1 = new NormalGeneric();
        normalGeneric1.setData(1);
        normalGeneric1.setData("dsf");
    }
}
```


## 2.泛型接口


```java
public interface Genertor<T> {
    public T next();
}
```


实现泛型接口的实现类。


- 实现方式一：

    ```java
    public class ImplGenertor<T> implements Genertor<T> {
        @Override
        public T next() {
            return null;
        }
    }
    ```

- 实现方式二：

    ```java
     public class ImplGenertor2 implements Genertor<String> {
            @Override
            public String next() {
                return null;
            }
        }
    ```


可见，如果是泛型类实现泛型接口的话，那么返回值也是泛型。


## 3.泛型方法


泛型方法是完全独立的


### 3.1 在普通类中的泛型方法代码如下：


```java
public class GenericMethod {

    public <T> T genericMethod(T...a){
        return a[a.length/2];
    }

    public void test(int x,int y){
        System.out.println(x+y);
    }

    public static void main(String[] args) {
        GenericMethod genericMethod = new GenericMethod();
        genericMethod.test(23,343);
        System.out.println(genericMethod.<String>genericMethod("apple","banana"));
        System.out.println(genericMethod.genericMethod(12,34));
    }
}
```


### 3.2 在泛型类里面使用泛型方法：


```java
public class GenericMethod2 {
    //这个类是个泛型类，在上面已经介绍过
    public class Generic<T>{
        private T key;
        public Generic(T key) {
            this.key = key;
        }
        public T getKey(){
            return key;
        }
    }
    
    public <T,K> K showKeyName(Generic<T> container){
     ...
    }
    
    public static void main(String[] args) {


    }
}
```


上述代码中,


```java
public T getKey(){
	return key;
}
```


虽然在方法中使用了泛型，但是这并不是一个泛型方法。
这只是类中一个普通的成员方法，只不过他的返回值是在声明泛型类已经声明过的泛型。
所以在这个方法中才可以继续使用 T 这个泛型。


```java
 public <T,K> K showKeyName(Generic<T> container){
     ...
 }
```


这才是一个真正的泛型方法。


### 小结1：


- 首先在public与返回值之间的必不可少，这表明这是一个泛型方法，
- 并且声明了一个泛型T_ 这个T可以出现在这个泛型方法的任意位置._
- 泛型的数量也可以为任意多个



再看个代码示例：


```java
public class GenericMethod3 {
    static class Fruit{
        @Override
        public String toString() {
            return "fruit";
        }
    }

    static class Apple extends Fruit{
        @Override
        public String toString() {
            return "apple";
        }
    }

    static class Person{
        @Override
        public String toString() {
            return "Person";
        }
    }

    static class GenerateTest<T>{
        //普通方法
        public void show_1(T t){
            System.out.println(t.toString());
        }

        //在泛型类中声明了一个泛型方法，使用泛型E，这种泛型E可以为任意类型。
        // 可以类型与T相同，也可以不同。
        //由于泛型方法在声明的时候会声明泛型<E>，因此即使在泛型类中并未声明泛型，
        // 编译器也能够正确识别泛型方法中识别的泛型。
        public <E> void show_3(E t){
            System.out.println(t.toString());
        }

        //在泛型类中声明了一个泛型方法，使用泛型T，
        // 注意这个T是一种全新的类型，可以与泛型类中声明的T不是同一种类型。
        public <T> void show_2(T t){
            System.out.println(t.toString());
        }
    }

    public static void main(String[] args) {
        Apple apple = new Apple();
        Person person = new Person();

        GenerateTest<Fruit> generateTest = new GenerateTest<>();
        generateTest.show_1(apple);
        //generateTest.show_1(person);

        generateTest.show_2(apple);
        generateTest.show_2(person);

        generateTest.show_3(apple);
        generateTest.show_3(person);
    }
}
```


![Xnip2020-04-24_13-23-18.jpg](https://cdn.nlark.com/yuque/0/2020/jpeg/1261993/1587706787742-1eb20c62-3cf1-452f-9092-2f97107f9062.jpeg#align=left&display=inline&height=353&margin=%5Bobject%20Object%5D&name=Xnip2020-04-24_13-23-18.jpg&originHeight=646&originWidth=1364&size=206580&status=done&style=none&width=746#align=left&display=inline&height=646&margin=%5Bobject%20Object%5D&originHeight=646&originWidth=1364&status=done&style=none&width=1364)


### 小结2：


泛型类里面定义泛型方法，参数可以完全不一样。
上面的T影响泛型类的普通方法。
但是对泛型方法没有影响。


# 3.如何限定类型变量


比如计算两个变量的最小值，最大值。


![屏幕快照 2020-04-24 下午1.55.12.png](https://cdn.nlark.com/yuque/0/2020/png/1261993/1587707736212-76da455f-c265-4eb0-b1dc-c547d3878607.png#align=left&display=inline&height=283&margin=%5Bobject%20Object%5D&name=%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202020-04-24%20%E4%B8%8B%E5%8D%881.55.12.png&originHeight=328&originWidth=864&size=37264&status=done&style=none&width=746#align=left&display=inline&height=328&margin=%5Bobject%20Object%5D&originHeight=328&originWidth=864&status=done&style=none&width=864)
但是怎么才能保证传入的变量一定有compareTo方法？
为了解决这个问题，就引入了限定类型变量的泛型。
代码如下：


```java
 public static <T extends Comparable> T min(T a, T b){
        if(a.compareTo(b)>0) {
            return a;
        } else {
            return b;
        }
}
```


**T extends Comparable中**
T表示应该绑定类型的子类型，后面的**Comparable**表示绑定类型，
**子类型和绑定类型可以是类也可以是接口**。


**但是如果类跟接口混用的话，规则如下：**


- 类只能有一个
- 并且类需要写到接口的前面
- 限定类型的泛型，对泛型类，泛型方法同样适用



代码如下：


```java
 public static <T extends ArrayList & Comparable> T min(T a, T b) {
        if (a.compareTo(b) > 0) return a;
        else return b;
 }
```


# 4.泛型使用中的约束和局限性


假设我们有以下泛型类：


```java
public class Restrict<T> {
    ...
}
```


- 不能用基本类型实例化类型参数

```java
//错误代码示例
//不能实例化类型变量 
public Restrict() {
    this.data = new T();
}
```

- 在静态域或者方法里面不能引用类型变量

```java
//错误代码示例
//静态域或者方法里不能引用类型变量
private static T instance;
```

  - 先执行static方法
  - 再执行构造方法
  - 所以先执行static T，根本不知道T的类型，所以行不通
  - 但是如果是静态方法本身是泛型方法是可以的
  
```java
//正确代码示例
private static <T> T getInstance(){}
```

- 基础类型不允许做实例化参数的，例如：double不能用在泛型参数里，必须用double的包装类Double

错误的写法：

```java
     //错误代码示例
    Restrict<double>
```

- 正确的写法：

```java
//正确代码示例
Restrict<Double>
```

- 泛型里面不允许使用**instanceof**

![屏幕快照 2020-04-28 下午2.44.25.png](https://cdn.nlark.com/yuque/0/2020/png/1261993/1588056330113-c11992b1-760c-42af-9833-857462135795.png#align=left&display=inline&height=168&margin=%5Bobject%20Object%5D&name=%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202020-04-28%20%E4%B8%8B%E5%8D%882.44.25.png&originHeight=168&originWidth=1088&size=42395&status=done&style=none&width=1088#align=left&display=inline&height=168&margin=%5Bobject%20Object%5D&originHeight=168&originWidth=1088&status=done&style=none&width=1088)
- 泛型中可以定义反省数组，但是不能创建参数化类型的数组

![屏幕快照 2020-04-28 下午2.52.27.png](https://cdn.nlark.com/yuque/0/2020/png/1261993/1588056771534-217f2090-b2ff-49f3-8c44-152cfa9e536e.png#align=left&display=inline&height=202&margin=%5Bobject%20Object%5D&name=%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202020-04-28%20%E4%B8%8B%E5%8D%882.52.27.png&originHeight=202&originWidth=1112&size=51695&status=done&style=none&width=1112#align=left&display=inline&height=202&margin=%5Bobject%20Object%5D&originHeight=202&originWidth=1112&status=done&style=none&width=1112)
- 不能捕获泛型类的实例

![屏幕快照 2020-04-28 下午3.47.05.png](https://cdn.nlark.com/yuque/0/2020/png/1261993/1588060045684-c74ed072-4475-4c37-9f3f-41f889e3a328.png#align=left&display=inline&height=330&margin=%5Bobject%20Object%5D&name=%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202020-04-28%20%E4%B8%8B%E5%8D%883.47.05.png&originHeight=330&originWidth=1154&size=56276&status=done&style=none&width=1154#align=left&display=inline&height=330&margin=%5Bobject%20Object%5D&originHeight=330&originWidth=1154&status=done&style=none&width=1154)


但是如果把异常抛出来，是可以，如下：

![屏幕快照 2020-04-28 下午3.47.58.png](https://cdn.nlark.com/yuque/0/2020/png/1261993/1588060109008-f7eede8f-e412-456b-a28e-95b5c9fdb4a0.png#align=left&display=inline&height=330&margin=%5Bobject%20Object%5D&name=%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202020-04-28%20%E4%B8%8B%E5%8D%883.47.58.png&originHeight=330&originWidth=1000&size=40351&status=done&style=none&width=1000#align=left&display=inline&height=330&margin=%5Bobject%20Object%5D&originHeight=330&originWidth=1000&status=done&style=none&width=1000)



### 小结：


1. 不能用基本类型实例化类型参数。
1. 在静态域或者方法里面不能引用类型变量。
1. 基础类型不允许做实例化参数的，例如：double不能用在泛型参数里，必须用double的包装类Double。
1. 泛型里面不允许使用**instanceof。**
1. 泛型中可以定义反省数组，但是不能创建参数化类型的数组。
1. 不能捕获泛型类的实例。



# 5.泛型类型能继承吗？


例子如下：


```java
public class Employee {
    ....
}

public class Worker extends Employee {
}

public class Pair<T> {
    ...
}
```


```java
Pair<Employee> employeePair2 = new Pair<Worker>();
```


会报错。
![image.png](https://cdn.nlark.com/yuque/0/2020/png/1261993/1588061003533-d816417c-cef0-4507-ad47-96844d2e1a79.png#align=left&display=inline&height=597&margin=%5Bobject%20Object%5D&name=image.png&originHeight=404&originWidth=1358&size=80794&status=done&style=none&width=1358#align=left&display=inline&height=404&margin=%5Bobject%20Object%5D&originHeight=404&originWidth=1358&status=done&style=none&width=1358)
但是，泛型类是可以继承或者扩展其他类的！


```java
 /*泛型类可以继承或者扩展其他泛型类，比如List和ArrayList*/
    private static class ExtendPair<T> extends Pair<T>{

    }
```


### 小结


1. 泛型类无法协变的
1. 泛型可以继承或者扩展其他的泛型类



# 6.泛型中的通配符类型


现有如下类的派生关系：


![屏幕快照 2020-05-13 下午1.55.14.png](https://cdn.nlark.com/yuque/0/2020/png/1261993/1589349360860-1646886c-a618-4c0a-80d7-7fe2478cfb2c.png#align=left&display=inline&height=820&margin=%5Bobject%20Object%5D&name=%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202020-05-13%20%E4%B8%8B%E5%8D%881.55.14.png&originHeight=820&originWidth=1122&size=42212&status=done&style=none&width=1122#align=left&display=inline&height=820&margin=%5Bobject%20Object%5D&originHeight=820&originWidth=1122&status=done&style=none&width=1122)


代码如下:


```java
//我们有如下方法和泛型类
public static void print(GenericType<Fruit> p) {
    System.out.println(p.getData().getColor());
}

public class GenericType<T> {
    private T data;

    public T getData() {
        return data;
    }

    public void setData(T data) {
        this.data = data;
    }
}
```


```java
public class Food {
    ...
}

public class Fruit extends Food {
  ...
}

public class Apple extends Fruit {
    ...
}

public class Orange extends Fruit {
    ...
}

public class HongFuShi extends Apple {
    ...
}
```


但是使用的时候，`print(b)`是不允许的。正如之前提到的，`Orange`虽然是派生自`Fruit`，但是`GenericType<Orange>`和`GenericType<Fruit>`是没有任何关系的。


```java
	public static void use() {
        GenericType<Fruit> a = new GenericType<>();
        print(a);
        GenericType<Orange> b = new GenericType<>();
        //print(b); 不允许
    }
```


所以为了解决**泛型无法协变的问题的**的问题，就引入了泛型通配符的概念。


> 1.协变指的就是Orange派生自Fruit，那么List也派生自List,但是泛型是不支持的。
> 2.泛型T是确定的参数类型，一旦传了就定下来了，而通配符非常的灵活是不确定的类型，更多的情况用于扩充参数范围。
> 3.通配符不是类型参数变量，通配符更像一种规定，规定你只能传哪些类型的参数。



就上述代码，我们想要**print(b),**可以这么做:


```java
    public static void print2(GenericType<? extends Fruit> p) {
        System.out.println(p.getData().getColor());
    }
```


```java
	public static void use2() {
        GenericType<Fruit> a = new GenericType<>();
        print2(a);
        GenericType<Orange> b = new GenericType<>();
        print2(b);
    }
```


这里的**？**就是通配符。


## 6.1 上界通配符<? extends T>


利用 `<? extends T>` 形式的通配符，可以实现泛型的向上转型。


```java
? extends Fruit 表示通配符的上界是Fruit
```


![屏幕快照 2020-05-13 下午1.55.14 2.png](https://cdn.nlark.com/yuque/0/2020/png/1261993/1589362208227-dd08601c-ce8b-4e81-826c-e33c403aeb5b.png#align=left&display=inline&height=820&margin=%5Bobject%20Object%5D&name=%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202020-05-13%20%E4%B8%8B%E5%8D%881.55.14%202.png&originHeight=820&originWidth=1122&size=41069&status=done&style=none&width=1122#align=left&display=inline&height=820&margin=%5Bobject%20Object%5D&originHeight=820&originWidth=1122&status=done&style=none&width=1122)


> `GenericType<? extends Fruit>` 代表一个可以持有`Fruit`  及其子类（如：Apple，Orange）的实例的GenericTyp对象。



### 6.1.1 如果`？`是`Fruit`的父类，会怎样？


![屏幕快照 2020-05-13 下午3.30.36.png](https://cdn.nlark.com/yuque/0/2020/png/1261993/1589355054634-b2bc98b9-0422-4cc2-b895-24361ef12265.png#align=left&display=inline&height=388&margin=%5Bobject%20Object%5D&name=%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202020-05-13%20%E4%B8%8B%E5%8D%883.30.36.png&originHeight=388&originWidth=1762&size=117503&status=done&style=none&width=1762#align=left&display=inline&height=388&margin=%5Bobject%20Object%5D&originHeight=388&originWidth=1762&status=done&style=none&width=1762)


### 6.1.2 上界只能外围取，不能往里放


`GenericType<T>` 方法中有`getData()`和`setData()`方法。


```java
public class GenericType<T> {
    private T data;

    public T getData() {
        return data;
    }

    public void setData(T data) {
        this.data = data;
    }
}
```


代码调用如下：


```java
	public static void use2() {
        GenericType<? extends Fruit> c = new GenericType<>();
        Apple apple = new Apple();
        Fruit fruit = new Fruit();
        //c.setData(apple);//Error
        //c.setData(fruit);//Error
        Fruit x = c.getData();
    }
```


![屏幕快照 2020-05-13 下午4.16.48.png](https://cdn.nlark.com/yuque/0/2020/png/1261993/1589357826694-927eada9-2aef-4922-84d1-21061a84d1d3.png#align=left&display=inline&height=466&margin=%5Bobject%20Object%5D&name=%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202020-05-13%20%E4%B8%8B%E5%8D%884.16.48.png&originHeight=466&originWidth=1412&size=91510&status=done&style=none&width=1412#align=left&display=inline&height=466&margin=%5Bobject%20Object%5D&originHeight=466&originWidth=1412&status=done&style=none&width=1412)


> 我们发现往水果里面设置水果类型的方法`setData()` 会失效，但是获取某种水果类型的`getData()` 方法还有效。



原因：
`? extends T` 表示类型的上界，类型参数是T的子类或者他本身，那么可以肯定的说，get方法返回的一定是T，这个编译器是可以确定的。但是set方法只知道传入的是个T，具体是T还是T的哪个子类，编译器不能确定。


> Java编译期只知道容器里面存放的是Fruit或者是Fruit的派生类，具体是什么类型，编译器并不知道。当编译器执行到 `c.setData(apple)` ，`GenericType` 并没有将值设置成apple，而是标记了一个占位符`capture #1` ,用来表示
> 编译器捕获到一个Fruit类或者他的派生类，具体什么类型不知道。所以在`setdata()` 的时候传入的Apple和Fruit，编译器不确定是否能跟之前标记的`capture #1` 匹配。
> `getData()` 方法，这个可以正常用其实就很好理解了，因为上界通配符只能往容器里面放Fruit类或者他的派生类，所以获取到的类都可以隐式转换为他们的基类(或者Object基类)



## 6.2 下界通配符<? super T>


下界通配符只能往容器中放T或者T的基类类型的数据。`<? super Apple>`


![屏幕快照 2020-05-13 下午1.55.14 2 2.png](https://cdn.nlark.com/yuque/0/2020/png/1261993/1589362628740-a06ccaa9-fa95-4e0c-94d4-19187d8ed710.png#align=left&display=inline&height=820&margin=%5Bobject%20Object%5D&name=%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202020-05-13%20%E4%B8%8B%E5%8D%881.55.14%202%202.png&originHeight=820&originWidth=1122&size=75592&status=done&style=none&width=1122#align=left&display=inline&height=820&margin=%5Bobject%20Object%5D&originHeight=820&originWidth=1122&status=done&style=none&width=1122)


代码如下：


```java
	public static void printSuper(GenericType<? super Apple> p) {
        System.out.println(p.getData());
    }
```


### 6.2.1 下界范围


```java
	public static void useSuper() {
        GenericType<Fruit> fruitGenericType = new GenericType<>();
        GenericType<Apple> appleGenericType = new GenericType<>();
        GenericType<HongFuShi> hongFuShiGenericType = new GenericType<>();
        GenericType<Orange> orangeGenericType = new GenericType<>();
        printSuper(fruitGenericType);
        printSuper(appleGenericType);
//        printSuper(hongFuShiGenericType); //Error
//        printSuper(orangeGenericType);    //Error
    }
```


`printSuper(hongFuShiGenericType)`和
`printSuper(orangeGenericType)`就会在编译期报错，因为超过了通配符的下界。


![屏幕快照 2020-05-13 下午5.47.00.png](https://cdn.nlark.com/yuque/0/2020/png/1261993/1589363237036-71cab071-3e00-45f3-8402-2a0e8c48bb04.png#align=left&display=inline&height=684&margin=%5Bobject%20Object%5D&name=%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202020-05-13%20%E4%B8%8B%E5%8D%885.47.00.png&originHeight=684&originWidth=1762&size=224978&status=done&style=none&width=1762#align=left&display=inline&height=684&margin=%5Bobject%20Object%5D&originHeight=684&originWidth=1762&status=done&style=none&width=1762)


### 6.2.2 下界不影响往里存，但往外取只能放在Object 对象里


同样`GenericType<T>` 方法中有`getData()`和`setData()`方法。


```java
public class GenericType<T> {
    private T data;

    public T getData() {
        return data;
    }

    public void setData(T data) {
        this.data = data;
    }
}
```


代码调用如下：


```java
 	public static void useSuper() {
        //表示GenericType的类型参数的下界是Apple
        GenericType<? super Apple> x = new GenericType<>();
        x.setData(new Apple());
        x.setData(new HongFuShi());
        //x.setData(new Fruit()); // Error
        Object data = x.getData();

    }
```


![屏幕快照 2020-05-13 下午6.01.09.png](https://cdn.nlark.com/yuque/0/2020/png/1261993/1589364251417-89dc052b-db4b-4c79-a722-4bfcf35a3110.png#align=left&display=inline&height=544&margin=%5Bobject%20Object%5D&name=%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202020-05-13%20%E4%B8%8B%E5%8D%886.01.09.png&originHeight=544&originWidth=1352&size=116070&status=done&style=none&width=1352#align=left&display=inline&height=544&margin=%5Bobject%20Object%5D&originHeight=544&originWidth=1352&status=done&style=none&width=1352)


我们发现`setData()`是可以正常调用的，`getDat()`方法返回了Object对象。但是为什么`setData()`的时候传入了Apple的父类，就会报错呢？以及为什么`getData()`只能用object来接收呢?


原因：


> `? super T` ,表示的类型的下界，也就是说表示的是T的基类。所以我们实际上是不知道这个类到底是什么，但是肯定是T的超类或者他本身。
> 1. 因此我们使用`x.setData()`的时候，如果set的类是T的子类或者T，那么他们可以安全的向上转型为T。所以我们`x.setData()`可以放T的子类或者T本身。对于T的基类，编译器并不知道这个类对象是否是安全的，所以在使用下界通配符的时候是不能直接添加T的基类。
> 1. 我们在使用`x.getData()`的时候，返回的值一定是T的超类或者他本身，但是要转成哪个超类，编译器不知道，唯一可以确定的，Object是所有类的基类，所以只有转成Object才是安全的。这就是我们在使用`getData()`的时候必须使用Object来接收原因。



### 6.3 PECS


> PECS:_Producer extends Consumer super （生产者使用extends，consumer使用super）_



- 上界<? extends T>不能往里存，只能往外取(不能set，只能get)，适合频繁往外面读取内容的场景。
- 下界<? super T>不影响往里存，但往外取只能放在Object对象里，适合经常往里面插入数据的场景。



# 7.虚拟机如何实现泛型？


> 泛型是JDK 1.5的一项新增特性，它的本质是参数化类型（Parametersized Type）的应用，Java中的泛型基本上都是在编译器这个层次来实现的。在生成的Java字节码中是不包含泛型中的类型信息的。使用泛型的时候加上的类型参数，会在编译器在编译的时候去掉。这个过程就称为类型擦除。



看下面一个简单的泛型例子：
例子1:
我们有如下类：


```java
public class Apple extends Fruit {
	...
}
```


```java
public class Orange extends Fruit {
	...
}
```


```java
	public static void main(String[] args) {
        ArrayList<Apple> apples = new ArrayList<>();
        ArrayList<Orange> orange = new ArrayList<>();
        System.out.println(apples.getClass() == orange.getClass());
    }
```


上述例子中，apples只能存`Apple`，orange中只能存`Orange`，最好我们通过`xxx.getClass()`方法来获取他们的类信息，最后结果发现为true。说明泛型类型`Apple`和`Orange`都被擦除了，只剩下**原始类型**。


例子2：


```java
 	public static void main(String[] args) {
        Map<String,String> map = new HashMap<>();
        map.put("Roy","18");
        System.out.println(map.get("Roy"));
    }
```


把这段Java代码编译成Class文件，然后字节码反编译一下，发现所有的泛型都不见了。
`javac xxxx.java`


```java
  	public static void main(String[] var0) {
        HashMap var1 = new HashMap();
        var1.put("Roy", "18");
        System.out.println((String)var1.get("Roy"));
    }
```


待续。。。






