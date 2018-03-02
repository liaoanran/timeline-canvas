# 使用canvas编写时间轴插件 #


## **背景** ##
项目中有一个视频广场的功能，需要一个时间轴类似视频播放中进度条功能一样显示录像情况，并且可以点击、拖动、放大缩小展示时间轴，获取到时间轴的某个时间。原来的时间轴是使用了***timeslider*** 这个插件，原插件中是使用原生的js 绘制dom节点来显示时间轴，后面使用起来发现每一次重绘就要操作上百个dom节点，性能很差，所以决定采用canvas来重写时间轴。

## 效果如下 ##
![](https://i.imgur.com/7ZMccjo.png)

## **实现的功能** ##
**1. 绘制时间轴：**
上面包括刻度、录像段、时间点

**2. 点击/拖动时间轴：**
可以返回释放的时间点，控制台有打印值

**3. 放大缩小：**
在时间轴上滚动鼠标滚轮可以放大缩小时间轴

**4. 设置时间点：**
demo界面上有一个setTime 按钮可以设置 "2018-03-02 15:00:00" 这样格式的时间让时间轴的中间跳到这个点

**5. 显示播放点：**
中间是播放点，因为原来项目是要求直播和回放在同一时间轴显示，所以以中间来划分

**6. 改变录像段：**
demo界面上是有一个toggleCell这个按钮进行切换成另一段录像段

**7. hover显示时间：**
鼠标放在时间轴上可以显示时间

## 代码说明 ##
因为整篇主要是说canvas绘制，所以这里就简单说一下我使用到的canvas相关方法有哪些。
时间轴主要是由刻度、刻度时间、长方形录像段

**刻度：**
主要是add_graduations 这个方法，在这个方法中绘制是调用了drawLine方法，其实就是绘制一条线，如下：
<pre>
<code>
TimeSlider.prototype.drawLine = function(beginX,beginY,endX,endY,color,width){
	this.ctx.beginPath();
	this.ctx.moveTo(beginX,beginY);
	this.ctx.lineTo(endX,endY);
	this.ctx.strokeStyle = color;
	this.ctx.lineWidth = width;
	this.ctx.stroke();
}    
</code>
</pre>

**刻度时间：**
<pre>
<code>
    this.ctx.fillText(middle_date,graduation_left-20,30);
    this.ctx.fillStyle = "rgba(151,158,167,1)";
</code>
</pre>

**录像段：**
主要是add_cells这个方法，在这个方法中绘制是调用了draw_cell方法，其实就是绘制一个长方形，如下：
<pre>
<code>
    this.ctx.fillStyle = cell.style.background;
    this.ctx.fillRect(beginX,0,cell_width,15);
</code>
</pre>

其他基本上如果要绘制线例如中间红线、整体的框架线、hover的线都是调用drawLine方法

## 遇到的困难 ##
1、因为里面的刻度和录像段的绘制都是需要x轴距离，即距离左边的距离，所以这里可以通过

    距离=开始的偏移距离+格数*px/格 
这样的思想一个个值算出来，这里需要说明的一点是开始的偏移距离是因为最左边的那一个刻度不一定会和左侧边界重合，可能会向右一些，所以这一段距离需要算出来，通过下面得出
<pre>
<code>
 var ms_offset = _this.ms_to_next_step(start_timestamp,min_per_step*60*1000);

/**
 * 左侧开始时间的偏移，返回单位ms
 * @param {*} timestamp 
 * @param {*} step 
 */
TimeSlider.prototype.ms_to_next_step = function(timestamp, step) {
    var remainder = timestamp % step;
    return remainder ? step - remainder : 0;
}
</code>
</pre>

2、canvas绑定事件是通过addEventListener这个方法实现，因为这个时间轴没进行一次操作都是重绘canvas所以原来的绑定事件方法add_events都写在重绘方法里导致事件累加，浏览器内存爆了，然后就卡顿了。后面发现只要canvas初始化第一次绑定之后，其他的重绘不需要再绑定了，以此在init方法中会看到redrawFlag这个变量，是判断是否添加事件的，只有初始化需要添加

## 最后说明 ##
最后说一下这个插件里面现在总共有两个版本，带有1的是封装之后的版本，没有带1是最初简单版可以直接写入业务文件调用的。另外，因为这只是个demo，所以界面上setTime这个按钮的功能并没有对输入框时间做格式限制，只能输入“2018-03-02 15:00:00”这样的才有效，其余格式会出现NAN-NAN-NAN NAN：NAN：NAN的bug