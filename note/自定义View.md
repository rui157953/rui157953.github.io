# 自定义View

## Measure
重写` OnMeasure(int widthMeasureSpec, int heightMeasureSpec)`方法.
### MeasureSpec
```
int mode = MeasureSpec.getMode(measureSpec);  
int size = MeasureSpec.getSize(measureSpec);
```
mode三种模式：
- UNSPECIFIED  
父容器不对View有任何的限制，要多大给多大，这种情况一般用于系统内部，表示一种测量状态。
- EXACTLY  
父容器已经检测出View所需要的精确大小，这个时候View的最终大小就是SpecSize所指定的值，它对应于match_parent和具体数值这两种模式。
- AT_MOST  
父容器指定了一个可用大小即SpecSize，View的大小不能大于这个值，具体是什么要看不同View的具体实现，它对应于wrap_content。  

子容器MeasureSpec受父容器影响，规则如下：  


| &nbsp;&nbsp;&nbsp;&nbsp;&nbsp; \ parentSpecMode <br/> chlidLayoutParams | EXACTLY| AT_MOST| UNSPECIFIED|
| --------------------------------| -------| -------| -----------|
| dp/px| EXACTLY <br/> childSize| EXACTLY  <br/>childSize| EXACTLY <br/>childSize|
| match_parent| EXACTLY <br/>parentSize| AT_MOST <br/>parentSize| UNSPECIFIED <br/>0|
| wrap_content| AT_MOST <br/>parentSize| AT_MOST <br/>parentSize| UNSPECIFIED <br/>0|  


#### ViewGroup
viewGroup 测量方法跟View 不同，在ViewGroup 中需遍历所有子元素的measure 方法，然后获取子View 的测量值再根据业务需求确定自己的宽高。  
调用 ` setMeasuredDimension(int measuredWidth, int measuredHeight)`方法保存测量后的尺寸。    
获取View 最终测量值最好在` onLayout`方法中。

## Layout
重写` onLayout(boolean changed, int left, int top, int right, int bottom)`方法。
此方法只在ViewGroup 中有意义，通过重写此方法对ViewGrope 中的子元素进行布局。  

## Draw
重写` onDraw(Canvas canvas)`方法。  
其他Draw 方法：
- ` dispatchDraw(Canvas canvas)`
该方法用于绘制子View ，在onDraw 之后调用。
- ` onDrawForeground(Canvas canvas)` 
该方法是api 23之后才有，用于绘制前景 ，dispatchDraw 之后调用。




