---
title: 由Dialog里面嵌套ListView之后的高度自适应引起的ListView性能优化
date: 2018-03-30 13:15:37
tags:
- Android
- 性能优化
- ListView
categories:
- Android开发笔记
---
先说ListView给高的正确做法.
**android:layout_height属性：**

> 必须将ListView的布局高度属性设置为非“wrap_content”（可以是“match_parent /  fill_parent  /  400dp等绝对数值”）

<!--more-->
废话少说先来张bug图填楼

![图片](https://dn-coding-net-production-pp.qbox.me/db684e69-16b3-41af-bd42-fd92506790ce.png)
<!--more-->
# 前言

随着RecyclerView的普及,ListView差不多是安卓快要淘汰的控件了,但是我们有时候还是会用到,基本上可以说是前些年最常用的Android控件之一了.抛开我们的主题,我们先来谈谈ListView的一些小小的细节,可能是很多开发者在开发过程中并没有注意到的细节,这些细节设置会影响到我们的App的性能.

- **android:layout_height属性**

我们在使用ListView的时候很可能随手就会写一个`layout_height=”wrap_content”`或者`layout_height=”match_parent”`,非常非常普通,咋一看,我写的没错啊...可是实际上`layout_height=”wrap_content”` **是错误的写法!!!会严重影响程序的性能** 我们先来做一个实验:
xml布局文件如下
```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <ListView
        android:id="@+id/list_view"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        ></ListView>
</LinearLayout>

```

java部分代码
![图片](https://dn-coding-net-production-pp.qbox.me/917604eb-c83f-468f-a509-e1adcea53472.png)

运行log
![图片](https://dn-coding-net-production-pp.qbox.me/19b0ceaa-ffd6-44d6-80a1-3d4034e5ead5.png)

我们会发现getView总共被调用了15次!其中4次是null的,11次为重复调用,ListView的item数目只有3项!!!**太可怕了**

我们试着将ListView的高度属性改为`layout_height=”match_parent”`,然后看看
![](https://ws4.sinaimg.cn/large/006tNc79ly1fpusaq5hm5j30rs01l0up.jpg)
我们可以看到`getView()`只被调用了3次!这应该是我们期望的结果!

**原因分析:**
了解原因前,我们应该先了解View的绘制流程,之前我的博客没有关于View绘制流程的介绍,那么在这边说一下,是一个很重要的知识点.
View的绘制流程是通过 `onMeasure()`->`onLayout()`->`onDraw()`

`onMeasure()` :主要工作是测量视图的大小.从顶层的父View到子View递归调用measure方法,measure方法又回调onMeasure().

`onLayout`: 主要工作是确定View的位置,进行页面布局.从顶层的父View向子View的递归调用view.layout方法的过程,即父View根据上一步measure子view所得到的布局大小和布局参数,将子view放在合适的位置上

`onDraw()` 主要工作是绘制视图.ViewRoot创建一个Canvas对象,然后调用onDraw()方法.总共6个步骤.1.绘制视图背景,2.保存当前画布的图层(Layer),3.绘制View内容,4.绘制View的子View视图,没有的话就不绘制,5.还原图层,6.绘制滚动条.

了解了View的绘制流程,那么我们回到这个问题上.设置ListView的属性`layout_height=”wrap_content”`,就意味着Listview的高度由子View决定,当在onMeasure()的时候,需要测量子View的高度,那我们来看看Listview的onMeasure()方法.
``` java
@Override
    protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
        // Sets up mListPadding
        super.onMeasure(widthMeasureSpec, heightMeasureSpec);

        int widthMode = MeasureSpec.getMode(widthMeasureSpec);
        int heightMode = MeasureSpec.getMode(heightMeasureSpec);
        int widthSize = MeasureSpec.getSize(widthMeasureSpec);
        int heightSize = MeasureSpec.getSize(heightMeasureSpec);

        int childWidth = 0;
        int childHeight = 0;
        int childState = 0;

        mItemCount = mAdapter == null ? 0 : mAdapter.getCount();
        if (mItemCount > 0 && (widthMode == MeasureSpec.UNSPECIFIED ||
                heightMode == MeasureSpec.UNSPECIFIED)) {
            final View child = obtainView(0, mIsScrap);

            measureScrapChild(child, 0, widthMeasureSpec);

            childWidth = child.getMeasuredWidth();
            childHeight = child.getMeasuredHeight();
            childState = combineMeasuredStates(childState, child.getMeasuredState());

            if (recycleOnMeasure() && mRecycler.shouldRecycleViewType(
                    ((LayoutParams) child.getLayoutParams()).viewType)) {
                mRecycler.addScrapView(child, 0);
            }
        }

        if (widthMode == MeasureSpec.UNSPECIFIED) {
            widthSize = mListPadding.left + mListPadding.right + childWidth +
                    getVerticalScrollbarWidth();
        } else {
            widthSize |= (childState&MEASURED_STATE_MASK);
        }

        if (heightMode == MeasureSpec.UNSPECIFIED) {
            heightSize = mListPadding.top + mListPadding.bottom + childHeight +
                    getVerticalFadingEdgeLength() * 2;
        }

        if (heightMode == MeasureSpec.AT_MOST) {
            // TODO: after first layout we should maybe start at the first visible position, not 0
            heightSize = measureHeightOfChildren(widthMeasureSpec, 0, NO_POSITION, heightSize, -1);
        }

        setMeasuredDimension(widthSize , heightSize);
        mWidthMeasureSpec = widthMeasureSpec;
    }
```

其中
```
if (heightMode == MeasureSpec.AT_MOST) {
            // TODO: after first layout we should maybe start at the first visible position, not 0
            heightSize = measureHeightOfChildren(widthMeasureSpec, 0, NO_POSITION, heightSize, -1);
        }
```
比较重要

再看`measureHeightOfChildren()`
```
final int measureHeightOfChildren(int widthMeasureSpec, int startPosition, int endPosition,
            final int maxHeight, int disallowPartialChildPosition) {

        ...

        for (i = startPosition; i <= endPosition; ++i) {
            child = obtainView(i, isScrap);

            measureScrapChild(child, i, widthMeasureSpec);
            ...

            // Recycle the view before we possibly return from the method
            if (recyle && recycleBin.shouldRecycleViewType(
                    ((LayoutParams) child.getLayoutParams()).viewType)) {
                recycleBin.addScrapView(child, -1);
            }

            returnedHeight += child.getMeasuredHeight();

            if (returnedHeight >= maxHeight) {
                ...
            }

            if ((disallowPartialChildPosition >= 0) && (i >= disallowPartialChildPosition)) {
                ...
            }
        }
        return returnedHeight;
    }
```

`obtainView(i, isScrap)`是子View的实例
`measureScrapChild(child, i, widthMeasureSpec);` 测量子View
`recycleBin.addScrapView(child, -1);`将子View加入缓存,可以用来复用
`if (returnedHeight >= maxHeight) {return ...;}`如果已经测量的子View的高度大于maxHeight的话就直接return出循环,这样的做法也很好理解,其实是ListView很聪明的一种做法,你可以想想比如说这个屏幕只能画10个Item高度,你有20个Item,那么画出10个就行了,剩下的十个就没必要画了~

我们现在看下`obtainView()`方法

```
View obtainView(int position, boolean[] isScrap) {
        Trace.traceBegin(Trace.TRACE_TAG_VIEW, "obtainView");

        isScrap[0] = false;

        // Check whether we have a transient state view. Attempt to re-bind the
        // data and discard the view if we fail.
        final View transientView = mRecycler.getTransientStateView(position);
        if (transientView != null) {
            final LayoutParams params = (LayoutParams) transientView.getLayoutParams();

            // If the view type hasn't changed, attempt to re-bind the data.
            if (params.viewType == mAdapter.getItemViewType(position)) {
                final View updatedView = mAdapter.getView(position, transientView, this);

                // If we failed to re-bind the data, scrap the obtained view.
                if (updatedView != transientView) {
                    setItemViewLayoutParams(updatedView, position);
                    mRecycler.addScrapView(updatedView, position);
                }
            }

            // Scrap view implies temporary detachment.
            isScrap[0] = true;
            return transientView;
        }

        final View scrapView = mRecycler.getScrapView(position);
        final View child = mAdapter.getView(position, scrapView, this);
        if (scrapView != null) {
            if (child != scrapView) {
                // Failed to re-bind the data, return scrap to the heap.
                mRecycler.addScrapView(scrapView, position);
            } else {
                isScrap[0] = true;

                child.dispatchFinishTemporaryDetach();
            }
        }

        if (mCacheColorHint != 0) {
            child.setDrawingCacheBackgroundColor(mCacheColorHint);
        }

        if (child.getImportantForAccessibility() == IMPORTANT_FOR_ACCESSIBILITY_AUTO) {
            child.setImportantForAccessibility(IMPORTANT_FOR_ACCESSIBILITY_YES);
        }

        setItemViewLayoutParams(child, position);

        if (AccessibilityManager.getInstance(mContext).isEnabled()) {
            if (mAccessibilityDelegate == null) {
                mAccessibilityDelegate = new ListItemAccessibilityDelegate();
            }
            if (child.getAccessibilityDelegate() == null) {
                child.setAccessibilityDelegate(mAccessibilityDelegate);
            }
        }

        Trace.traceEnd(Trace.TRACE_TAG_VIEW);

        return child;
    }
```


得到一个视图,它显示的数据与指定的位置。这叫做当我们已经发现的观点不是可供重用的回收站。剩下的唯一的选择是将一个古老的视图或制作一个新的.(这是方法注释的翻译,大致可以理解他的意思)

我们应该关注下以下两行代码:
```
...
  final View scrapView = mRecycler.getScrapView(position);
  final View child = mAdapter.getView(position, scrapView, this);
...
```
这两行代码的意思就是说先从缓存里面取出来一个废弃的view,然后将当前的位置跟view作为参数传入到getView()方法中.这个废弃的,然后又作为参数的view就是convertView.


然后我们总结下刚刚的步骤:
A、测量第0项的时候，convertView肯定是null的 View scrapView = mRecycler.getScrapView(position)也是空的,所以我们在log上可以看到.
![](https://ws2.sinaimg.cn/large/006tNc79ly1fpusdjl9loj30iq00qaaa.jpg)
B、第0项测量结束,这个第0项的View就被加入到复用缓存当中了；
C、开始测量第1项，这时因为是有第0项的View缓存的，所以getView的参数convertView就是这个第0项的View缓存，然后重复B步骤添加到缓存，只不过这个View缓存还是第0项的View；
D、继续测量第2项，重复C。

所以前面说到onMeasure方法会导致getView调用，而一个View的onMeasure方法调用时机并不是由自身决定，而是由其父视图来决定。ListView放在FrameLayout和RelativeLayout中其onMeasure方法的调用次数是完全不同的。在RelativeLayout中oMeasure()方法调用会翻倍.

由于onMeasure方法会多次被调用，上述问题中是两次，其实完整的调用顺序是onMeasure - onLayout - onMeasure - onLayout - onDraw。

所以根据上面的结论我们可以得出,如果LitsView的**android:layout_height属性设置为wrap_content**将会引起`getView的多次测量`

# 现象
如上bug图...
# 产生的原因

- ListView的高度设置成了android:layout_height属性设置为wrap_content

- ListView的父类是RelativeLayout,RelativiLayout布局会使子布局View的Measure周期翻倍,有兴趣可以看下[三大基础布局性能比较](http://blog.csdn.net/megatronkings/article/details/52270461)

# 解决办法
根据每个Item的高度,然后再根据Adapter的count来动态算高.
代码如下:
```
public class SetHeight {

    public void setListViewHeightBasedOnChildren(ListView listView, android.widget.BaseAdapter adapter) {

        if (adapter==null){
            return;
        }
        int totalHeight = 0;

        for (int i = 0; i < adapter.getCount(); i++) { // listAdapter.getCount()返回数据项的数目

            View listItem = adapter.getView(i, null, listView);

            listItem.measure(0, 0); // 计算子项View 的宽高

            totalHeight += listItem.getMeasuredHeight(); // 统计所有子项的总高度

        }

        ViewGroup.LayoutParams params = listView.getLayoutParams();

        params.height = totalHeight
                + (listView.getDividerHeight() * (adapter.getCount() - 1));

        // listView.getDividerHeight()获取子项间分隔符占用的高度

        // params.height最后得到整个ListView完整显示需要的高度

        listView.setLayoutParams(params);

    }

}

```


xml布局,**注意要将ListView的父类设置为LinearLayout**
```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="fill_parent"
    android:layout_height="wrap_content"
    android:orientation="vertical">

    <LinearLayout
        android:layout_width="fill_parent"
        android:layout_height="wrap_content"
        android:layout_above="@+id/txt_cancel"
        android:orientation="vertical">
        <View
            android:layout_width="fill_parent"
            android:layout_height="@dimen/y2"
            android:background="#cccccc" />

        <ListView
            android:id="@+id/lv_remain_item"
            android:layout_width="fill_parent"
            android:layout_height="0dp"
            android:cacheColorHint="#00000000"
            ></ListView>

        <View
            android:layout_width="fill_parent"
            android:layout_height="@dimen/y2"
            android:background="#cccccc" />

    </LinearLayout>

    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:gravity="center"
        android:orientation="horizontal"
        >

        <TextView
            android:id="@+id/txt_cancel"
            android:layout_width="fill_parent"
            android:layout_height="@dimen/y120"
            android:layout_alignParentBottom="true"
            android:gravity="center"
            android:text="cancel"
            android:textSize="@dimen/x32" />
    </LinearLayout>
</LinearLayout>
```

然后在Listview使用处,调用该方法.
```
 userListDialog.getmListView().setAdapter(scaleUserAdapter);
 SetHeight.setListViewHeightBasedOnChildren(userListDialog.getmListView(),scaleUserAdapter);
```

# 运行结果
![](https://ws3.sinaimg.cn/large/006tNc79ly1fpusfb8d48j308c0etjsh.jpg)

getView()调用情况
![](https://ws1.sinaimg.cn/large/006tNc79ly1fpusgaa5ixj30rs082tl8.jpg)
GitHub代码地址:[ListViewDialog](https://github.com/pvphero/ListViewDialog),喜欢的话欢迎Start