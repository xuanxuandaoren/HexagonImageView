# 自定义六角形头像图片控件

##效果图
最近笔者的一个朋友又委托我帮忙写一个六角形展示，公司给出的效果图![公司给出的效果图](http://img.blog.csdn.net/20160816155235398)，朋友在网上找不到相应的Demo，于是向笔者求助，笔者最近又不是太忙，所以就欣然答应。于是笔者就随便在网上找了一个圆形头像显示图片随意改了一下，下面是效果图![效果图](http://img.blog.csdn.net/20160816155531467)
Demo下载地址[这里是六角形图片下载的地址](https://github.com/xuanxuandaoren/HexagonImageView)
参考项目的地址[这里是参考项目的地址](https://github.com/hdodenhof/CircleImageView)
其实该起来也是非常的简单，下面就由笔者交单介绍一下项目的关键代码
##项目的关键代码
###声明一个画笔并设置BitmapShader
首先声明一个画图片的画笔，再给画笔设置shader
```
//用于画图片的画笔
private BitmapShader mBitmapShader;
//创建一个BitmapShader对象，并从布局文件的src中获取图片对象，即mBitmap，并传递给BitmapShader
mBitmapShader = new BitmapShader(mBitmap, Shader.TileMode.CLAMP, Shader.TileMode.CLAMP);
mBitmapPaint.setAntiAlias(true);
//把BitmapShader设置给画笔
mBitmapPaint.setShader(mBitmapShader);
```
###计算绘图的区域
计算绘图的区域，是一个矩形，后期会根据矩形来确定六角形六个点的坐标
```
//计算绘制图片的区域，并设置给mBorderRect
/**计算绘图的区域及绘图的中心坐标*/
private final RectF mDrawableRect = new RectF();
mDrawableRect.set(calculateBounds());
mDrawableRadius = Math.min(mDrawableRect.height() / 2.0f, mDrawableRect.width() / 2.0f);

/**
 * 计算绘制图片的区域
 * @return
 */
private RectF calculateBounds() {int availableWidth  = getWidth() - getPaddingLeft() - getPaddingRight();
int availableHeight = getHeight() - getPaddingTop() - getPaddingBottom();
int sideLength = Math.min(availableWidth, availableHeight);
float left = getPaddingLeft() + (availableWidth - sideLength) / 2f;
float top = getPaddingTop() + (availableHeight - sideLength) / 2f;
return new RectF(left, top, left + sideLength, top + sideLength);
}
```
###绘制六边形
只要给画笔设置BitmapShader ，我们画什么样字的图形，图形所在位置的图片部分就可以显示出来，这时我们如果画个圆，图片就会成圆形，画个弧。图片就会成弧形，画个六角形，图片就会成六角形，所以你要更改图形的话就更改这里的代码
画六角形的话就用到path和canvas的drawpath方法了

```
//声明path对象
Path path=new Path();
//移动到定点
path.moveTo(mDrawableRect.centerX(),0);
//下面是点与点之间的连线
path.lineTo((float) (mDrawableRadius*(1.0f-Math.sin(Math.PI/3.0f))),mDrawableRadius/2.0f);
path.lineTo((float) (mDrawableRadius*(1.0f-Math.sin(Math.PI/3.0f))),mDrawableRadius*3/2.0f);        path.lineTo(mDrawableRect.centerX(),mDrawableRadius*2);
path.lineTo((float) (mDrawableRadius*(1.0f+Math.sin(Math.PI/3.0f))),mDrawableRadius*3/2.0f);
path.lineTo((float) (mDrawableRadius*(1.0f+Math.sin(Math.PI/3.0f))),mDrawableRadius/2.0f);
//连到第六个线时闭合路线
path.close();
//画六角形，就显示六角形图像
canvas.drawPath(path,mBitmapPaint);
```
##添加一些自定义的属性
###声明属性
在res文件夹下的values文件夹下创建一个attrs.xml文件
内容如下

```
<?xml version="1.0" encoding="utf-8"?>
<resources>
    <declare-styleable name="HexagonImageView">
        <attr name="civ_border_width" format="dimension" />
        <attr name="civ_border_color" format="color" />
        <attr name="civ_border_overlay" format="boolean" />
        <attr name="civ_fill_color" format="color" />
    </declare-styleable>
</resources>

```
###定义命名空间
使用时在布局文件的布局声明一下命名空间，即添加一下代码

```
xmlns:app="http://schemas.android.com/apk/res-auto"
```
末尾res-auto的意思在apk的res文件夹下自动寻找属性，这样我们就可以在控件里使用了

```
app:civ_border_color="@color/dark"
app:civ_border_width="2dp"
```
下面是布局文件的代码

```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical">

    <RelativeLayout
        android:layout_width="match_parent"
        android:layout_height="0dp"
        android:layout_weight="1"
        android:background="@color/light"
        android:padding="@dimen/base_padding">

        <com.example.hexagonimageview.view.HexagonImageView
            android:layout_width="160dp"
            android:layout_height="160dp"
            android:layout_centerInParent="true"
            android:src="@drawable/belle"
            app:civ_border_color="@color/dark"
            app:civ_border_width="2dp" />

    </RelativeLayout>

    <RelativeLayout
        android:layout_width="match_parent"
        android:layout_height="0dp"
        android:layout_weight="1"
        android:background="@color/dark"
        android:padding="@dimen/base_padding">

        <com.example.hexagonimageview.view.HexagonImageView
            android:layout_width="160dp"
            android:layout_height="160dp"
            android:layout_centerInParent="true"
            android:src="@drawable/belle"
            app:civ_border_color="@color/light"
            app:civ_border_width="2dp" />

    </RelativeLayout>

</LinearLayout>
```
###在代码里处理属性
以上三步配置的属性并不起作用，我们只有在代码里处理才有用处我们一般会在构造函数里去处理，代码如下

```
public HexagonImageView(Context context, AttributeSet attrs, int defStyle) {super(context, attrs, defStyle);
TypedArray a = context.obtainStyledAttributes(attrs, R.styleable.HexagonImageView, defStyle, 0);
mBorderWidth=a.getDimensionPixelSize(R.styleable.HexagonImageView_civ_border_width, DEFAULT_BORDER_WIDTH);
mBorderColor=a.getColor(R.styleable.HexagonImageView_civ_border_color, DEFAULT_BORDER_COLOR);
mBorderOverlay=a.getBoolean(R.styleable.HexagonImageView_civ_border_overlay, DEFAULT_BORDER_OVERLAY);
mFillColor=a.getColor(R.styleable.HexagonImageView_civ_fill_color, DEFAULT_FILL_COLOR);
a.recycle();
init();
}
```
这样就可以获取在布局文件里配置的属性并进行处理了，这样一个完整的六角形自定义图片空间就完成了
##注意
demo使用时需把res-values-attrs.xml的文件拷贝到对应的项目的res-values文件夹下
app:civ_border_color="@color/dark"是指图片边框的颜色
app:civ_border_width="2dp"是指图片边框的高度





