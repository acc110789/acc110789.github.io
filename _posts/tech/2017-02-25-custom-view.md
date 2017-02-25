---
title: 一个简单的Android自定义View
---

##{{page.title}}

用Android自定义view画了一个毕业设计中的图片，把源代码和贴上来。\\
总结一下Canvas的学习笔记。

~~~
package com.example.zhangxiaolong.customview;

import android.content.Context;
import android.graphics.Canvas;
import android.graphics.Color;
import android.graphics.DashPathEffect;
import android.graphics.Paint;
import android.graphics.Path;
import android.graphics.PathEffect;
import android.util.AttributeSet;
import android.view.View;

/**
 * Created by zhangxiaolong on 2017/2/25.
 */

public class ExpView extends View {
    /**
     * 点的半径
     */
    private int pointRadius = 5;
    private int smallRadius = 30;
    private int middleRadius = 2 * smallRadius;
    private int bigRadius = middleRadius + smallRadius;
    private final float density;
    private Paint mPaint = new Paint();
    private Path mPath = new Path();
    private PathEffect mPathEffect = new DashPathEffect(new float[]{20, 15}, 0);

    /**
     * 圆的宽度
     */
    private float width1 = 1f;
    /**
     * 线的宽度
     */
    private float width2 = 1.5f;

    private float lineLength = bigRadius + smallRadius / 2 + 20;
    /**
     * 箭头的长度
     */
    private float arrayLength = 8;

    private int dipTopix(float dip) {
        return (int) (density * dip + 0.5f);
    }

    public ExpView(Context context) {
        super(context);
        density = context.getResources().getDisplayMetrics().density;
    }

    public ExpView(Context context, AttributeSet attrs) {
        super(context, attrs);
        density = context.getResources().getDisplayMetrics().density;
    }

    public ExpView(Context context, AttributeSet attrs, int defStyleAttr) {
        super(context, attrs, defStyleAttr);
        density = context.getResources().getDisplayMetrics().density;
    }

    public ExpView(Context context, AttributeSet attrs, int defStyleAttr, int defStyleRes) {
        super(context, attrs, defStyleAttr, defStyleRes);
        density = context.getResources().getDisplayMetrics().density;
    }

    @Override
    protected void onDraw(Canvas canvas) {
        super.onDraw(canvas);
        //将canvas的中点设置为坐标原点
        canvas.translate(canvas.getWidth() / 2, canvas.getHeight() / 2);
        drawCircle(canvas);
        drawLine(canvas);

        //draw pcp节点,FILL是填充，会将图形的内部全部填充成相应的颜色
        mPaint.setStyle(Paint.Style.FILL);
        mPaint.setARGB(0xff, 75, 136, 203);
        float xPcp = 0;
        float yPcp = 0;
        canvas.drawCircle(xPcp,yPcp,dipTopix(pointRadius),mPaint);

        //B点
        float xB = dipTopix(bigRadius) * 2 / 5;
        float yB = dipTopix(bigRadius) * 3 / 5;
        canvas.drawCircle(xB,yB,dipTopix(pointRadius),mPaint);

        //C点
        float xC = dipTopix(bigRadius) * 3 / 5;
        float yC = -dipTopix(bigRadius) * 3 / 10;
        canvas.drawCircle(xC,yC,dipTopix(pointRadius),mPaint);

        //D点
        float xD = -dipTopix(bigRadius) * 2 / 5;
        float yD = -dipTopix(bigRadius) * 11 / 20;
        canvas.drawCircle(xD,yD,dipTopix(pointRadius),mPaint);

        //A点
        float xA = -dipTopix(bigRadius) * 2 / 5;
        float yA = dipTopix(bigRadius) * 3 / 4;
        canvas.drawCircle(xA,yA,dipTopix(pointRadius),mPaint);

        //pcp节点的文字
        mPaint.setColor(Color.RED);
        mPaint.setTextSize(dipTopix(12));
        mPaint.setFakeBoldText(true);
        canvas.drawText("PCP节点",xPcp - 50 ,yPcp - dipTopix(pointRadius) - 10,mPaint);

        //A节点的文案
        canvas.drawText("A(window=3)",xA - 80 ,yA - dipTopix(pointRadius) - 10,mPaint);

        //B节点的文案
        canvas.drawText("B",xB ,yB - dipTopix(pointRadius) - 10,mPaint);

        //C节点的文案
        canvas.drawText("C(window=8)",xC + dipTopix(pointRadius) + 10 ,
                yC,mPaint);

        //D节点的文案
        canvas.drawText("D(window=4)",xD - 80 ,yD - dipTopix(pointRadius) - 10,mPaint);
    }

    private void drawCircle(Canvas canvas) {
        //画三个圆圈，描边的边的宽度
        mPaint.setStrokeWidth(dipTopix(width1));
        //STROKE是描边，不会填充图形
        mPaint.setStyle(Paint.Style.STROKE);
        //画笔的颜色
        mPaint.setColor(Color.parseColor("#bcbcbc"));

        canvas.drawCircle(0, 0, dipTopix(smallRadius), mPaint);
        canvas.drawCircle(0, 0, dipTopix(middleRadius), mPaint);
        canvas.drawCircle(0, 0, dipTopix(bigRadius), mPaint);
    }

    private void drawLine(Canvas canvas) {
        canvas.save();

        mPaint.setStrokeWidth(dipTopix(width2));
        mPaint.setStyle(Paint.Style.STROKE);
        mPaint.setARGB(0xff, 75, 136, 203);
        mPaint.setPathEffect(mPathEffect);


        int iii = 0;
        final float x = 0;
        final float y = -dipTopix(lineLength);
        final int length = dipTopix(arrayLength);
        final float dx = (float) (length / 2 * Math.sqrt(2));
        final float dy = (float) (length / 2 * Math.sqrt(2));
        do {
            mPath.moveTo(0, -dipTopix(15));//15dp
            mPath.lineTo(x, y);
            mPaint.setPathEffect(mPathEffect);
            canvas.drawPath(mPath, mPaint);

            mPath.reset();
            mPath.moveTo(x,y);
            mPath.lineTo(x - dx,y + dy);
            mPaint.setPathEffect(null);
            canvas.drawPath(mPath,mPaint);

            mPath.reset();
            mPath.moveTo(x,y);
            mPath.lineTo(x + dx, y + dy);
            canvas.drawPath(mPath,mPaint);

            canvas.rotate(22.5f);
            iii++;
        } while (iii < 16);

        canvas.restore();
    }
}

~~~

~~~
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".MainActivity" >

    <com.example.zhangxiaolong.customview.ExpView
        android:id="@+id/line"
        android:layout_centerInParent="true"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        />

</RelativeLayout>
~~~

效果图：

<img src="/images/Screenshot_20170225-212817.png" width="360" height="640" alt="效果图" aligh="center"/>

总结：

1. 用同一个Path对象连续画两条路径，两条路径的PathEffect不一样，前一条\\
是实线，后一条是虚线(DashPathEffect),会出现问题(最后看到的现象就是两条Path都是实线)。\\
必须在第一条路径执行`canvas.drawPath()`之后，再执行`Path.reset()`,然后再绘制\\
第二条线就没有问题了。
2. 感觉并不是那么难，拿到一个图像之后，先进行分解基本的线，椭圆，圆，矩形，弧等元素，进行一些
运算，算出坐标，然后慢慢就绘制出来了。
