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

#一个简单的五子棋程序
##自定义棋盘
自定义View绘制一个棋盘
`
    public static final int MAX_COUNT_IN_LINE = 5; //5连珠
    /**黑棋悔棋次数*/
    public static int blackUndoCount = 3;   //黑棋悔棋次数
    /**白棋悔棋次数*/
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
//    private int[][] Piecis= new int[15][15];
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
//        animator = ValueAnimator.ofInt(bitmapPaint,"Alpha",);
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
    
##测量
##绘制
##判断胜利
##悔棋
##添加动画


