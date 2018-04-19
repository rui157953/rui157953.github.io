## Welcome to GitHub Pages

You can use the [editor on GitHub](https://github.com/rui157953/ryan-blog/edit/master/README.md) to maintain and preview the content for your website in Markdown files.

Whenever you commit to this repository, GitHub Pages will run [Jekyll](https://jekyllrb.com/) to rebuild the pages in your site, from the content in your Markdown files.

### Markdown

Markdown is a lightweight and easy-to-use syntax for styling your writing. It includes conventions for

```markdown
Syntax highlighted code block

# Header 1
## Header 2
### Header 3

- Bulleted
- List

1. Numbered
2. List

**Bold** and _Italic_ and `Code` text

[Link](url) and ![Image](src)
```

For more details see [GitHub Flavored Markdown](https://guides.github.com/features/mastering-markdown/).

### Jekyll Themes

Your Pages site will use the layout and styles from the Jekyll theme you have selected in your [repository settings](https://github.com/rui157953/ryan-blog/settings). The name of this theme is saved in the Jekyll `_config.yml` configuration file.

### Support or Contact

Having trouble with Pages? Check out our [documentation](https://help.github.com/categories/github-pages-basics/) or [contact support](https://github.com/contact) and we’ll help you sort it out.

# 一个简单的五子棋程序
自己实现一个五子棋游戏程序。点击查看[全部源码](https://github.com/rui157953/rui157953.github.io/blob/master/note/%E4%BA%94%E5%AD%90%E6%A3%8B%E6%B8%B8%E6%88%8F%E7%A8%8B%E5%BA%8F.md).
## 自定义棋盘
自定义View绘制一个棋盘
```markdown
    public static final int MAX_COUNT_IN_LINE = 5; //5连珠
    /*黑棋悔棋次数*/
    public static int blackUndoCount = 3;   //黑棋悔棋次数
    /*白棋悔棋次数*/
    public static int whiteUndoCount = blackUndoCount;
    
    private float broadWidth ;  //棋盘宽
    private float broadHeight ; //棋盘高
    private final int lineNum = 17; //棋盘线
    private float lineWidth;    //棋盘线间距
    private float ratioPieceOfLineWidth = 4 * 1.0f / 5;  //棋子为行宽的4/5；
    private float pieceWidth ;  //棋子宽
    private Paint linePaint;    
    private Paint blackPiecePaint;
    private Paint whitePiecePaint;
    private Paint bitmapPaint;
    private ArrayList<Point> blackPiecis = new ArrayList<>();   //黑棋列表
    private ArrayList<Point> whitePiecis = new ArrayList<>();    //白棋列表
    private HashMap<Integer,Point> undoPiecis = new HashMap<>();    //悔棋列表
    private Bitmap pieceDown;
    private boolean isAnimationRunning;     //动画执行中
    private boolean isWhiteTurn;       //轮到白棋
    private boolean isGameOver;     //游戏结束
    private boolean isUndo; //悔棋

    private OnResultListener onResultListener;  //结果监听
    private ValueAnimator animator;     //落子动画
    private ValueAnimator UndoAnimator; //悔棋动画
    private float undoAnimatorProgress; 

    public GobangView(Context context) {
        this(context,null,0);
    }

    public GobangView(Context context, @Nullable AttributeSet attrs) {
        this(context, attrs,0);
    }

    public GobangView(Context context, @Nullable AttributeSet attrs, int defStyleAttr) {
        super(context, attrs, defStyleAttr);
        init();
    }

    public void setOnResultListener(OnResultListener onResultListener) {
        this.onResultListener = onResultListener;
    }

    private void init(){
        linePaint = new Paint(Paint.ANTI_ALIAS_FLAG);
        blackPiecePaint = new Paint(Paint.ANTI_ALIAS_FLAG);
        whitePiecePaint = new Paint(Paint.ANTI_ALIAS_FLAG);
        bitmapPaint = new Paint(Paint.ANTI_ALIAS_FLAG);

        linePaint.setColor(Color.BLACK);
        blackPiecePaint.setColor(Color.BLACK);
        blackPiecePaint.setStyle(Paint.Style.FILL);
        whitePiecePaint.setColor(Color.WHITE);
        whitePiecePaint.setStyle(Paint.Style.FILL);
        BitmapFactory.Options options = new BitmapFactory.Options();
        options.inJustDecodeBounds = true;
        pieceDown = BitmapFactory.decodeResource(getResources(),R.drawable.piece_down,options);
        animator = ValueAnimator.ofInt(new int[]{255,0});
        animator.setDuration(200);
        animator.addUpdateListener(new ValueAnimator.AnimatorUpdateListener() {
            @Override
            public void onAnimationUpdate(ValueAnimator animation) {
                bitmapPaint.setAlpha((int)animation.getAnimatedValue());
                invalidate();
            }
        });
        animator.addListener(new Animator.AnimatorListener() {
            @Override
            public void onAnimationStart(Animator animation) {
                isAnimationRunning = true;
            }

            @Override
            public void onAnimationEnd(Animator animation) {
                isAnimationRunning = false;
            }

            @Override
            public void onAnimationCancel(Animator animation) {
                isAnimationRunning = false;
            }

            @Override
            public void onAnimationRepeat(Animator animation) {
                isAnimationRunning = true;
            }
        });
        UndoAnimator = ValueAnimator.ofFloat(1f,0f);
        UndoAnimator.setDuration(500);
        UndoAnimator.addUpdateListener(new ValueAnimator.AnimatorUpdateListener() {
            @Override
            public void onAnimationUpdate(ValueAnimator animation) {
                Log.d("GobangView",animation.getAnimatedValue()+"");
                undoAnimatorProgress = (float) animation.getAnimatedValue();
                invalidate();
            }
        });
        UndoAnimator.addListener(new Animator.AnimatorListener() {
            @Override
            public void onAnimationStart(Animator animation) {
                isAnimationRunning = true;
                isUndo = true;
            }

            @Override
            public void onAnimationEnd(Animator animation) {
                isAnimationRunning = false;
                isUndo = false;
                undoPiecis.clear();
            }

            @Override
            public void onAnimationCancel(Animator animation) {
                isAnimationRunning = false;
                isUndo = false;
            }

            @Override
            public void onAnimationRepeat(Animator animation) {
                isAnimationRunning = true;
            }
        });
    } 
```
## 测量
```markdown
    @Override
    protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {

        int widthMode = MeasureSpec.getMode(widthMeasureSpec);
        int heightMode = MeasureSpec.getMode(heightMeasureSpec);
        int widthSize = MeasureSpec.getSize(widthMeasureSpec);
        int heightSize = MeasureSpec.getSize(heightMeasureSpec);

        int width = Math.min(widthSize, heightSize); //取最小边
        if (widthMode == MeasureSpec.UNSPECIFIED) {//宽不确定
            width = heightSize;
        } else if (heightMode == MeasureSpec.UNSPECIFIED) {//高不确定
            width = widthSize;
        }
        setMeasuredDimension(width,width);
    }

    @Override
    protected void onSizeChanged(int w, int h, int oldw, int oldh) {
        super.onSizeChanged(w, h, oldw, oldh);
        lineWidth = w*1.0f/lineNum; //线间距
        broadWidth = w - lineWidth; //棋盘宽
        broadHeight = h;    //棋盘高
        pieceWidth = lineWidth * ratioPieceOfLineWidth; //棋子宽
        BitmapFactory.Options options = new BitmapFactory.Options();
        options.inJustDecodeBounds = true;
        pieceDown = getBitmapFromRes(getResources(),R.drawable.piece_down,Math.round(lineWidth*2),Math.round(lineWidth*2)); //落子效果图片
    }
```
## 绘制
```markdown
 @Override
    protected void onDraw(Canvas canvas) {
        canvas.drawARGB(255,239,175,65); //棋盘背景
        drawGrid(canvas);   //棋盘网格
        drawPieces(canvas); //棋子
        checkGameOver();    //判断游戏是否结束
    }
    
    private void drawPieces(Canvas canvas) {
        float r = pieceWidth / 2;
        float startX = (lineWidth/2);
        Point point = null;

        //落子痕迹
        if (isUndo){
            if (undoPiecis.isEmpty())return;
            if (undoPiecis.containsKey(0)){
                point = undoPiecis.get(0);
                canvas.drawCircle(point.x*lineWidth+startX,point.y*lineWidth+startX, r*undoAnimatorProgress,blackPiecePaint);
            }
            if (undoPiecis.containsKey(1)){
                point = undoPiecis.get(1);
                canvas.drawCircle(point.x*lineWidth+startX,point.y*lineWidth+startX, r*undoAnimatorProgress,whitePiecePaint);
            }

        }else {
            if (isWhiteTurn){
                if (blackPiecis.size() > 0)
                    point = blackPiecis.get(blackPiecis.size() - 1);
            }else {
                if (whitePiecis.size() > 0)
                    point = whitePiecis.get(whitePiecis.size() - 1);
            }
            if (point!=null)
                canvas.drawBitmap(pieceDown,point.x*lineWidth-startX,point.y*lineWidth-startX,bitmapPaint);
            /////////////////////////////////////////////
        }
        for (int i = 0; i < blackPiecis.size(); i++) {
            point = blackPiecis.get(i);
            canvas.drawCircle(point.x*lineWidth+startX,point.y*lineWidth+startX, r,blackPiecePaint);
        }
        for (int i = 0; i < whitePiecis.size(); i++) {
            point = whitePiecis.get(i);
            canvas.drawCircle(point.x*lineWidth+startX,point.y*lineWidth+startX,r,whitePiecePaint);
        }
    }

    private void drawGrid(Canvas canvas) {
        float startX = (lineWidth/2);
        float stopX = broadWidth+startX;
        float dy = (broadHeight-broadWidth)/2-startX;
        for (int i = 0; i <= lineNum; i++) {
            float y = startX +lineWidth*i;
            canvas.drawLine(startX,y+dy,stopX,y+dy,linePaint); //画横线
            canvas.drawLine(y,startX+dy,y,stopX+dy,linePaint); //画竖线
        }
    }
```
## 判断胜利
```
private void checkGameOver() {
        CheckWinner cw = new CheckWinner();
        boolean blackWin = false;
        boolean whiteWin = false;
        if (isWhiteTurn){
            blackWin = cw.fivePiecesLinking(blackPiecis);
        }else {
            whiteWin = cw.fivePiecesLinking(whitePiecis);
        }

        if (blackWin||whiteWin){
            if (onResultListener!=null){
                onResultListener.onResult(blackWin,whiteWin);
            }
            isGameOver = true;
            animator.cancel();
            String text = blackWin?"黑棋胜！":"白棋胜";
            Toast.makeText(getContext(),text,Toast.LENGTH_SHORT).show();
        }
    }
```
## 处理落子位置
```

    @Override
    public boolean onTouchEvent(MotionEvent event) {
        if (isGameOver || isAnimationRunning){
            return false;
        }
        if (event.getAction() == MotionEvent.ACTION_UP){
            int x = (int) event.getX();
            int y = (int) event.getY();
            Point point = getValidPoint(x, y);

            if (blackPiecis.contains(point)||whitePiecis.contains(point)){
                return false;
            }
            if (isWhiteTurn){
                whitePiecis.add(point);
            }else {
                blackPiecis.add(point);
            }
            isWhiteTurn = !isWhiteTurn;
            startAnimation();
        }
        return true;
    }

    private Point getValidPoint(int x, int y) {
        int validX = (int)(x / lineWidth);
        int validY = (int)(y / lineWidth);
        return new Point(validX, validY);
    }
```
## 悔棋
```

    public void undoWhite(){
        if (isAnimationRunning)return;
        if (whiteUndoCount<=0)return;
        whiteUndoCount--;
        if (whitePiecis.size()<1){
            return;
        }
        if (isWhiteTurn){
            Point remove1 = whitePiecis.remove(whitePiecis.size() - 1);
            undoPiecis.put(1,remove1);
            if (!blackPiecis.isEmpty()){
                Point remove = blackPiecis.remove(blackPiecis.size() - 1);
                undoPiecis.put(0,remove);
            }
        }else {
            isWhiteTurn = true;
            Point remove = whitePiecis.remove(whitePiecis.size() - 1);
            undoPiecis.put(1,remove);
        }

        UndoAnimator.start();
    }
    public void undoBlack(){
        if (isAnimationRunning)return;
        if (blackUndoCount<=0)return;
        blackUndoCount--;
        if (blackPiecis.size()<1){
            return;
        }
        if (isWhiteTurn){
            isWhiteTurn = false;
            Point remove = blackPiecis.remove(blackPiecis.size() - 1);
            undoPiecis.put(0,remove);
        }else {
            Point remove = blackPiecis.remove(blackPiecis.size() - 1);
            undoPiecis.put(0,remove);
            if (!whitePiecis.isEmpty()){
                Point remove1 = whitePiecis.remove(whitePiecis.size() - 1);
                undoPiecis.put(1,remove1);
            }
        }
        UndoAnimator.start();
    }

```
## 添加动画
这段代码在init方法中
```
animator = ValueAnimator.ofInt(new int[]{255,0});
        animator.setDuration(200);
        animator.addUpdateListener(new ValueAnimator.AnimatorUpdateListener() {
            @Override
            public void onAnimationUpdate(ValueAnimator animation) {
                bitmapPaint.setAlpha((int)animation.getAnimatedValue());
                invalidate();
            }
        });
        animator.addListener(new Animator.AnimatorListener() {
            @Override
            public void onAnimationStart(Animator animation) {
                isAnimationRunning = true;
            }

            @Override
            public void onAnimationEnd(Animator animation) {
                isAnimationRunning = false;
            }

            @Override
            public void onAnimationCancel(Animator animation) {
                isAnimationRunning = false;
            }

            @Override
            public void onAnimationRepeat(Animator animation) {
                isAnimationRunning = true;
            }
        });
        UndoAnimator = ValueAnimator.ofFloat(1f,0f);
        UndoAnimator.setDuration(500);
        UndoAnimator.addUpdateListener(new ValueAnimator.AnimatorUpdateListener() {
            @Override
            public void onAnimationUpdate(ValueAnimator animation) {
                Log.d("GobangView",animation.getAnimatedValue()+"");
                undoAnimatorProgress = (float) animation.getAnimatedValue();
                invalidate();
            }
        });
        UndoAnimator.addListener(new Animator.AnimatorListener() {
            @Override
            public void onAnimationStart(Animator animation) {
                isAnimationRunning = true;
                isUndo = true;
            }

            @Override
            public void onAnimationEnd(Animator animation) {
                isAnimationRunning = false;
                isUndo = false;
                undoPiecis.clear();
            }

            @Override
            public void onAnimationCancel(Animator animation) {
                isAnimationRunning = false;
                isUndo = false;
            }

            @Override
            public void onAnimationRepeat(Animator animation) {
                isAnimationRunning = true;
            }
        });
```

## 判断胜利CheckWinner类
```
public class CheckWinner {


    public boolean fivePiecesLinking(ArrayList<Point> pieces){

        for (int i = 0; i < pieces.size(); i++) {
            Point point = pieces.get(i);
            if (check(point.x,point.y,pieces)){
                return true;
            }
        }
        return false;
    }

    private boolean check(int x, int y, ArrayList<Point> pieces) {
        int count = 1;
        Point point;
        //******************水平方向***************************//
        for (int i = 1; i < GobangView.MAX_COUNT_IN_LINE; i++) {
            point = new Point(x-i,y);
            if (pieces.contains(point)){
                count ++;
            }else {
                break;
            }
        }
        for (int i = 1; i < GobangView.MAX_COUNT_IN_LINE; i++) {
            point = new Point(x+i,y);
            if (pieces.contains(point)){
                count ++;
            }else {
                break;
            }
        }
        if (count>= GobangView.MAX_COUNT_IN_LINE){
            return true;
        }

        //*********************************************************//
        //******************垂直方向***************************//
        count = 1;
        for (int i = 1; i < GobangView.MAX_COUNT_IN_LINE; i++) {
            point = new Point(x,y-i);
            if (pieces.contains(point)){
                count ++;
            }else {
                break;
            }
        }
        for (int i = 1; i < GobangView.MAX_COUNT_IN_LINE; i++) {
            point = new Point(x,y+i);
            if (pieces.contains(point)){
                count ++;
            }else {
                break;
            }
        }
        if (count>= GobangView.MAX_COUNT_IN_LINE){
            return true;
        }
        //*********************************************************//
        //******************左斜方向***************************//
        count = 1;
        for (int i = 1; i < GobangView.MAX_COUNT_IN_LINE; i++) {
            point = new Point(x-i,y-i);
            if (pieces.contains(point)){
                count ++;
            }else {
                break;
            }
        }
        for (int i = 1; i < GobangView.MAX_COUNT_IN_LINE; i++) {
            point = new Point(x+i,y+i);
            if (pieces.contains(point)){
                count ++;
            }else {
                break;
            }
        }
        if (count>= GobangView.MAX_COUNT_IN_LINE){
            return true;
        }
        //*********************************************************//
        //******************右斜方向***************************//
        count = 1;
        for (int i = 1; i < GobangView.MAX_COUNT_IN_LINE; i++) {
            point = new Point(x+i,y-i);
            if (pieces.contains(point)){
                count ++;
            }else {
                break;
            }
        }
        for (int i = 1; i < GobangView.MAX_COUNT_IN_LINE; i++) {
            point = new Point(x-i,y+i);
            if (pieces.contains(point)){
                count ++;
            }else {
                break;
            }
        }
        if (count>= GobangView.MAX_COUNT_IN_LINE){
            return true;
        }
        //*********************************************************//
        return false;
    }
}
```
