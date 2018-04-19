# 自定义View

## Measure
重写OnMeasure(int widthMeasureSpec, int heightMeasureSpec)方法  
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

