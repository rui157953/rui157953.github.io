# 自定义View

## Measure
重写OnMeasure(int widthMeasureSpec, int heightMeasureSpec)方法  
### MeasureSpec
```
int widthMode = MeasureSpec.getMode(widthMeasureSpec);  
int widthSize = MeasureSpec.getSize(widthMeasureSpec);
```

