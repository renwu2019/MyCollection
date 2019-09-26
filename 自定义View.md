# 自定义View

[自定义View全解(简书来源)](https://www.jianshu.com/p/705a6cb6bfee)

## 一、基础

1. **分类**

2. **绘制流程：measure()、layout()、draw()**

3. **坐标系，左上角为坐标原点(0,0)，向右x轴，向下为y轴**

   以手指触摸的点为对象，getX()和getY()是获取该点在所在控件的横纵坐标；getLeft()、getRight()、getTop()、getBottom()是获取View到其父控件的距离；getRawX()、getRawY()是获取该点在屏幕上的坐标。

   ![1568856937752](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1568856937752.png)

   **View获取自身高度**：width=getRight()-getLeft()；height=getBottom()-getTop();或者直接getWidth()、getHeight()获取

4. **构造函数**

   无论继承系统View还是直接继承View，都需要对构造函数重写。构造函数有多个，至少重写一个。如CustomView

   ```java
   public class CustomView extends View{
       //在java代码里new的时候会用到
       public CustomView(Context context) {
           super(context);
       }
   
       //在xml布局文件中使用时自动调用
       public CustomView(Context context, @Nullable AttributeSet attrs) {
           super(context, attrs);
       }
   
       //不会自动调用，如果有默认style时，在第二个构造函数中调用
       public CustomView(Context context, @Nullable AttributeSet attrs, int defStyleAttr) {
           super(context, attrs, defStyleAttr);
       }
   
       //只有在API版本>21时才会用到
       //不会自动调用，如果有默认style时，在第二个构造函数中调用
       @RequiresApi(api = Build.VERSION_CODES.LOLLIPOP)
       public CustomView(Context context, @Nullable AttributeSet attrs, int defStyleAttr, int defStyleRes) {
           super(context, attrs, defStyleAttr, defStyleRes);
       }
   }
   
   ```

5. **自定义属性**

   **步骤：**

   * 定义属性，values/attrs.xml文件。在其中编写styleable和item等标签

   * 自定义View，在View的构造方法中通过TypeArray获取属性，java文件

   * 在局文件使用，layout文件

     ```xml
     <?xml version="1.0" encoding="utf-8"?>
     <!--自定义属性声明文件-->
     <resources>
         <declare-styleable name="custom">
             <attr name="text" format="string"/>
             <attr name="customAttr" format="integer"/>
         </declare-styleable>
     </resources>
     ```

     ```java
         //在xml布局文件中使用时自动调用
         public CustomView(Context context, @Nullable AttributeSet attrs) {
             super(context, attrs);
             //在View的构造函数中通过TypedArray获取
             TypedArray ta = context.obtainStyledAttributes(attrs, R.styleable.custom);
             String text = ta.getString(R.styleable.custom_text);
             int textAttr = ta.getInteger(R.styleable.custom_customAttr, -1);
             Log.e(TAG,"text="+text+"textAttr="+textAttr);
             ta.recycle();
         }
     ```

     ```xml
         <com.nero.customview.CustomView
             android:layout_width="wrap_content"
             android:layout_height="wrap_content"
             app:textAttr="200"
             app:text="i am a custom view"/>*
     ```

   **属性值的类型format**

   | 属性      | 含义                                                         |
   | --------- | ------------------------------------------------------------ |
   | reference | 参考某一资源ID                                               |
   | color     | 颜色值                                                       |
   | boolean   | 布尔值                                                       |
   | dimension | 尺寸值                                                       |
   | float     | 浮点值                                                       |
   | integer   | 整形值                                                       |
   | string    | 字符串                                                       |
   | fraction  | 百分数                                                       |
   | enum      | 枚举值,枚举类型的属性在使用的过程中只能同时使用其中一个，不能 android:orientation = “horizontal｜vertical" |
   | flag      | 位或运算,位运算类型的属性在使用的过程中可以使用多个值        |
   | 混合类型  | 属性定义时可以指定多种类型值                                 |

   属性定义：

   ```xml
   <declare-styleable name = "名称">
       <attr name = "background" format = "reference" />
       <attr name = "textColor" format = "color" />
       <attr name = "focusable" format = "boolean" />
       <attr name = "layout_width" format = "dimension" />
       <attr name = "fromAlpha" format = "float" />
       <attr name = "framesCount" format="integer" />
       <attr name = "text" format = "string" />
       <attr name = "pivotX" format = "fraction" />
       <attr name="orientation">
           <enum name="horizontal" value="0" />
           <enum name="vertical" value="1" />
       </attr>
       <attr name="gravity">
               <flag name="top" value="0x01" />
               <flag name="bottom" value="0x02" />
               <flag name="left" value="0x04" />
               <flag name="right" value="0x08" />
               <flag name="center_vertical" value="0x16" />
               ...
       </attr>
       <attr name = "background" format = "reference|color" />
   </declare-styleable>
   ```

   属性使用：

   ```xml
   <ImageView android:background = "@drawable/图片ID"/>
   <TextView android:textColor = "#00FF00" />
   <Button android:focusable = "true"/>
   <Button android:layout_width = "42dip"/>
   <alpha android:fromAlpha = "1.0"/>
   <animated-rotate android:framesCount = "12"/>
   <TextView android:text = "我是文本"/>
   <rotate android:pivotX = "200%"/>
   <LinearLayout  android:orientation = "vertical"></LinearLayout>
   <TextView android:gravity="bottom|left"/>
   <ImageView android:background = "@drawable/图片ID" />
   <ImageView android:background = "#00FF00" />
   ```

## 二、绘制流程

| 函数      | 作用                         | 相关方法                                     |
| --------- | ---------------------------- | -------------------------------------------- |
| measure() | 测量View的宽高               | measure(),setMeasuredDimension(),onMeasure() |
| layout()  | 计算当前View以及子View的位置 | layout(),onLayout(),setFrame()               |
| draw()    | 视图的绘制工作               | draw(),onDraw()                              |

1. **measure()**

   * MeasureSpec，是View的内部类，它封装了View的尺寸，其值保存在int值当中。一个int值有32位，前两位比是模式mode，后30位表示大小size。

     三种模式：
   
     | 模式        | 意义                                                         | 对应             |
     | ----------- | ------------------------------------------------------------ | ---------------- |
     | EXACTLY     | 精准模式，View需要一个精确值，这个值即为MeasureSpec当中的Size | match_parent     |
     | AT_MOST     | 最大模式，View的尺寸有一个最大值，View不可以超过MeasureSpec当中的Size值 | wrap_content     |
     | UNSPECIFIED | 无限制，View对尺寸没有任何限制，View设置为多大就应当为多大   | 一般系统内部使用 |
   
     使用方式：
   
     ```java
     // 获取测量模式（Mode）
     int specMode = MeasureSpec.getMode(measureSpec)
     // 获取测量大小（Size）
     int specSize = MeasureSpec.getSize(measureSpec)
     // 通过Mode 和 Size 生成新的SpecMode
     int measureSpec=MeasureSpec.makeMeasureSpec(size, mode);
     ```
   
   * onMeasure()，测量入口位于View的measure方法中，该方法做了一系列参数初始化后调用了onMeasure()方法
   
     ```java
     protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
         setMeasuredDimension(getDefaultSize(getSuggestedMinimumWidth(), widthMeasureSpec),
                     getDefaultSize(getSuggestedMinimumHeight(), heightMeasureSpec));
         }
     ```
   
   * ViewGroup的测量过程与View有一点点区别，它继承View但没有重写View的measure以及onMeasure方法。因为ViewGroup除了测量自身宽高还要测量各个子View的宽高，提供了测量子View的方法measureChildren()和measureChild()方法。
   
2. **Layout()**

   * layout()是整个Layout()流程的入口
   * setOpticalFrame(l,t,r,b)或setFrame(l,t,r,b)方法对View自身的位置进行设置
   * onLayout(changed,l,t,r,b)是ViewGroup对子View的位置进行计算

3. **Draw()**

   流程入口View中的draw()方法中。步骤如下：

   * 如有必要，绘制背景、保存当前Canvas
   * 绘制View的内容
   * 绘制子View
   * 如有必要，绘制边缘、阴影等效果。
   * 绘制装饰，如滚动条等
   
   

## 三、分类

* 自定义组合控件
* 继承系统View控件（TextView等）、系统ViewGroup（LinearLayout等），进行功能扩展
* 继承View、ViewGroup，进行功能定义

1. **自定义组合控件**

   * 编写布局文件

     ```xml
     <?xml version="1.0" encoding="utf-8"?>
     <RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
         android:layout_width="match_parent"
         android:id="@+id/header_root_layout"
         android:layout_height="45dp"
         android:background="#827192">
     
         <ImageView
             android:id="@+id/header_left_img"
             android:layout_width="45dp"
             android:layout_height="45dp"
             android:layout_alignParentLeft="true"
             android:paddingLeft="12dp"
             android:paddingRight="12dp"
             android:src="@drawable/back"
             android:scaleType="fitCenter"/>
     
         <TextView
             android:id="@+id/header_center_text"
             android:layout_width="wrap_content"
             android:layout_height="wrap_content"
             android:layout_centerInParent="true"
             android:lines="1"
             android:maxLines="11"
             android:ellipsize="end"
             android:text="title"
             android:textStyle="bold"
             android:textColor="#ffffff"/>
     
         <ImageView
             android:id="@+id/header_right_img"
             android:layout_width="45dp"
             android:layout_height="45dp"
             android:layout_alignParentRight="true"
             android:src="@drawable/add"
             android:scaleType="fitCenter"
             android:paddingRight="12dp"
             android:paddingLeft="12dp"/>
     
     </RelativeLayout>
     
     ```

     

   * 实现构造方法、初始化UI、提供对外的方法

     ```java
     public class HeaderView extends RelativeLayout{
         private ImageView img_left,img_right;
         private  TextView text_center;
         private RelativeLayout layout_root;
         private Context context;
         String element;
         private int showView;
         /**
          * 1）构造函数
          */
         public HeaderView(Context context) {
             super(context);
             this.context = context;
             initView(context);
         }
         public HeaderView(Context context, AttributeSet attrs) {
             super(context, attrs);
             this.context = context;
             initView(context);
             initAttrs(context, attrs);
         }
         public HeaderView(Context context, AttributeSet attrs, int defStyleAttr) {
             super(context, attrs, defStyleAttr);
             initView(context);
             initAttrs(context, attrs);
         }
         /**
          * 2）初始化UI，可根据业务需求设置默认值。
          */
         private void initView(Context context) {
             LayoutInflater.from(context).inflate(R.layout.view_header, this, true);
             img_left = (ImageView) findViewById(R.id.header_left_img);
             img_right = (ImageView) findViewById(R.id.header_right_img);
             text_center = (TextView) findViewById(R.id.header_center_text);
             layout_root = (RelativeLayout) findViewById(R.id.header_root_layout);
             layout_root.setBackgroundColor(Color.BLACK);
             text_center.setTextColor(Color.WHITE);
     
         }
         private void initAttrs(Context context, AttributeSet attrs) {
             TypedArray mTypedArray = context.obtainStyledAttributes(attrs, R.styleable.HeaderBar);
             //获取title_text属性
             String title = mTypedArray.getString(R.styleable.HeaderBar_title_text);
             if (!TextUtils.isEmpty(title)) {
                 text_center.setText(title);
             }
             //获取show_views属性，如果没有设置时默认为0x26
             showView = mTypedArray.getInt(R.styleable.HeaderBar_show_views, 0x26);
             text_center.setTextColor(mTypedArray.getColor(R.styleable.HeaderBar_title_text_clolor, Color.WHITE));
             mTypedArray.recycle();
             showView(showView);
     
         }
         private void showView(int showView) {
             //将showView转换为二进制数，根据不同位置上的值设置对应View的显示或者隐藏。
             Long data = Long.valueOf(Integer.toBinaryString(showView));
             element = String.format("%06d", data);
             for (int i = 0; i < element.length(); i++) {
                 if(i == 0) ;
                 if(i == 1) text_center.setVisibility(element.substring(i,i+1).equals("1")? View.VISIBLE:View.GONE);
                 if(i == 2) img_right.setVisibility(element.substring(i,i+1).equals("1")? View.VISIBLE:View.GONE);
                 if(i == 3) ;
                 if(i == 4) img_left.setVisibility(element.substring(i,i+1).equals("1")? View.VISIBLE:View.GONE);
                 if(i == 5) ;
             }
     
         }
         /**
          * 3）对外方法
          */
         //设置标题文字的方法
         private void setTitle(String title) {
             if (!TextUtils.isEmpty(title)) {
                 text_center.setText(title);
             }
         }
         //对左边按钮设置事件的方法
         private void setLeftListener(OnClickListener onClickListener) {
             img_left.setOnClickListener(onClickListener);
         }
         //对右边按钮设置事件的方法
         private void setRightListener(OnClickListener onClickListener) {
             img_right.setOnClickListener(onClickListener);
         }
     }
     ```

     

   * 布局中引用该控件

     ```xml
         <com.nero.customview.HeaderView
             android:layout_width="match_parent"
             android:layout_height="45dp"
             app:title_text="标题"
             app:show_views="center_text|left_img|right_img">
     
         </com.nero.customview.HeaderView>
     ```

   * 自定义属性

     ```xml
         <declare-styleable name="HeaderBar">
             <attr name="title_text_clolor" format="color"></attr>
             <attr name="title_text" format="string"></attr>
             <attr name="show_views">
                 <flag name="left_text" value="0x01" />
                 <flag name="left_img" value="0x02" />
                 <flag name="right_text" value="0x04" />
                 <flag name="right_img" value="0x08" />
                 <flag name="center_text" value="0x10" />
                 <flag name="center_img" value="0x20" />
             </attr>
         </declare-styleable>
     ```

2. **继承系统控件**

   ```java
   /**
    * 需求：为文字设置背景，并在布局中间添加一条横线。
    * 因为这种实现方式会复用系统的逻辑，大多数情况下我们希望复用系统的onMeaseur和onLayout流程，
    * 所以我们只需要重写onDraw方法 。
    */
   public class LineTextView extends android.support.v7.widget.AppCompatTextView {
       //定义画笔，用来绘制中心曲线
       private Paint mPaint;
       /**
        * 创建构造方法
        * @param context
        */
       public LineTextView(Context context) {
           super(context);
           init();
       }
       public LineTextView(Context context, @Nullable AttributeSet attrs) {
           super(context, attrs);
           init();
       }
       public LineTextView(Context context, @Nullable AttributeSet attrs, int defStyleAttr) {
           super(context, attrs, defStyleAttr);
           init();
       }
       private void init() {
           mPaint = new Paint();
           mPaint.setColor(Color.BLACK);
       }
       //重写draw方法，绘制我们需要的中间线以及背景
       @Override
       protected void onDraw(Canvas canvas) {
           super.onDraw(canvas);
           int width = getWidth();
           int height = getHeight();
           mPaint.setColor(Color.BLUE);
           //绘制方形背景
           RectF rectF = new RectF(0,0,width,height);
           canvas.drawRect(rectF,mPaint);
           mPaint.setColor(Color.BLACK);
           //绘制中心曲线，起点坐标（0,height/2），终点坐标（width,height/2）
           canvas.drawLine(0,height/2,width,height/2,mPaint);
       }
   }
   ```

3. **直接继承View**

   ```java
   /**
    * 1、在onDraw当中对padding属性进行处理。
    * 2、在onMeasure过程中对wrap_content属性进行处理。
    * 3、至少要有一个构造方法。
    */
   
   public class RectView extends View {
       //定义画笔
       private Paint mPaint = new Paint();
       public RectView(Context context) {
           super(context);
           init();
       }
       public RectView(Context context, @Nullable AttributeSet attrs) {
           super(context, attrs);
           init();
       }
       public RectView(Context context, @Nullable AttributeSet attrs, int defStyleAttr) {
           super(context, attrs, defStyleAttr);
           init();
       }
       private void init() {
           mPaint.setColor(Color.BLUE);
       }
       @Override
       protected void onDraw(Canvas canvas) {
           super.onDraw(canvas);
           //获取各个编剧的padding值
           int paddingLeft = getPaddingLeft();
           int paddingRight = getPaddingRight();
           int paddingTop = getPaddingTop();
           int paddingBottom = getPaddingBottom();
           //获取绘制的View的宽度
           int width = getWidth()-paddingLeft-paddingRight;
           //获取绘制的View的高度
           int height = getHeight()-paddingTop-paddingBottom;
           //绘制View，左上角坐标（0+paddingLeft,0+paddingTop），右下角坐标（width+paddingLeft,height+paddingTop）
           canvas.drawRect(0+paddingLeft,0+paddingTop,width+paddingLeft,height+paddingTop,mPaint);
       }
       /**
        * 在View的源码当中并没有对AT_MOST和EXACTLY两个模式做出区分，
        * 也就是说View在wrap_content和match_parent两个模式下是完全相同的，
        * 都会是match_parent，显然这与我们平时用的View不同，所以我们要重写onMeasure方法。
        */
       @Override
       protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
           super.onMeasure(widthMeasureSpec, heightMeasureSpec);
           int widthSize = MeasureSpec.getSize(widthMeasureSpec);
           int widthMode = MeasureSpec.getMode(widthMeasureSpec);
           int heightSize = MeasureSpec.getSize(heightMeasureSpec);
           int heightMode = MeasureSpec.getMode(heightMeasureSpec);
   
           //处理wrap_content的情况
           if (widthMode == MeasureSpec.AT_MOST && heightMode == MeasureSpec.AT_MOST) {
               setMeasuredDimension(300, 300);
           } else if (widthMode == MeasureSpec.AT_MOST) {
               setMeasuredDimension(300, heightSize);
           } else if (heightMode == MeasureSpec.AT_MOST) {
               setMeasuredDimension(widthSize, 300);
           }
       }
   }
   
   ```

4. **继承ViewGroup**

   除了对自身大小和位置进行测量之外，还需要对子View测量参数负责

   ```java
   /**
    * 实现一个类似于Viewpager的可左右滑动的布局。
    */
   
   public class HorizontalView extends ViewGroup {
   
       private int lastX;
       private int lastY;
   
       private int currentIndex = 0;
       private int childWidth = 0;
       private Scroller scroller;
       private VelocityTracker tracker;
   
   
       /**
        * 1.创建View类，实现构造函数
        * 实现构造方法
        * @param context
        */
       public HorizontalView(Context context) {
           super(context);
           init(context);
       }
   
       public HorizontalView(Context context, AttributeSet attrs) {
           super(context, attrs);
           init(context);
       }
   
       public HorizontalView(Context context, AttributeSet attrs, int defStyleAttr) {
           super(context, attrs, defStyleAttr);
           init(context);
       }
   
       private void init(Context context) {
           scroller = new Scroller(context);
           tracker = VelocityTracker.obtain();
       }
   
       /**
        * 2、根据自定义View的绘制流程，重写`onMeasure`方法，注意对wrap_content的处理
        * 重写onMeasure方法
        * @param widthMeasureSpec
        * @param heightMeasureSpec
        */
       @Override
       protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
           super.onMeasure(widthMeasureSpec, heightMeasureSpec);
           //获取宽高的测量模式以及测量值
           int widthMode = MeasureSpec.getMode(widthMeasureSpec);
           int widthSize = MeasureSpec.getSize(widthMeasureSpec);
           int heightMode = MeasureSpec.getMode(heightMeasureSpec);
           int heightSize = MeasureSpec.getSize(heightMeasureSpec);
           //测量所有子View
           measureChildren(widthMeasureSpec, heightMeasureSpec);
           //如果没有子View，则View大小为0，0
           if (getChildCount() == 0) {
               setMeasuredDimension(0, 0);
           } else if (widthMode == MeasureSpec.AT_MOST && heightMode == MeasureSpec.AT_MOST) {
               View childOne = getChildAt(0);
               int childWidth = childOne.getMeasuredWidth();
               int childHeight = childOne.getMeasuredHeight();
               //View的宽度=单个子View宽度*子View个数，View的高度=子View高度
               setMeasuredDimension(getChildCount() * childWidth, childHeight);
           } else if (widthMode == MeasureSpec.AT_MOST) {
               View childOne = getChildAt(0);
               int childWidth = childOne.getMeasuredWidth();
               //View的宽度=单个子View宽度*子View个数，View的高度=xml当中设置的高度
               setMeasuredDimension(getChildCount() * childWidth, heightSize);
           } else if (heightMode == MeasureSpec.AT_MOST) {
               View childOne = getChildAt(0);
               int childHeight = childOne.getMeasuredHeight();
               //View的宽度=xml当中设置的宽度，View的高度=子View高度
               setMeasuredDimension(widthSize, childHeight);
           }
       }
   
       /**
        * 3、接下来重写`onLayout`方法，对各个子View设置位置。
        * 设置子View位置
        * @param changed
        * @param l
        * @param t
        * @param r
        * @param b
        */
       @Override
       protected void onLayout(boolean changed, int l, int t, int r, int b) {
           int childCount = getChildCount();
           int left = 0;
           View child;
           for (int i = 0; i < childCount; i++) {
               child = getChildAt(i);
               if (child.getVisibility() != View.GONE) {
                   childWidth = child.getMeasuredWidth();
                   child.layout(left, 0, left + childWidth, child.getMeasuredHeight());
                   left += childWidth;
               }
           }
       }
   
       /**
        * 4、因为我们定义的是ViewGroup，从onInterceptTouchEvent开始。
        * 重写onInterceptTouchEvent,对横向滑动事件进行拦截
        * @param event
        * @return
        */
       @Override
       public boolean onInterceptTouchEvent(MotionEvent event) {
           boolean intercrpt = false;
           //记录当前点击的坐标
           int x = (int) event.getX();
           int y = (int) event.getY();
           switch (event.getAction()) {
               case MotionEvent.ACTION_MOVE:
                   int deltaX = x - lastX;
                   int delatY = y - lastY;
                   //当X轴移动的绝对值大于Y轴移动的绝对值时，表示用户进行了横向滑动，对事件进行拦截
                   if (Math.abs(deltaX) > Math.abs(delatY)) {
                       intercrpt = true;
                   }
                   break;
           }
           lastX = x;
           lastY = y;
           //intercrpt = true表示对事件进行拦截
           return intercrpt;
       }
   
       /**
        * 5、当ViewGroup拦截下用户的横向滑动事件以后，后续的Touch事件将交付给`onTouchEvent`进行处理。
        * 重写onTouchEvent方法
        * @param event
        * @return
        */
       @Override
       public boolean onTouchEvent(MotionEvent event) {
           tracker.addMovement(event);
           //获取事件坐标(x,y)
           int x = (int) event.getX();
           int y = (int) event.getY();
           switch (event.getAction()) {
               case MotionEvent.ACTION_MOVE:
                   int deltaX = x - lastX;
                   int delatY = y - lastY;
                   //scrollBy方法将对我们当前View的位置进行偏移
                   scrollBy(-deltaX, 0);
                   break;
               //当产生ACTION_UP事件时，也就是我们抬起手指
               case MotionEvent.ACTION_UP:
                   //getScrollX()为在X轴方向发生的便宜，childWidth * currentIndex表示当前View在滑动开始之前的X坐标
                   //distance存储的就是此次滑动的距离
                   int distance = getScrollX() - childWidth * currentIndex;
                   //当本次滑动距离>View宽度的1/2时，切换View
                   if (Math.abs(distance) > childWidth / 2) {
                       if (distance > 0) {
                           currentIndex++;
                       } else {
                           currentIndex--;
                       }
                   } else {
                       //获取X轴加速度，units为单位，默认为像素，这里为每秒1000个像素点
                       tracker.computeCurrentVelocity(1000);
                       float xV = tracker.getXVelocity();
                       //当X轴加速度>50时，也就是产生了快速滑动，也会切换View
                       if (Math.abs(xV) > 50) {
                           if (xV < 0) {
                               currentIndex++;
                           } else {
                               currentIndex--;
                           }
                       }
                   }
                   //对currentIndex做出限制其范围为【0，getChildCount() - 1】
                   currentIndex = currentIndex < 0 ? 0 : currentIndex > getChildCount() - 1 ? getChildCount() - 1 : currentIndex;
                   //滑动到下一个View
                   smoothScrollTo(currentIndex * childWidth, 0);
                   tracker.clear();
                   break;
           }
           lastX = x;
           lastY = y;
           return true;
       }
   
   
       private void smoothScrollTo(int destX, int destY) {
           //startScroll方法将产生一系列偏移量，从（getScrollX(), getScrollY()），destX - getScrollX()和destY - getScrollY()为移动的距离
           scroller.startScroll(getScrollX(), getScrollY(), destX - getScrollX(), destY - getScrollY(), 1000);
           //invalidate方法会重绘View，也就是调用View的onDraw方法，而onDraw又会调用computeScroll()方法
           invalidate();
       }
   
       //重写computeScroll方法
       @Override
       public void computeScroll() {
           super.computeScroll();
           //当scroller.computeScrollOffset()=true时表示滑动没有结束
           if (scroller.computeScrollOffset()) {
               //调用scrollTo方法进行滑动，滑动到scroller当中计算到的滑动位置
               scrollTo(scroller.getCurrX(), scroller.getCurrY());
               //没有滑动结束，继续刷新View
               postInvalidate();
           }
       }
   }
   
   ```

   ```xml
       <com.nero.customview.HorizontalView
           android:layout_width="match_parent"
           android:layout_height="wrap_content">
           <TextView
               android:layout_width="wrap_content"
               android:layout_height="wrap_content" />
           <TextView
               android:layout_width="wrap_content"
               android:layout_height="wrap_content" />
       </com.nero.customview.HorizontalView>
   ```

   

