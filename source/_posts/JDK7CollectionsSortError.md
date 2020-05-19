---
title: JDK7 Collections.sort()方法报错分析
date: 2016-12-06 16:53:38
tags:
- Java
---

java.lang.IllegalArgumentException: Comparison method violates its general contract!
<!--more-->

# 问题背景
## 起因
前些天测试给提了一个项目里的bug,在查看项目的一个在线数据的列表的时候发生了崩溃.然后我根据bugly定位发现是在使用Collection.sort()对list排序的时候产生Comparison method violates its general contract异常.但是Collection.sort()在JDK1.6中并没有出现过这样的异常啊...
# 问题定位
首先我怀着很纳闷的心情看了下bugly的崩溃日志.有这么一段日志信息

``` java

11-23 14:02:55.357 27729 32217 W System.err: java.lang.IllegalArgumentException: Comparison method violates its general contract!
11-23 14:02:55.357 27729 32217 W System.err: at java.util.TimSort.mergeLo(TimSort.java:743)
11-23 14:02:55.357 27729 32217 W System.err: at java.util.TimSort.mergeAt(TimSort.java:479)
11-23 14:02:55.357 27729 32217 W System.err: at java.util.TimSort.mergeCollapse(TimSort.java:406)
11-23 14:02:55.357 27729 32217 W System.err: at java.util.TimSort.sort(TimSort.java:210)
11-23 14:02:55.357 27729 32217 W System.err: at java.util.TimSort.sort(TimSort.java:169)
11-23 14:02:55.357 27729 32217 W System.err: at java.util.Arrays.sort(Arrays.java:2010)
11-23 14:02:55.358 27729 32217 W System.err: at java.util.Collections.sort(Collections.java:1883)


```

然后我定位到出错的代码

``` java
if (list != null && list.size() > 0){
        //降序排序
        Collections.sort(list, new Comparator<Spo2Result>() {
            @Override
            public int compare(Spo2Result lhs, Spo2Result rhs) {
                return lhs.getMeasureTime()>rhs.getMeasureTime() ? -1 : 1;
            }

        });
    }
```

抛出异常的地方是对一个list按照测量时间降序排序,逻辑很简单.然后我仔仔细细前前后后里里外外的看了好几遍这段代码,为了验证代码的可靠性,我还特意z针对这个排序方法写了一个测试案例,如下:

``` java
import java.util.ArrayList;
import java.util.Collections;
import java.util.Comparator;
import java.util.List;

public class TestSortError {

	/**
	 * @param args
	 */
	public static void main(String[] args) {
		// TODO Auto-generated method stub
		List<Integer> l = new ArrayList<Integer>();
		l.add(1);
		l.add(3);
		l.add(2);
		l.add(2);
		l.add(2);
		l.add(2);
		l.add(5);
		l.add(6);
		l.add(4);
		l.add(1);
		testSort(l);
	}

	private static void testSort(List<Integer> l) {

		Collections.sort(l, new Comparator<Integer>() {

			public int compare(Integer o1, Integer o2) {
				// TODO Auto-generated method stub
				return o1 > o2 ? -1 : 1;
			}
		});

		for (int i = 0; i < l.size(); i++) {
			System.out.println(l.get(i));
		}

	}
}

```

然后输出:

    6
    5
    4
    3
    2
    2
    2
    2
    1
    1


# 问题分析
并没有出现自己脑海中的崩溃现象...于是只能查这个异常,找到了一个有关[JDK不兼容的声明](http://www.oracle.com/technetwork/java/javase/compatibility-417013.html).

![这里写图片描述](http://img.blog.csdn.net/20161206175819540?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcHZwaGVyb3N6dw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

这个的意思大概是说JDK比较器的排序算法更换了实现方法,新的比较器在违背规则的情况下有可能抛出异常.看到这个就可以确定大概的原因了,因为项目中的排序代码违背了比较器的规则.

于是google了一下比较器的规则:
> Compares its two arguments for order. Returns a negative integer, zero, or a positive integer as the first argument is less than, equal to, or greater than the second.In the foregoing description, the notation sgn(expression) designates the mathematical signum function, which is defined to return one of -1, 0, or 1 according to whether the value ofexpression is negative, zero or positive. The implementor must ensure that sgn(compare(x, y)) == -sgn(compare(y, x)) for all x and y. (This implies that compare(x, y) must throw an exception if and only if compare(y, x) throws an exception.) The implementor must also ensure that the relation is transitive: ((compare(x, y)>0) && (compare(y, z)>0)) implies compare(x, z)>0. Finally, the implementor must ensure that compare(x, y)==0 implies that sgn(compare(x, z))==sgn(compare(y, z)) for all z. It is generally the case, but not strictly required that (compare(x, y)==0) == (x.equals(y)). Generally speaking, any comparator that violates this condition should clearly indicate this fact. The recommended language is “Note: this comparator imposes orderings that are inconsistent with equals.”

即满足三个规则:
1. 确保：sgn(compare(x, y)) == -sgn(compare(y, x)).
2. 确保：如果((compare(x, y)>0) && (compare(y, z)>0))，那么compare(x, z)>0.
3. 确保：如果compare(x, y)==0，那么对于任意的z都有sgn(compare(x, z))==sgn(compare(y, z))成立.

# 问题解决
代码中出现的三目运算符明显违背了比较器规则,所以把代码稍微改了下,增加了相等情况的判断.

``` java
if (list != null && list.size() > 0){
        //降序排序
        Collections.sort(list, new Comparator<Spo2Result>() {
            @Override
            public int compare(Spo2Result lhs, Spo2Result rhs) {
                return lhs.getMeasureTime()==rhs.getMeasureTime()
                        ?0:(lhs.getMeasureTime()>rhs.getMeasureTime() ? -1 : 1);
            }

        });
    }
```
# 总结
使用三目运算符要注意一定要判断相等的情况.否则会违背比较器的规则.



