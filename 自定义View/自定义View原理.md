来自Carson_Ho的[自定义View系列文章](https://www.jianshu.com/p/146e5cec4863)



# 一、知识储备

## 1.1 基础概念



###1、视图（View）定义

视图（View）表现为显示在屏幕上的各种视图，如TextView、LinearLayout等。



###2. 视图（View）分类

- 视图View主要分为两类：

| 类别     |                   解释                    |         特点 |
| -------- | :---------------------------------------: | -----------: |
| 单一视图 |          即一个View，如TextView           | 不包含子View |
| 视图组   | 即多个View组成的ViewGroup，如LinearLayout |   包含子View |

- Android中的UI组件都由View、ViewGroup组成。

###3. View类简介

- View类是Android中各种组件的基类，如View是ViewGroup基类
- View的构造函数：共有4个，具体如下：

> 自定义View必须重写至少一个构造函数：



```java
// 如果View是在Java代码里面new的，则调用第一个构造函数
 public CarsonView(Context context) {
        super(context);
    }

// 如果View是在.xml里声明的，则调用第二个构造函数
// 自定义属性是从AttributeSet参数传进来的
    public  CarsonView(Context context, AttributeSet attrs) {
        super(context, attrs);
    }

// 不会自动调用
// 一般是在第二个构造函数里主动调用
// 如View有style属性时
    public  CarsonView(Context context, AttributeSet attrs, int defStyleAttr) {
        super(context, attrs, defStyleAttr);
    }

    //API21之后才使用
    // 不会自动调用
    // 一般是在第二个构造函数里主动调用
    // 如View有style属性时
    public  CarsonView(Context context, AttributeSet attrs, int defStyleAttr, int defStyleRes) {
        super(context, attrs, defStyleAttr, defStyleRes);
    }
```

更加具体的使用请看：[深入理解View的构造函数](https://links.jianshu.com/go?to=http%3A%2F%2Fwww.jcodecraeer.com%2Fa%2Fanzhuokaifa%2Fandroidkaifa%2F2016%2F0806%2F4575.html)和
 [理解View的构造函数](https://links.jianshu.com/go?to=http%3A%2F%2Fwww.cnblogs.com%2Fangeldevil%2Fp%2F3479431.html%23three)

###4. View视图结构

- 对于多View的视图，**结构是树形结构**：最顶层是ViewGroup
- ViewGroup下可能有多个ViewGroup或View，如下图：

![img](https:////upload-images.jianshu.io/upload_images/944365-afb2be431e523baf.png)



一定要记住：无论是measure过程、layout过程还是draw过程，**永远都是从View树的根节点开始测量或计算（即从树的顶端开始），一层一层、一个分支一个分支地进行（即树形递归），**最终计算整个View树中各个View，最终确定整个View树的相关属性。

###5. Android坐标系

Android的坐标系定义为：

- 屏幕的左上角为坐标原点
- 向右为x轴增大方向
- 向下为y轴增大方向

具体如下图：



![img](https:////upload-images.jianshu.io/upload_images/944365-ee0cd39fd788e293.png)



**注：区别于一般的数学坐标系**

![img](https:////upload-images.jianshu.io/upload_images/944365-bddc2366bca78eff.png)



###6. View位置（坐标）描述

- View的位置由4个顶点决定的（如下A、B、C、D）

![img](https:////upload-images.jianshu.io/upload_images/944365-398c610a464cbdc8.png)

4个顶点的位置描述分别由4个值决定：
 （请记住：**View的位置是相对于父控件而言的**）

- Top：子View上边界到父view上边界的距离
- Left：子View左边界到父view左边界的距离
- Bottom：子View下边距到父View上边界的距离
- Right：子View右边界到父view左边界的距离

如下图：



![img](https:////upload-images.jianshu.io/upload_images/944365-2fb2682c45d05ff9.png)

View的位置描述

**个人建议**：按顶点位置来记忆：

- Top：子View左上角距父View顶部的距离；
- Left：子View左上角距父View左侧的距离；
- Bottom：子View右下角距父View顶部的距离
- Right：子View右下角距父View左侧的距离

###7. 位置获取方式

- View的位置是通过`view.getxxx()`函数进行获取：**（以Top为例）**

```java
// 获取Top位置
public final int getTop() {  
    return mTop;  
}  

// 其余如下：
  getLeft();      //获取子View左上角距父View左侧的距离
  getBottom();    //获取子View右下角距父View顶部的距离
  getRight();     //获取子View右下角距父View左侧的距离
```

- 与MotionEvent中 `get()`和`getRaw()`的区别

```csharp
//get() ：触摸点相对于其所在组件坐标系的坐标
 event.getX();       
 event.getY();

//getRaw() ：触摸点相对于屏幕默认坐标系的坐标
 event.getRawX();    
 event.getRawY();
```

具体如下图：

![img](https:////upload-images.jianshu.io/upload_images/944365-e50a2705cdd632d3.png)



###8. 角度（angle）& 弧度（radian）

- 自定义View实际上是将一些简单的形状通过计算，从而组合到一起形成的效果。

> 这会涉及到画布的相关操作(旋转)、正余弦函数计算等，即会涉及到角度(angle)与弧度(radian)的相关知识。

- 角度和弧度都是描述角的一种度量单位，区别如下图：：

![img](https:////upload-images.jianshu.io/upload_images/944365-7a81d3e1715eda0b.png)



**在默认的屏幕坐标系中角度增大方向为顺时针。**

![img](https:////upload-images.jianshu.io/upload_images/944365-de35d1bdfea46470.png)

**注：在常见的数学坐标系中角度增大方向为逆时针**

###9. 颜色相关

Android中的颜色相关内容包括颜色模式，创建颜色的方式，以及颜色的混合模式等。

####9.1 颜色模式

Android支持的颜色模式：

![img](https:////upload-images.jianshu.io/upload_images/944365-43d2051c332e0f95.png)



以ARGB8888为例介绍颜色定义:

![img](https:////upload-images.jianshu.io/upload_images/944365-f63d3055739f08b2.png)



####9.2 定义颜色的方式

 **1、在java中定义颜色**

```cpp
 //java中使用Color类定义颜色
 int color = Color.GRAY;     //灰色

  //Color类是使用ARGB值进行表示
  int color = Color.argb(127, 255, 0, 0);   //半透明红色
  int color = 0xaaff0000;                   //带有透明度的红色
```



**2、在xml文件中定义颜色**

在/res/values/color.xml 文件中如下定义：

```xml
<?xml version="1.0" encoding="utf-8"?>
<resources>

    //定义了红色（没有alpha（透明）通道）
    <color name="red">#ff0000</color>
    //定义了蓝色（没有alpha（透明）通道）
    <color name="green">#00ff00</color>
</resources>
```

在xml文件中以”#“开头定义颜色，后面跟十六进制的值，有如下几种定义方式：

```bash
  #f00            //低精度 - 不带透明通道红色
  #af00           //低精度 - 带透明通道红色

  #ff0000         //高精度 - 不带透明通道红色
  #aaff0000       //高精度 - 带透明通道红色
```

####9.3 引用颜色的方式

**1、在java文件中引用xml中定义的颜色：**

```cpp
//方法1
int color = getResources().getColor(R.color.mycolor);

//方法2（API 23及以上）
int color = getColor(R.color.myColor);    
```

**2、 在xml文件(layout或style)中引用或者创建颜色：**

```xml
 <!--在style文件中引用-->
    <style name="AppTheme" parent="Theme.AppCompat.Light.DarkActionBar">
        <item name="colorPrimary">@color/red</item>
    </style>

 <!--在layout文件中引用在/res/values/color.xml中定义的颜色-->
  android:background="@color/red"     

 <!--在layout文件中创建并使用颜色-->
  android:background="#ff0000"        
```

#### 9.4 取色工具

- 颜色都是用RGB值定义的，而我们一般是无法直观的知道自己需要颜色的值，需要借用取色工具直接从图片或者其他地方获取颜色的RGB值。
- 有时候一些简单的颜色选取就不用去麻烦UI了，**开发者自己去选取效率更高**
- 这里，取色工具我强推**Markman**：一款设计师用于标注的工具，主要用于尺寸标注、字体大小标注、颜色标注，而且使用简单。**本人强烈推荐！**

<img src="https:////upload-images.jianshu.io/upload_images/944365-c0fbba225d4042ac.png" alt="img" style="zoom:33%;" />

## 1.2 底层原理



# 二、measure

##1. 作用

测量`View`的宽 / 高

> 1. 在某些情况下，需要多次测量`（measure）`才能确定`View`最终的宽/高；
> 2. 该情况下，`measure`过程后得到的宽 / 高可能不准确；
> 3. 此处建议：在`layout`过程中`onLayout()`去获取最终的宽 / 高

##2. 储备知识

了解`measure`过程前，需要先了解传递尺寸（宽 / 高测量值）的2个类：

- `ViewGroup.LayoutParams`类（）
- `MeasureSpecs` 类（父视图对子视图的测量要求）

###2.1 ViewGroup.LayoutParams

- 简介
   布局参数类

> 1. `ViewGroup` 的子类`（RelativeLayout、LinearLayout）`有其对应的 `ViewGroup.LayoutParams` 子类
> 2. 如：`RelativeLayout`的 `ViewGroup.LayoutParams`子类
>     = `RelativeLayoutParams`

- 作用
   指定视图`View` 的高度`（height）`  和 宽度`（width）`等布局参数。

- 具体使用
   通过以下参数指定

  | 参数         |                             解释                             |
  | :----------- | :----------------------------------------------------------: |
  | 具体值       |                           dp / px                            |
  | fill_parent  |  强制性使子视图的大小扩展至与父视图大小相等（不含 padding )  |
  | match_parent |        与fill_parent相同，用于Android 2.3 & 之后版本         |
  | wrap_content | 自适应大小，强制性地使视图扩展以便显示其全部内容(含 padding ) |

```cpp
android:layout_height="wrap_content"   //自适应大小  
android:layout_height="match_parent"   //与父视图等高  
android:layout_height="fill_parent"    //与父视图等高  
android:layout_height="100dip"         //精确设置高度值为 100dip  
```

- 构造函数
   构造函数 = `View`的入口，可用于初始化 & 获取自定义属性

```java
// View的构造函数有四种重载
    public DIY_View(Context context){
        super(context);
    }

    public DIY_View(Context context,AttributeSet attrs){
        super(context, attrs);
    }

    public DIY_View(Context context,AttributeSet attrs,int defStyleAttr ){
        super(context, attrs,defStyleAttr);

// 第三个参数：默认Style
// 默认Style：指在当前Application或Activity所用的Theme中的默认Style
// 且只有在明确调用的时候才会生效，
    }
    
    public DIY_View(Context context,AttributeSet attrs,int defStyleAttr ，int defStyleRes){
        super(context, attrs，defStyleAttr，defStyleRes);
    }

// 最常用的是1和2
}
```

###2.2 MeasureSpec

####2.2.1 概览

![img](https:////upload-images.jianshu.io/upload_images/944365-0cf0a1ffd083cad1.png)



####2.2.2 格式定义

测量规格`（MeasureSpec）` = 测量模式`（mode）` + 测量大小`（size）`

![img](https:////upload-images.jianshu.io/upload_images/944365-7d0f873cee3912bb.png)

其中，测量模式`（Mode）`的类型有3种：UNSPECIFIED、EXACTLY 和
 AT_MOST。具体如下：

![img](https:////upload-images.jianshu.io/upload_images/944365-e631b96ea1906e34.png)



####2.2.3 使用与源码分析

- `MeasureSpec` 被封装在`View`类中的一个内部类里：`MeasureSpec`类

- MeasureSpec类 用1个变量封装了2个数据`（size，mode）`：**通过使用二进制**，将测量模式`（mode）` & 测量大小`(size）`打包成一个`int`值来，并提供了打包 & 解包的方法

  > 该措施的目的 = 减少对象内存分配

- 实际使用

  ```java
  /**
    * MeasureSpec类的具体使用
    **/
  
      // 1. 获取测量模式（Mode）
      int specMode = MeasureSpec.getMode(measureSpec)
  
      // 2. 获取测量大小（Size）
      int specSize = MeasureSpec.getSize(measureSpec)
  
      // 3. 通过Mode 和 Size 生成新的SpecMode
      int measureSpec=MeasureSpec.makeMeasureSpec(size, mode);
  ```

- 源码分析

  ```java
  /**
    * MeasureSpec类的源码分析
    **/
      public class MeasureSpec {
  
          // 进位大小 = 2的30次方
          // int的大小为32位，所以进位30位 = 使用int的32和31位做标志位
          private static final int MODE_SHIFT = 30;  
            
          // 运算遮罩：0x3为16进制，10进制为3，二进制为11
          // 3向左进位30 = 11 00000000000(11后跟30个0)  
          // 作用：用1标注需要的值，0标注不要的值。因1与任何数做与运算都得任何数、0与任何数做与运算都得0
          private static final int MODE_MASK  = 0x3 << MODE_SHIFT;  
    
          // UNSPECIFIED的模式设置：0向左进位30 = 00后跟30个0，即00 00000000000
          // 通过高2位
          public static final int UNSPECIFIED = 0 << MODE_SHIFT;  
          
          // EXACTLY的模式设置：1向左进位30 = 01后跟30个0 ，即01 00000000000
          public static final int EXACTLY = 1 << MODE_SHIFT;  
  
          // AT_MOST的模式设置：2向左进位30 = 10后跟30个0，即10 00000000000
          public static final int AT_MOST = 2 << MODE_SHIFT;  
    
          /**
            * makeMeasureSpec（）方法
            * 作用：根据提供的size和mode得到一个详细的测量结果吗，即measureSpec
            **/ 
              public static int makeMeasureSpec(int size, int mode) {  
              
                  return size + mode;  
              // measureSpec = size + mode；此为二进制的加法 而不是十进制
              // 设计目的：使用一个32位的二进制数，其中：32和31位代表测量模式（mode）、后30位代表测量大小（size）
              // 例如size=100(4)，mode=AT_MOST，则measureSpec=100+10000...00=10000..00100  
  
              }  
        
          /**
            * getMode（）方法
            * 作用：通过measureSpec获得测量模式（mode）
            **/    
  
              public static int getMode(int measureSpec) {  
                  return (measureSpec & MODE_MASK);  
                  // 即：测量模式（mode） = measureSpec & MODE_MASK;  
                  // MODE_MASK = 运算遮罩 = 11 00000000000(11后跟30个0)
                  //原理：保留measureSpec的高2位（即测量模式）、使用0替换后30位
                  // 例如10 00..00100 & 11 00..00(11后跟30个0) = 10 00..00(AT_MOST)，这样就得到了mode的值
  
              }  
          /**
            * getSize方法
            * 作用：通过measureSpec获得测量大小size
            **/       
              public static int getSize(int measureSpec) {  
                  return (measureSpec & ~MODE_MASK);  
                  // size = measureSpec & ~MODE_MASK;  
                 // 原理类似上面，即 将MODE_MASK取反，也就是变成了00 111111(00后跟30个1)，将32,31替换成0也就是去掉mode，保留后30位的size  
              } 
  
      }  
  ```

  

####2.2.4 MeasureSpec值的计算

- 上面讲了那么久`MeasureSpec`，那么`MeasureSpec`值到底是如何计算得来?

- 结论：

  1. 普通View：根据**子View的布局参数（LayoutParams）和父容器的MeasureSpec值**计算得来的，具体计算逻辑封装在`getChildMeasureSpec()`里。如下图：

     ![img](https:////upload-images.jianshu.io/upload_images/944365-d059b1afdeae0256.png)

     > 即：子`view`的大小由父`view`的`MeasureSpec`值 和 子`view`的`LayoutParams`属性 共同决定

  2. `DecorView`：取决于 **自身布局参数 & 自身窗口尺寸**

     ![img](https:////upload-images.jianshu.io/upload_images/944365-560312570515aef4.png)

     

- 下面，我们来看`getChildMeasureSpec()`的源码分析：

```dart
/**
  * 源码分析：ViewGroup.getChildMeasureSpec（）
  * 作用：根据父视图的MeasureSpec & 布局参数LayoutParams，计算单个子View的MeasureSpec
  * 注：子view的大小由父view的MeasureSpec值 和 子view的LayoutParams属性 共同决定
  **/
    public static int getChildMeasureSpec(int spec, int padding, int childDimension) {  
         //参数说明
         * @param spec 父view的详细测量值(MeasureSpec) 
         * @param padding view当前尺寸的的内边距和外边距(padding,margin) 
         * @param childDimension 子视图的布局参数（宽/高）

            //父view的测量模式
            int specMode = MeasureSpec.getMode(spec);     

            //父view的大小
            int specSize = MeasureSpec.getSize(spec);     
          
            //通过父view计算出的子view = 父大小-边距（父要求的大小，但子view不一定用这个值）   
            int size = Math.max(0, specSize - padding);  
          
            //子view想要的实际大小和模式（需要计算）  
            int resultSize = 0;  
            int resultMode = 0;  
          
            //通过父view的MeasureSpec和子view的LayoutParams确定子view的大小  


            // 当父view的模式为EXACITY时，父view强加给子view确切的值
           //一般是父view设置为match_parent或者固定值的ViewGroup 
            switch (specMode) {  
            case MeasureSpec.EXACTLY:  
                if (childDimension >= 0) {  
                    //子view大小为子自身所赋的值，模式大小为EXACTLY  
                    resultSize = childDimension;  
                    resultMode = MeasureSpec.EXACTLY;  

                // 当子view的LayoutParams为MATCH_PARENT时(-1)  
                } else if (childDimension == LayoutParams.MATCH_PARENT) {  
                    //子view大小为父view大小，模式为EXACTLY  
                    resultSize = size;  
                    resultMode = MeasureSpec.EXACTLY;  

                // 当子view的LayoutParams为WRAP_CONTENT时(-2)      
                } else if (childDimension == LayoutParams.WRAP_CONTENT) {  
                    //子view决定自己的大小，但最大不能超过父view，模式为AT_MOST  
                    resultSize = size;  
                    resultMode = MeasureSpec.AT_MOST;  
                }  
                break;  
          
            // 当父view的模式为AT_MOST时，父view强加给子view一个最大的值。（一般是父view设置为wrap_content）  
            case MeasureSpec.AT_MOST:  
                // 道理同上  
                if (childDimension >= 0) {  
                    resultSize = childDimension;  
                    resultMode = MeasureSpec.EXACTLY;  
                } else if (childDimension == LayoutParams.MATCH_PARENT) {  
                    resultSize = size;  
                    resultMode = MeasureSpec.AT_MOST;  
                } else if (childDimension == LayoutParams.WRAP_CONTENT) {  
                    resultSize = size;  
                    resultMode = MeasureSpec.AT_MOST;  
                }  
                break;  
          
            // 当父view的模式为UNSPECIFIED时，父容器不对view有任何限制，要多大给多大
            // 多见于ListView、GridView  
            case MeasureSpec.UNSPECIFIED:  
                if (childDimension >= 0) {  
                    // 子view大小为子自身所赋的值  
                    resultSize = childDimension;  
                    resultMode = MeasureSpec.EXACTLY;  
                } else if (childDimension == LayoutParams.MATCH_PARENT) {  
                    // 因为父view为UNSPECIFIED，所以MATCH_PARENT的话子类大小为0  
                    resultSize = 0;  
                    resultMode = MeasureSpec.UNSPECIFIED;  
                } else if (childDimension == LayoutParams.WRAP_CONTENT) {  
                    // 因为父view为UNSPECIFIED，所以WRAP_CONTENT的话子类大小为0  
                    resultSize = 0;  
                    resultMode = MeasureSpec.UNSPECIFIED;  
                }  
                break;  
            }  
            return MeasureSpec.makeMeasureSpec(resultSize, resultMode);  
        }  
```

- 关于`getChildMeasureSpec()`里对子`View`的测量模式 & 大小的判断逻辑有点复杂；
- 别担心，我已帮大家总结好。具体请看下表：

![img](https:////upload-images.jianshu.io/upload_images/944365-76261325e6576361.png)



其中的规律总结：（以子`View`为标准，横向观察）

![img](https:////upload-images.jianshu.io/upload_images/944365-6088d2d291bbae09.png)



> 由于`UNSPECIFIED`模式适用于系统内部多次`measure`情况，很少用到，故此处不讨论



##3. measure过程详解

- `measure`过程 根据**View的类型**分为2种情况：

![img](https:////upload-images.jianshu.io/upload_images/944365-556bf094df91b9de.png)

- 接下来，我将详细分析这两种`measure`过程

###3.1 单一View的measure过程

- 应用场景
   在无现成的控件`View`满足需求、需自己实现时，则使用自定义单一`View`

> 1. 如：制作一个支持加载网络图片的`ImageView`控件
> 2. 注：自定义`View`在多数情况下都有替代方案：图片 / 组合动画，但二者可能会导致内存耗费过大，从而引起内存溢出等问题。

- 具体使用
   继承自`View`、`SurfaceView` 或 其他`View`；不包含子`View`
- 具体流程

![img](https:////upload-images.jianshu.io/upload_images/944365-6fd614936d045071.png)

单一View的measure过程

下面我将一个个方法进行详细分析：入口 = `measure（）`

```dart
/**
  * 源码分析：View.measure（），final类型，即子类不能重写此方法，
  * 过程：父View调用子View的measure方法，而measure又会调用onMeasure，实际的测量工作是在onMeasure中完成的，只有onMeasure可以重写，也必须重写
  * 作用：计算View大小；
  **/ 
    public final void measure(int widthMeasureSpec, int heightMeasureSpec) {
        // 参数说明：View的宽 / 高 测量规格
        ...
        int cacheIndex = (mPrivateFlags & PFLAG_FORCE_LAYOUT) == PFLAG_FORCE_LAYOUT ? -1 :
                mMeasureCache.indexOfKey(key);

        if (cacheIndex < 0 || sIgnoreMeasureCache) {
            onMeasure(widthMeasureSpec, heightMeasureSpec);
            // 计算视图大小 ->>分析1
        } else {
            ...
    }

/**
  * 分析1：onMeasure（）
  * 作用：a. 根据View宽/高的测量规格计算View的宽/高值：getDefaultSize()
  *      b. 存储测量后的View宽 / 高：setMeasuredDimension()
  **/ 
  protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {  
    // 参数说明：View的宽 / 高测量规格

    setMeasuredDimension(getDefaultSize(getSuggestedMinimumWidth(), widthMeasureSpec),  
                         getDefaultSize(getSuggestedMinimumHeight(), heightMeasureSpec));  
    // setMeasuredDimension() ：获得View宽/高的测量值 ->>分析2
    // 传入的参数通过getDefaultSize()获得 ->>分析3
}

/**
  * 分析2：setMeasuredDimension()
  * 作用：存储测量后的View宽 / 高
  * 注：该方法即为我们重写onMeasure()所要实现的最终目的
  **/
    protected final void setMeasuredDimension(int measuredWidth, int measuredHeight) {  
    //参数说明：测量后子View的宽 / 高值

        // 将测量后子View的宽 / 高值进行传递
            mMeasuredWidth = measuredWidth;  
            mMeasuredHeight = measuredHeight;  
          
            mPrivateFlags |= PFLAG_MEASURED_DIMENSION_SET;  
        } 
    // 由于setMeasuredDimension（）的参数是从getDefaultSize()获得的
    // 下面我们继续看getDefaultSize()的介绍

/**
  * 分析3：getDefaultSize()
  * 作用：根据View宽/高的测量规格计算View的宽/高值
  **/
  public static int getDefaultSize(int size, int measureSpec) {  
        // 参数说明：
        // size：提供的默认大小
        // measureSpec：宽/高的测量规格（含模式 & 测量大小）

            // 设置默认大小
            int result = size; 
            
            // 获取宽/高测量规格的模式 & 测量大小
            int specMode = MeasureSpec.getMode(measureSpec);  
            int specSize = MeasureSpec.getSize(measureSpec);  
          
            switch (specMode) {  
                // 模式为UNSPECIFIED时，使用提供的默认大小 = 参数Size
                case MeasureSpec.UNSPECIFIED:  
                    result = size;  
                    break;  

                // 模式为AT_MOST,EXACTLY时，使用View测量后的宽/高值 = measureSpec中的Size
                case MeasureSpec.AT_MOST:  
                case MeasureSpec.EXACTLY:  
                    result = specSize;  
                    break;  
            }  

         // 返回View的宽/高值
            return result;  
        }    
```

- 上面提到，当模式是`UNSPECIFIED`时，使用的是提供的默认大小（即第一个参数size）；那么，提供的默认大小具体是多少呢？
- 答：在onMeasure（）方法中，getDefaultSize(getSuggestedMinimumWidth(), widthMeasureSpec)中传入的默认大小是getSuggestedMinimumWidth()。

接下来我们继续看`getSuggestedMinimumWidth()`的源码分析

> 由于`getSuggestedMinimumHeight()`类似，所以此处仅分析`getSuggestedMinimumWidth()`

- 源码分析如下：

```csharp
protected int getSuggestedMinimumWidth() {
    return (mBackground == null) ? mMinWidth : max(mMinWidth,mBackground.getMinimumWidth());
}

//getSuggestedMinimumHeight()同理
```

从代码可以看出：

- 若 `View` 无设置背景，那么`View`的宽度 = `mMinWidth`

> 1. ·mMinWidth· = `android:minWidth`属性所指定的值；
> 2. 若`android:minWidth`没指定，则默认为0

- 若 `View`设置了背景，`View`的宽度为`mMinWidth`和`mBackground.getMinimumWidth()`中的最大值

那么，`mBackground.getMinimumWidth()`的大小具体指多少？继续看`getMinimumWidth()`的源码分析：

```java
public int getMinimumWidth() {
    final int intrinsicWidth = getIntrinsicWidth();
    //返回背景图Drawable的原始宽度
    return intrinsicWidth > 0 ? intrinsicWidth :0 ;
}

// 由源码可知：mBackground.getMinimumWidth()的大小 = 背景图Drawable的原始宽度
// 若无原始宽度，则为0；
// 注：BitmapDrawable有原始宽度，而ShapeDrawable没有
```

总结：`getDefaultSize()`计算View的宽/高值的逻辑

![img](https:////upload-images.jianshu.io/upload_images/944365-bf6b3dc2261012dc.png)





**至此，单一View的宽/高值已经测量完成，即对于单一View的measure过程已经完成。**



##### 解惑

1. 根据这里的计算过程，一个100x100的LinearLayout，包含一个wrap_content的子View，子View的大小应该也是100x100，但是印象中TextView是文字的尺寸、ImageView是图片的尺寸啊 怎么回事？

   > 1. 他们都覆写了onMeasure方法，改变了默认测量值
   >
   > 2. 普通View确实会充满
   > 3. TextView、ImageView也确实是这样的表现

2. MeasureSpec的计算模式是固定的，但是具体View尺寸的计算，需要子View根据这个规格在onMeasure中自定义实现

   ```java
   // ViewGroup类 
   
   //静态方法 父布局的约束spec + 子布局自身的要求childDimension ===>> 最终子布局的MeasureSpec
   // childDimension 可以是具体值，也可以是LayoutParams.MATCH_PARENT等值
   public static int getChildMeasureSpec(int spec, int padding, int childDimension)
     
    // getChildMeasureSpec 使用举例（ ViewGroup类 ）
    protected void measureChildWithMargins(View child,
            int parentWidthMeasureSpec, int widthUsed,
            int parentHeightMeasureSpec, int heightUsed) {
        final MarginLayoutParams lp = (MarginLayoutParams) child.getLayoutParams();
   
        final int childWidthMeasureSpec = getChildMeasureSpec(parentWidthMeasureSpec,
                mPaddingLeft + mPaddingRight + lp.leftMargin + lp.rightMargin
                        + widthUsed, lp.width);
        final int childHeightMeasureSpec = getChildMeasureSpec(parentHeightMeasureSpec,
                mPaddingTop + mPaddingBottom + lp.topMargin + lp.bottomMargin
                        + heightUsed, lp.height);
   
        child.measure(childWidthMeasureSpec, childHeightMeasureSpec);
    }
   ```

   

3. Fs



####总结

对于单一View的measure过程，如下：

![img](https:////upload-images.jianshu.io/upload_images/944365-2ba9801fcad2b48c.png)

示意图

> 实际作用的方法：`getDefaultSize()` = 计算View的宽/高值、`setMeasuredDimension（）` = 存储测量后的View宽 / 高





###3.2 ViewGroup的measure过程

- 应用场景
   利用现有的组件根据特定的布局方式来组成新的组件
- 具体使用
   继承自`ViewGroup` 或 各种`Layout`；含有子 `View`

> 如：底部导航条中的条目，一般都是上图标(ImageView)、下文字(TextView)，那么这两个就可以用自定义ViewGroup组合成为一个Veiw，提供两个属性分别用来设置文字和图片，使用起来会更加方便。
>
> <img src="https:////upload-images.jianshu.io/upload_images/944365-3eb8d5d9d7d9e9fd.png" alt="img" style="zoom:33%;" />

- 原理

  1. 遍历 测量所有子`View`的尺寸
  2. 合并将所有子`View`的尺寸进行，最终得到`ViewGroup`父视图的测量值

  > 自上而下、一层层地传递下去，直到完成整个`View`树的`measure（）`过程

![img](https:////upload-images.jianshu.io/upload_images/944365-7133935cb1e56190.png)

示意图

- 流程

![img](https:////upload-images.jianshu.io/upload_images/944365-1438a7fbd93d0987.png)

下面我将一个个方法进行详细分析：入口 = `measure（）`

> 若需进行自定义`ViewGroup`，则需重写`onMeasure()`，下文会提到

```dart
/**
  * 源码分析：measure()
  * 作用：基本测量逻辑的判断；调用onMeasure()
  * 注：与单一View measure过程中讲的measure()一致
  **/ 
  public final void measure(int widthMeasureSpec, int heightMeasureSpec) {
    ...
    int cacheIndex = (mPrivateFlags & PFLAG_FORCE_LAYOUT) == PFLAG_FORCE_LAYOUT ? -1 :
            mMeasureCache.indexOfKey(key);
    if (cacheIndex < 0 || sIgnoreMeasureCache) {

        // 调用onMeasure()计算视图大小
        onMeasure(widthMeasureSpec, heightMeasureSpec);
        mPrivateFlags3 &= ~PFLAG3_MEASURE_NEEDED_BEFORE_LAYOUT;
    } else {
        ...
}

/**
  * 分析1：onMeasure()
  * 作用：遍历子View & 测量
  * 注：ViewGroup = 一个抽象类 = 无重写View的onMeasure（），需自身复写
  **/ 
```

**为什么`ViewGroup`的`measure`过程不像单一`View`的`measure`过程那样对`onMeasure（）`做统一的实现？**

- 答：因为不同的`ViewGroup`子类（`LinearLayout`、`RelativeLayout` / 自定义`ViewGroup`子类等）具备不同的布局特性，这导致他们子`View`的测量方法各有不同

> 而`onMeasure（）`的作用 = 测量View的宽/高值

**因此，ViewGroup无法对onMeasure（）作统一实现。这个也是单一View的measure过程与ViewGroup过程最大的不同。**

> 1. 即 单一`View measure`过程的`onMeasure（）`具有统一实现，而`ViewGroup`则没有
> 2. 注：其实，在单一`View measure`过程中，`getDefaultSize()`只是简单的测量了宽高值，在实际使用时有时需更精细的测量。所以有时候也需重写`onMeasure（）`

在自定义`ViewGroup`中，关键在于：**根据需求复写`onMeasure()`从而实现你的子View测量逻辑**。复写`onMeasure()`的套路如下：

```dart
/**
  * 根据自身的测量逻辑复写onMeasure（），分为3步
  * 1. 遍历所有子View & 测量：measureChildren（）
  * 2. 合并所有子View的尺寸大小,最终得到ViewGroup父视图的测量值（自身实现）
  * 3. 存储测量后View宽/高的值：调用setMeasuredDimension()  
  **/ 
  @Override
  protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {  

        // 定义存放测量后的View宽/高的变量
        int widthMeasure ;
        int heightMeasure ;

        // 1. 遍历所有子View & 测量(measureChildren（）)
        // ->> 分析1
        measureChildren(widthMeasureSpec, heightMeasureSpec)；

        // 2. 合并所有子View的尺寸大小，最终得到ViewGroup父视图的测量值
         void measureCarson{
             ... // 自身实现
         }

        // 3. 存储测量后View宽/高的值：调用setMeasuredDimension()  
        // 类似单一View的过程，此处不作过多描述
        setMeasuredDimension(widthMeasure,  heightMeasure);  
  }
  // 从上可看出：
  // 复写onMeasure（）有三步，其中2步直接调用系统方法
  // 需自身实现的功能实际仅为步骤2：合并所有子View的尺寸大小

/**
  * 分析1：measureChildren()
  * 作用：遍历子View & 调用measureChild()进行下一步测量
  **/ 
    protected void measureChildren(int widthMeasureSpec, int heightMeasureSpec) {
        // 参数说明：父视图的测量规格（MeasureSpec）

                final int size = mChildrenCount;
                final View[] children = mChildren;

                // 遍历所有子view
                for (int i = 0; i < size; ++i) {
                    final View child = children[i];
                     // 调用measureChild()进行下一步的测量 ->>分析1
                  	// 注意：这里没有计算GONE的View
                    if ((child.mViewFlags & VISIBILITY_MASK) != GONE) {
                        measureChild(child, widthMeasureSpec, heightMeasureSpec);
                    }
                }
            }

/**
  * 分析2：measureChild()
  * 作用：a. 计算单个子View的MeasureSpec
  *      b. 测量每个子View最后的宽 / 高：调用子View的measure()
  **/ 
  protected void measureChild(View child, int parentWidthMeasureSpec,
            int parentHeightMeasureSpec) {

        // 1. 获取子视图的布局参数
        final LayoutParams lp = child.getLayoutParams();

        // 2. 根据父视图的MeasureSpec & 布局参数LayoutParams，计算单个子View的MeasureSpec
        // getChildMeasureSpec() 请看上面第2节储备知识处
        final int childWidthMeasureSpec = getChildMeasureSpec(parentWidthMeasureSpec,// 获取 ChildView 的 widthMeasureSpec
                mPaddingLeft + mPaddingRight, lp.width);
        final int childHeightMeasureSpec = getChildMeasureSpec(parentHeightMeasureSpec,// 获取 ChildView 的 heightMeasureSpec
                mPaddingTop + mPaddingBottom, lp.height);

        // 3. 将计算好的子View的MeasureSpec值传入measure()，进行最后的测量
        // 下面的流程即类似单一View的过程，此处不作过多描述
        child.measure(childWidthMeasureSpec, childHeightMeasureSpec);
    }
    // 回到调用原处
```

至此，`ViewGroup`的`measure`过程分析完毕

------

- 总结
   `ViewGroup`的`measure`过程如下：

![img](https:////upload-images.jianshu.io/upload_images/944365-c9ea47e8b5e325bf.png)

示意图

- 为了让大家更好地理解`ViewGroup`的`measure`过程（特别是复写`onMeasure()`），下面，我将用`ViewGroup`的子类`LinearLayout`来分析下`ViewGroup`的`measure`过程

###3.3 ViewGroup的measure过程实例解析（LinearLayout）

此处直接进入`LinearLayout`复写的`onMeasure（）`代码分析：

> 详细分析请看代码注释

```dart
  protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
      // 根据不同的布局属性进行不同的计算
      // 此处只选垂直方向的测量过程，即measureVertical()->>分析1
      if (mOrientation == VERTICAL) {
          measureVertical(widthMeasureSpec, heightMeasureSpec);
      } else {
          measureHorizontal(widthMeasureSpec, heightMeasureSpec);
      }   
}

  /**
    * 分析1：measureVertical()
    * 作用：测量LinearLayout垂直方向的测量尺寸
    **/ 
  void measureVertical(int widthMeasureSpec, int heightMeasureSpec) {
      /**
       *  其余测量逻辑
       **/
          // 获取垂直方向上的子View个数
          final int count = getVirtualChildCount();

          // 遍历子View获取其高度，并记录下子View中最高的高度数值
          for (int i = 0; i < count; ++i) {
              final View child = getVirtualChildAt(i);

              // 子View不可见，直接跳过该View的measure过程，getChildrenSkipCount()返回值恒为0
              // 注：若view的可见属性设置为VIEW.INVISIBLE，还是会计算该view大小
              if (child.getVisibility() == View.GONE) {
                 i += getChildrenSkipCount(child, i);
                 continue;
              }

              // 记录子View是否有weight属性设置，用于后面判断是否需要二次measure
              totalWeight += lp.weight;

              if (heightMode == MeasureSpec.EXACTLY && lp.height == 0 && lp.weight > 0) {
                  // 如果LinearLayout的specMode为EXACTLY且子View设置了weight属性，在这里会跳过子View的measure过程
                  // 同时标记skippedMeasure属性为true，后面会根据该属性决定是否进行第二次measure
                // 若LinearLayout的子View设置了weight，会进行两次measure计算，比较耗时
                  // 这就是为什么LinearLayout的子View需要使用weight属性时候，最好替换成RelativeLayout布局
                  final int totalLength = mTotalLength;
                  mTotalLength = Math.max(totalLength, totalLength + lp.topMargin + lp.bottomMargin);
                  skippedMeasure = true;
              } else {
                  int oldHeight = Integer.MIN_VALUE;
     /**
       *  步骤1：遍历所有子View & 测量：measureChildren（）
       *  注：该方法内部，最终会调用measureChildren（），从而 遍历所有子View & 测量
       **/
            measureChildBeforeLayout(
                   child, i, widthMeasureSpec, 0, heightMeasureSpec,
                   totalWeight == 0 ? mTotalLength : 0);
                   ...
            }

      /**
       *  步骤2：合并所有子View的尺寸大小,最终得到ViewGroup父视图的测量值（自身实现）
       **/        
              final int childHeight = child.getMeasuredHeight();

              // 1. mTotalLength用于存储LinearLayout在竖直方向的高度
              final int totalLength = mTotalLength;

              // 2. 每测量一个子View的高度， mTotalLength就会增加
              mTotalLength = Math.max(totalLength, totalLength + childHeight + lp.topMargin +
                     lp.bottomMargin + getNextLocationOffset(child));
      
              // 3. 记录LinearLayout占用的总高度
              // 即除了子View的高度，还有本身的padding属性值
              mTotalLength += mPaddingTop + mPaddingBottom;
              int heightSize = mTotalLength;

      /**
       *  步骤3：存储测量后View宽/高的值：调用setMeasuredDimension()  
       **/ 
       setMeasureDimension(resolveSizeAndState(maxWidth,width))

    ...

  }
```

至此，自定义`View`的中最重要、最复杂的`measure`过程讲解完毕。

##4. 总结

- 本文对自定义View中最重要、最复杂的`measure`过程进行了详细分析，具体如下图：

![img](https:////upload-images.jianshu.io/upload_images/944365-29a36501ce27a5cb.png)



![img](https:////upload-images.jianshu.io/upload_images/944365-1250b5f61c90147f.png)

# 三、layout

##1. 作用

计算视图`（View）`的位置

> 即计算`View`的四个顶点位置：`Left`、`Top`、`Right` 和 `Bottom`

##2. 知识储备

具体请看文章：[（1）自定义View基础 - 最易懂的自定义View原理系列](https://www.jianshu.com/p/146e5cec4863)

##3. layout过程详解

类似`measure`过程，`layout`过程根据**View的类型**分为2种情况：

![img](https:////upload-images.jianshu.io/upload_images/944365-6e978f448667eb52.png)



接下来，我将详细分析这2种情况下的`layout`过程

### 3.1 单一View的layout过程

- 应用场景
   在无现成的控件`View`满足需求、需自己实现时，则使用自定义单一`View`

> 1. 如：制作一个支持加载网络图片的`ImageView`控件
> 2. 注：自定义`View`在多数情况下都有替代方案：图片 / 组合动画，但二者可能会导致内存耗费过大，从而引起内存溢出等问题。

- 具体使用
   继承自`View`、`SurfaceView` 或 其他`View`；不包含子`View`
- 具体流程

![img](https:////upload-images.jianshu.io/upload_images/944365-05b688ab79b57ecf.png)

下面我将一个个方法进行详细分析

- 源码分析
   `layout`过程的入口 = `layout（）`，具体如下：

```java
/**
  * 源码分析：layout（）
  * 作用：确定View本身的位置，即设置View本身的四个顶点位置
  * 参数：相对于父布局的位置
  */ 
  public void layout(int l, int t, int r, int b) {  
    // 当前视图的四个顶点
    int oldL = mLeft;  
    int oldT = mTop;  
    int oldB = mBottom;  
    int oldR = mRight;  
      
    // 1. 确定View的位置：setFrame（） / setOpticalFrame（）
    // 即初始化四个顶点的值、判断当前View大小和位置是否发生了变化 & 返回 
    // ->>分析1、分析2
    boolean changed = isLayoutModeOptical(mParent) ?
            setOpticalFrame(l, t, r, b) : setFrame(l, t, r, b);

    // 2. 若视图的大小 & 位置发生变化
    // 会重新确定该View所有的子View在父容器的位置：onLayout（）
    if (changed || (mPrivateFlags & PFLAG_LAYOUT_REQUIRED) == PFLAG_LAYOUT_REQUIRED) {  
        onLayout(changed, l, t, r, b);  
        // 对于单一View的laytou过程：由于单一View是没有子View的，故onLayout（）是一个空实现->>分析3
        // 对于ViewGroup的laytou过程：由于确定位置与具体布局有关，所以onLayout（）在ViewGroup为1个抽象方法，需重写实现（后面会详细说）
  ...

}  

/**
  * 分析1：setFrame（）
  * 作用：根据传入的4个位置值，设置View本身的四个顶点位置
  * 即：最终确定View本身的位置
  */ 
  protected boolean setFrame(int left, int top, int right, int bottom) {
        ...
    // 通过以下赋值语句记录下了视图的位置信息，即确定View的四个顶点
    // 从而确定了视图的位置
    mLeft = left;
    mTop = top;
    mRight = right;
    mBottom = bottom;

    mRenderNode.setLeftTopRightBottom(mLeft, mTop, mRight, mBottom);
    }

/**
  * 分析2：setOpticalFrame（）
  * 作用：根据传入的4个位置值，设置View本身的四个顶点位置
  * 即：最终确定View本身的位置
  */ 
  private boolean setOpticalFrame(int left, int top, int right, int bottom) {
        Insets parentInsets = mParent instanceof View ?
                ((View) mParent).getOpticalInsets() : Insets.NONE;

        Insets childInsets = getOpticalInsets();

        // 内部实际上是调用setFrame（）
        return setFrame(
                left   + parentInsets.left - childInsets.left,
                top    + parentInsets.top  - childInsets.top,
                right  + parentInsets.left + childInsets.right,
                bottom + parentInsets.top  + childInsets.bottom);
    }
    // 回到调用原处

/**
  * 分析3：onLayout（）
  * 注：对于单一View的laytou过程
  *    a. 由于单一View是没有子View的，故onLayout（）是一个空实现
  *    b. 由于在layout（）中已经对自身View进行了位置计算，所以单一View的layout过程在layout（）后就已完成了
  */ 
 protected void onLayout(boolean changed, int left, int top, int right, int bottom) {
   // 参数说明
   // changed 当前View的大小和位置改变了 
   // left 左部位置
   // top 顶部位置
   // right 右部位置
   // bottom 底部位置
}  
```

至此，单一`View`的`layout`过程已分析完毕。

- 总结

  单一View的layout过程解析如下：

  ![img](https:////upload-images.jianshu.io/upload_images/944365-756f72f8ccc58d2c.png)

  



### 3.2 ViewGroup的layout过程

- 应用场景
   利用现有的组件根据特定的布局方式来组成新的组件
- 具体使用
   继承自`ViewGroup` 或 各种`Layout`；含有子 `View`

> 如：底部导航条中的条目，一般都是上图标(ImageView)、下文字(TextView)，那么这两个就可以用自定义ViewGroup组合成为一个Veiw，提供两个属性分别用来设置文字和图片，使用起来会更加方便。
>
> <img src="https:////upload-images.jianshu.io/upload_images/944365-3eb8d5d9d7d9e9fd.png" alt="img" style="zoom:33%;" />

- 原理（步骤）
  1. 计算自身`ViewGroup`的位置：`layout（）`
  2. 遍历子`View` & 确定自身子View在`ViewGroup`的位置（调用子`View` 的 `layout（）`）：`onLayout（）`

> a. 步骤2 类似于 单一`View`的`layout`过程
>  b. 自上而下、一层层地传递下去，直到完成整个`View`树的`layout（）`过程

![img](https:////upload-images.jianshu.io/upload_images/944365-7133935cb1e56190.png)



- 流程

![img](https:////upload-images.jianshu.io/upload_images/944365-7ebd03609c758d47.png)

此处需注意：
 `ViewGroup` 和 `View` 同样拥有`layout（）`和`onLayout()`，但二者不同的：

- 一开始计算`ViewGroup`位置时，调用的是`ViewGroup`的`layout（）`和`onLayout()`；
- 当开始遍历子`View` & 计算子`View`位置时，调用的是子`View`的`layout（）`和`onLayout()`

> 类似于单一`View`的`layout`过程

- 下面我将一个个方法进行详细分析：`layout`过程入口为`layout（）`

```dart
/**
  * 源码分析：layout（）
  * 作用：确定View本身的位置，即设置View本身的四个顶点位置
  * 注：与单一View的layout（）源码一致，此处省略
  */ 
  public void layout(int l, int t, int r, int b) {  
   ...
  }

/**
  * 分析3：onLayout（）
  * 作用：计算该ViewGroup包含所有的子View在父容器的位置（）
  * 注： 
  *      a. 定义为抽象方法，需重写，因：子View的确定位置与具体布局有关，所以onLayout（）在ViewGroup没有实现
  *      b. 在自定义ViewGroup时必须复写onLayout（）！！！！！
  *      c. 复写原理：遍历子View 、计算当前子View的四个位置值 & 确定自身子View的位置（调用子View layout（））
  */ 
  protected void onLayout(boolean changed, int left, int top, int right, int bottom) {
     // 参数说明
     // changed 当前View的大小和位置改变了 
     // left 左部位置
     // top 顶部位置
     // right 右部位置
     // bottom 底部位置

     // 1. 遍历子View：循环所有子View
          for (int i=0; i<getChildCount(); i++) {
              View child = getChildAt(i);   

              // 2. 计算当前子View的四个位置值
                // 2.1 位置的计算逻辑
                ...// 需自己实现，也是自定义View的关键

                // 2.2 对计算后的位置值进行赋值
                int mLeft  = Left
                int mTop  = Top
                int mRight = Right
                int mBottom = Bottom

              // 3. 根据上述4个位置的计算值，设置子View的4个顶点：调用子view的layout() & 传递计算过的参数
              // 即确定了子View在父容器的位置
              child.layout(mLeft, mTop, mRight, mBottom);
              // 该过程类似于单一View的layout过程中的layout（）和onLayout（），此处不作过多描述
          }
      }
  }
```

### 3.3 总结

对于ViewGroup的layout过程，如下：

![img](https:////upload-images.jianshu.io/upload_images/944365-6e27d40b50081d60.png)




 此处需注意：
`ViewGroup` 和 `View` 同样拥有`layout（）`和`onLayout()`，但二者不同的：

- 一开始计算`ViewGroup`位置时，调用的是`ViewGroup`的`layout（）`和`onLayout()`；
- 当开始遍历子`View` & 计算子`View`位置时，调用的是子`View`的`layout（）`和`onLayout()`

> 类似于单一`View`的`layout`过程

至此，`ViewGroup`的 `layout`过程已讲解完毕。

##4. 实例讲解

- 为了更好理解`ViewGroup`的`layout`过程（特别是复写`onLayout（）`）
- 下面，我将用2个实例来加深对ViewGroup layout过程的理解
  1. 系统提供的`ViewGroup`的子类：`LinearLayout`
  2. 自定义`View`（继承了`ViewGroup`类）

### 4.1 实例解析1（LinearLayout）

#### 4.1.1 原理

1. 计算出`LinearLayout`本身在父布局的位置
2. 计算出`LinearLayout`中所有子`View`在容器中的位置

#### 4.1.2 具体流程

![img](https:////upload-images.jianshu.io/upload_images/944365-7537ab3496115ac7.png)



#### 4.1.2 源码分析

- 在上述流程中，对于LinearLayout的`layout（）`的实现与上面所说是一样的，此处不作过多阐述
- 故直接进入`LinearLayout`复写的`onLayout（）`分析

```java
/**
  * 源码分析：LinearLayout复写的onLayout（）
  * 注：复写的逻辑 和 LinearLayout measure过程的 onMeasure()类似
  */ 
  @Override
  protected void onLayout(boolean changed, int l, int t, int r, int b) {
      // 根据自身方向属性，而选择不同的处理方式
      if (mOrientation == VERTICAL) {
          layoutVertical(l, t, r, b);
      } else {
          layoutHorizontal(l, t, r, b);
      }
  }
      // 由于垂直 / 水平方向类似，所以此处仅分析垂直方向（Vertical）的处理过程 ->>分析1

/**
  * 分析1：layoutVertical(l, t, r, b)
  */
    void layoutVertical(int left, int top, int right, int bottom) {
        // 子View的数量
        final int count = getVirtualChildCount();

        // 1. 遍历子View
        for (int i = 0; i < count; i++) {
            final View child = getVirtualChildAt(i);
            if (child == null) {
                childTop += measureNullChild(i);
            } else if (child.getVisibility() != GONE) {
                // 2. 计算子View的测量宽 / 高值
                final int childWidth = child.getMeasuredWidth();
                final int childHeight = child.getMeasuredHeight();

                // 3. 确定自身子View的位置
                // 即：递归调用子View的setChildFrame()，实际上是调用了子View的layout() ->>分析2
                setChildFrame(child, childLeft, childTop + getLocationOffset(child),
                        childWidth, childHeight);

                // childTop逐渐增大，即后面的子元素会被放置在靠下的位置
                // 这符合垂直方向的LinearLayout的特性
                childTop += childHeight + lp.bottomMargin + getNextLocationOffset(child);

                i += getChildrenSkipCount(child, i);
            }
        }
    }

/**
  * 分析2：setChildFrame()
  */
    private void setChildFrame( View child, int left, int top, int width, int height){       
        // setChildFrame（）仅仅只是调用了子View的layout（）而已
        child.layout(left, top, left ++ width, top + height);
        }
    // 在子View的layout（）又通过调用setFrame（）确定View的四个顶点
    // 即确定了子View的位置
    // 如此不断循环确定所有子View的位置，最终确定ViewGroup的位置
```

------

### 4.2  实例解析2：自定义View

- 上面讲的例子是系统提供的、已经封装好的`ViewGroup`子类：`LinearLayout`
- 但是，一般来说我们使用的都是自定义View；
- 接下来，我用一个简单的例子讲下自定义`View`的`layout（）`过程

### 4.2.1 实例视图说明

实例视图 = 1个`ViewGroup`（灰色视图），包含1个黄色的子`View`，如下图：

<img src="https:////upload-images.jianshu.io/upload_images/944365-b5d0de9e0342ea19.png" alt="img" style="zoom:33%;" />



### 4.2.2 原理

1. 计算出`ViewGroup`在父布局的位置
2. 计算出`ViewGroup`中子`View`在容器中的位置

![img](https:////upload-images.jianshu.io/upload_images/944365-0385d1abe692086d.png)



### 4.2.3 具体计算逻辑

- 具体计算逻辑是指计算子View的位置，即计算四顶点位置 = 计算Left、Top、Right和Bottom；
- 主要是写在**复写的onLayout（）**
- 计算公式如下：

![img](https:////upload-images.jianshu.io/upload_images/944365-cf72b2ef1fac691a.png)

示意图



```cpp
r = Left + width + Left；// 因左右间距一样
b = Top + height + Top；// 因上下间距一样

Left = (r - width) / 2；
Top = (b - height) / 2；

Right = width + Left;
Bottom = height + Top;
```

### 4.2.3 代码分析

因为其余方法同上，这里不作过多描述，所以这里只分析复写的`onLayout（）`

```java
/**
  * 源码分析：LinearLayout复写的onLayout（）
  * 注：复写的逻辑 和 LinearLayout measure过程的 onMeasure()类似
  */ 
  @Override  
protected void onLayout(boolean changed, int l, int t, int r, int b) {  
     // 参数说明
     // changed 当前View的大小和位置改变了 
     // left 左部位置
     // top 顶部位置
     // right 右部位置
     // bottom 底部位置

        // 1. 遍历子View：循环所有子View
        // 注：本例中其实只有一个
        for (int i=0; i<getChildCount(); i++) {
            View child = getChildAt(i);

            // 取出当前子View宽 / 高
            int width = child.getMeasuredWidth();
            int height = child.getMeasuredHeight();

            // 2. 计算当前子View的四个位置值
                // 2.1 位置的计算逻辑
                int mLeft = (r - width) / 2;
                int mTop = (b - height) / 2;
                int mRight =  mLeft + width；
                int mBottom = mTop + height；

            // 3. 根据上述4个位置的计算值，设置子View的4个顶点
            // 即确定了子View在父容器的位置
            child.layout(mLeft, mTop, mRight,mBottom);
        }
    }
}
```

布局文件如下：

```xml
<?xml version="1.0" encoding="utf-8"?>
<scut.carson_ho.layout_demo.Demo_ViewGroup xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:background="#eee998"
    tools:context="scut.carson_ho.layout_demo.MainActivity">

    <Button
        android:text="ChildView"
        android:layout_width="200dip"
        android:layout_height="200dip"
        android:background="#333444"
        android:id="@+id/ChildView" />
</scut.carson_ho.layout_demo.Demo_ViewGroup >
```

- 效果图

<img src="https:////upload-images.jianshu.io/upload_images/944365-929a6b286bbf49a9.png" alt="img" style="zoom:33%;" />

示意图

好了，你是不是发现，粘了我的代码但是画不出来？！（如下图）

<img src="https:////upload-images.jianshu.io/upload_images/944365-0a4760b05951de1f.png" alt="img" style="zoom:33%;" />

实际示意图

**因为我还没说draw流程啊哈哈哈！**

> draw流程：将`View`最终绘制出来

`layout（）`过程讲到这里讲完了，接下来我将继续将自定义`View`的最后一个流程`draw`流程



##5.getWidth/getMeasuredWidth

getWidth() （ getHeight()）与 getMeasuredWidth() （getMeasuredHeight()）获取的宽 （高）有什么区别？

首先明确定义：

- `getWidth()`  / `getHeight()`：获得`View`最终的宽 / 高
- `getMeasuredWidth()`  / `getMeasuredHeight()`：获得 `View`测量的宽 / 高

先看下各自的源码：

```java
// 获得View测量的宽 / 高
  public final int getMeasuredWidth() {  
      return mMeasuredWidth & MEASURED_SIZE_MASK;  
      // measure过程中返回的mMeasuredWidth
  }  

  public final int getMeasuredHeight() {  
      return mMeasuredHeight & MEASURED_SIZE_MASK;  
      // measure过程中返回的mMeasuredHeight
  }  


// 获得View最终的宽 / 高
  public final int getWidth() {  
      return mRight - mLeft;  
      // View最终的宽 = 子View的右边界 - 子view的左边界。
  }  

  public final int getHeight() {  
      return mBottom - mTop;  
     // View最终的高 = 子View的下边界 - 子view的上边界。
  }  
```

二者的区别：

![img](https:////upload-images.jianshu.io/upload_images/944365-6b27b9835d927e04.png)

**上面标红：一般情况下，二者获取的宽 / 高是相等的。**那么，“非一般”情况是什么？

答：人为设置：通过重写`View`的 `layout（）`强行设置

```java
@Override
public void layout( int l , int t, int r , int b){  
   // 改变传入的顶点位置参数
   super.layout(l，t，r+100，b+100)；

   // 如此一来，在任何情况下，getWidth() / getHeight()获得的宽/高 总比 getMeasuredWidth() / getMeasuredHeight()获取的宽/高大100px
   // 即：View的最终宽/高 总比 测量宽/高 大100px
}
```

虽然这样的人为设置无实际意义，但证明了`View`的最终宽 / 高 与 测量宽 / 高是可以不一样

### 特别注意

网上流传这么一个原因描述：

> - 实际上在当屏幕可包裹内容时，他们的值是相等的；
> - 只有当view超出屏幕后，才能看出他们的区别：getMeasuredWidth()是实际View的大小，与屏幕无关，而getHeight的大小此时则是屏幕的大小。当超出屏幕后getMeasuredWidth()等于getWidth()加上屏幕之外没有显示的大小

这个结论是错的！详细请[点击文章](https://links.jianshu.com/go?to=http%3A%2F%2Fblog.csdn.net%2Fdmk877%2Farticle%2Fdetails%2F49734869%2F)

**结论：**

在非人为设置的情况下，`View`的最终宽/高（`getWidth()` / `getHeight()`）
 与 `View`的测量宽/高 （`getMeasuredWidth()`  /  `getMeasuredHeight()`）永远是相等

##6. 总结

- 本文主要讲解了自定义`View`中的`Layout`过程，总结如下：

![img](https:////upload-images.jianshu.io/upload_images/944365-bb11305f1e40a8fb.png)



![img](https:////upload-images.jianshu.io/upload_images/944365-6baebb31c56040dc.png)



# 四、draw

##1. 作用

绘制`View`视图

##2. 储备知识

具体请看文章：[自定义View基础 - 最易懂的自定义View原理系列](https://www.jianshu.com/p/146e5cec4863)

##3. draw过程详解

类似`measure`过程、`layout`过程，`draw`过程根据**View的类型**分为2种情况：

![img](https:////upload-images.jianshu.io/upload_images/944365-2dc3a798a3039bb5.png)



接下来，我将详细分析这2种情况下的`draw`过程

### 3.1 单一View的draw过程

- 应用场景
   在无现成的控件`View`满足需求、需自己实现时，则使用自定义单一`View`

> 1. 如：制作一个支持加载网络图片的`ImageView`控件
> 2. 注：自定义`View`在多数情况下都有替代方案：图片 / 组合动画，但二者可能会导致内存耗费过大，从而引起内存溢出等问题。

- 具体使用
   继承自`View`、`SurfaceView` 或 其他`View`；不包含子`View`
- 原理（步骤）
  1. `View`绘制自身（含背景、内容）；
  2. 绘制装饰（滚动指示器、滚动条、和前景）
- 具体流程

![img](https:////upload-images.jianshu.io/upload_images/944365-1554a862ed0c3f95.png)



下面我将一个个方法进行详细分析：`draw`过程的入口 = `draw（）`

```dart
/**
  * 源码分析：draw（）
  * 作用：根据给定的 Canvas 自动渲染 View（包括其所有子 View）。
  * 绘制过程：
  *   1. 绘制view背景
  *   2. 绘制view内容
  *   3. 绘制子View
  *   4. 绘制装饰（渐变框，滑动条等等）
  * 注：
  *    a. 在调用该方法之前必须要完成 layout 过程
  *    b. 所有的视图最终都是调用 View 的 draw （）绘制视图（ ViewGroup 没有复写此方法）
  *    c. 在自定义View时，不应该复写该方法，而是复写 onDraw(Canvas) 方法进行绘制
  *    d. 若自定义的视图确实要复写该方法，那么需先调用 super.draw(canvas)完成系统的绘制，然后再进行自定义的绘制
  */ 
  public void draw(Canvas canvas) {
    ...// 仅贴出关键代码
    int saveCount;

    // 步骤1： 绘制本身View背景
        if (!dirtyOpaque) {
            drawBackground(canvas);
        }

    // 若有必要，则保存图层（还有一个复原图层）
    // 优化技巧：当不需绘制 Layer 时，“保存图层“和“复原图层“这两步会跳过
    // 因此在绘制时，节省 layer 可以提高绘制效率
    final int viewFlags = mViewFlags;
    if (!verticalEdges && !horizontalEdges) {

    // 步骤2：绘制本身View内容
        if (!dirtyOpaque) 
            onDraw(canvas);
        // View 中：默认为空实现，需复写
        // ViewGroup中：需复写

    // 步骤3：绘制子View
    // 由于单一View无子View，故View 中：默认为空实现
    // ViewGroup中：系统已经复写好对其子视图进行绘制我们不需要复写
        dispatchDraw(canvas);
        
    // 步骤4：绘制装饰，如滑动条、前景色等等
        onDrawForeground(canvas);

        return;
    }
    ...    
}
```

下面，我们继续分析在`draw（）`中4个步骤调用的`drawBackground（）`、  `onDraw()`、`dispatchDraw()`、`onDrawScrollBars(canvas)`

```dart
/**
  * 步骤1：drawBackground(canvas)
  * 作用：绘制View本身的背景
  */
  private void drawBackground(Canvas canvas) {
        // 获取背景 drawable
        final Drawable background = mBackground;
        if (background == null) {
            return;
        }
        // 根据在 layout 过程中获取的 View 的位置参数，来设置背景的边界
        setBackgroundBounds();

        .....

        // 获取 mScrollX 和 mScrollY值 
        final int scrollX = mScrollX;
        final int scrollY = mScrollY;
        if ((scrollX | scrollY) == 0) {
            background.draw(canvas);
        } else {
            // 若 mScrollX 和 mScrollY 有值，则对 canvas 的坐标进行偏移
            canvas.translate(scrollX, scrollY);

            // 调用 Drawable 的 draw 方法绘制背景
            background.draw(canvas);
            canvas.translate(-scrollX, -scrollY);
        }
   } 

/**
  * 步骤2：onDraw(canvas)
  * 作用：绘制View本身的内容
  * 注：
  *   a. 由于 View 的内容各不相同，所以该方法是一个空实现
  *   b. 在自定义绘制过程中，需由子类去实现复写该方法，从而绘制自身的内容
  *   c. 谨记：自定义View中 必须 且 只需复写onDraw（）
  */
  protected void onDraw(Canvas canvas) {
        ... // 复写从而实现绘制逻辑
  }

/**
  * 步骤3： dispatchDraw(canvas)
  * 作用：绘制子View
  * 注：由于单一View中无子View，故为空实现
  */
  protected void dispatchDraw(Canvas canvas) {
        ... // 空实现
  }

/**
  * 步骤4： onDrawScrollBars(canvas)
  * 作用：绘制装饰，如 滚动指示器、滚动条、和前景等
  */
  public void onDrawForeground(Canvas canvas) {
        onDrawScrollIndicators(canvas);
        onDrawScrollBars(canvas);

        final Drawable foreground = mForegroundInfo != null ? mForegroundInfo.mDrawable : null;
        if (foreground != null) {
            if (mForegroundInfo.mBoundsChanged) {
                mForegroundInfo.mBoundsChanged = false;
                final Rect selfBounds = mForegroundInfo.mSelfBounds;
                final Rect overlayBounds = mForegroundInfo.mOverlayBounds;

                if (mForegroundInfo.mInsidePadding) {
                    selfBounds.set(0, 0, getWidth(), getHeight());
                } else {
                    selfBounds.set(getPaddingLeft(), getPaddingTop(),
                            getWidth() - getPaddingRight(), getHeight() - getPaddingBottom());
                }

                final int ld = getLayoutDirection();
                Gravity.apply(mForegroundInfo.mGravity, foreground.getIntrinsicWidth(),
                        foreground.getIntrinsicHeight(), selfBounds, overlayBounds, ld);
                foreground.setBounds(overlayBounds);
            }

            foreground.draw(canvas);
        }
    }
```

至此，单一`View`的`draw`过程已分析完毕。

### 总结

单一View的`draw`过程解析如下：

> 即 只需绘制`View`自身

![img](https:////upload-images.jianshu.io/upload_images/944365-b63b64086782a217.png)



### 3.2 ViewGroup的draw过程

- 应用场景
   利用现有的组件根据特定的布局方式来组成新的组件
- 具体使用
   继承自`ViewGroup` 或 各种`Layout`；含有子 `View`

> 如：底部导航条中的条目，一般都是上图标(ImageView)、下文字(TextView)，那么这两个就可以用自定义ViewGroup组合成为一个Veiw，提供两个属性分别用来设置文字和图片，使用起来会更加方便。
>
> <img src="https:////upload-images.jianshu.io/upload_images/944365-3eb8d5d9d7d9e9fd.png" alt="img" style="zoom:33%;" />

- 原理（步骤）

  1. `ViewGroup`绘制自身（含背景、内容）；

  2. `ViewGroup`遍历子`View` & 绘制其所有子View；

     > 类似于单一`View`的`draw`过程

  3. ViewGroup`绘制装饰（滚动指示器、滚动条、和前景）

  

> 自上而下、一层层地传递下去，直到完成整个`View`树的`draw`过程

![img](https:////upload-images.jianshu.io/upload_images/944365-7133935cb1e56190.png)



- 具体流程

![img](https:////upload-images.jianshu.io/upload_images/944365-ff799e17e24e4a3b.png)



ViewGroup没有覆写View的draw方法，因为基本过程都是相同的：背景>自身>分发>装饰等。

最大的不同，就是分发：`dispatchDraw()`

- 下面直接进入与单一`View` `draw`过程最大不同的步骤4：`dispatchDraw()`

```dart
/**
  * 源码分析：dispatchDraw（）
  * 作用：遍历子View & 绘制子View
  * 注：
  *   a. ViewGroup中：由于系统为我们实现了该方法，故不需重写该方法
  *   b. View中默认为空实现（因为没有子View可以去绘制）
  */ 
    protected void dispatchDraw(Canvas canvas) {
        ......

         // 1. 遍历子View
        final int childrenCount = mChildrenCount;
        ......

        for (int i = 0; i < childrenCount; i++) {
                ......
                if ((transientChild.mViewFlags & VISIBILITY_MASK) == VISIBLE ||
                        transientChild.getAnimation() != null) {
                  // 2. 绘制子View视图 ->>分析1
                    more |= drawChild(canvas, transientChild, drawingTime);
                }
                ....
        }
    }

/**
  * 分析1：drawChild（）
  * 作用：绘制子View
  */
    protected boolean drawChild(Canvas canvas, View child, long drawingTime) {
        // 最终还是调用了子 View 的 draw （）进行子View的绘制
        return child.draw(canvas, this, drawingTime);
    }
```

至此，`ViewGroup`的`draw`过程已分析完毕。

### 总结

`ViewGroup`的`draw`过程如下：

![img](https:////upload-images.jianshu.io/upload_images/944365-cf42edcab0a206fa.png)



##4. View.setWillNotDraw()

```dart
/**
  * 源码分析：setWillNotDraw()
  * 定义：View 中的特殊方法
  * 作用：设置 WILL_NOT_DRAW 标记位；
  * 注：
  *   a. 该标记位的作用是：当一个View不需要绘制内容时，系统进行相应优化
  *   b. 默认情况下：View 不启用该标记位（设置为false）；ViewGroup 默认启用（设置为true）
  */ 

public void setWillNotDraw(boolean willNotDraw) {
    setFlags(willNotDraw ? WILL_NOT_DRAW : 0, DRAW_MASK);
}

// 应用场景
// a. setWillNotDraw参数设置为true：当自定义View继承自 ViewGroup 、且本身并不具备任何绘制时，设置为 true 后，系统会进行相应的优化。
// b. setWillNotDraw参数设置为false：当自定义View继承自 ViewGroup 、且需要绘制内容时，那么设置为 false，来关闭 WILL_NOT_DRAW 这个标记位。
```



##5. 总结

- 本文全面总结了自定义`View`的`Draw`过程，总结如下

![img](https:////upload-images.jianshu.io/upload_images/944365-2c28edbbb3f1936f.png)

示意图

![img](https:////upload-images.jianshu.io/upload_images/944365-c9d3cd1d746be319.png)

