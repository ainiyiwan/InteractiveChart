## Zoomer类值得学习的地方

#### 每个属性都有详细注释
```java
 /**
 * The interpolator, used for making zooms animate 'naturally.'
 */
private Interpolator mInterpolator;


/**
 * The total animation duration for a zoom.
 */
private int mAnimationDurationMillis;
```
#### 注释方式规范统一
```java
/**
 * The time the zoom started, computed using {@link SystemClock#elapsedRealtime()}.
 */
private long mStartRTC;
```
#### 每个方法及其自有属性都有详细注释
```java
/**
 * Forces the zoom finished state to the given value. Unlike {@link #abortAnimation()}, the
 * current zoom value isn't set to the ending value.
 *
 * @see android.widget.Scroller#forceFinished(boolean)
 */
public void forceFinished(boolean finished) {
    mFinished = finished;
}
```

## InteractiveLineGraphView类值得学习的地方
#### 类注释，阐明类的作用

#### 构造函数写法
```java
public InteractiveLineGraphView(Context context) {
    this(context, null, 0);
}

public InteractiveLineGraphView(Context context, AttributeSet attrs) {
    this(context, attrs, 0);
}

public InteractiveLineGraphView(Context context, AttributeSet attrs, int defStyle) {
    super(context, attrs, defStyle);
}
```
#### 属性写法
- 相同类型属性写在一起并且一起写注释
- 显而易见的属性不用写注释
- 属性命名见名知意
- 本地属性用m开头
这一些规范其实阿里的Java编程规范写的很详细，可以好好研究一番

#### 标准写法
```java
TypedArray a = context.getTheme().obtainStyledAttributes(
                attrs, R.styleable.InteractiveLineGraphView, defStyle, defStyle);

        try {
            mLabelTextColor = a.getColor(
                    R.styleable.InteractiveLineGraphView_labelTextColor, mLabelTextColor);
            mLabelTextSize = a.getDimension(
                    R.styleable.InteractiveLineGraphView_labelTextSize, mLabelTextSize);
            mLabelSeparation = a.getDimensionPixelSize(
                    R.styleable.InteractiveLineGraphView_labelSeparation, mLabelSeparation);

            mGridThickness = a.getDimension(
                    R.styleable.InteractiveLineGraphView_gridThickness, mGridThickness);
            mGridColor = a.getColor(
                    R.styleable.InteractiveLineGraphView_gridColor, mGridColor);

            mAxisThickness = a.getDimension(
                    R.styleable.InteractiveLineGraphView_axisThickness, mAxisThickness);
            mAxisColor = a.getColor(
                    R.styleable.InteractiveLineGraphView_axisColor, mAxisColor);

            mDataThickness = a.getDimension(
                    R.styleable.InteractiveLineGraphView_dataThickness, mDataThickness);
            mDataColor = a.getColor(
                    R.styleable.InteractiveLineGraphView_dataColor, mDataColor);
        } finally {
            a.recycle();
        }

        initPaints();
```
#### 奇妙的方法，之前见廖神用过，可以研究一下
ViewCompat.postInvalidateOnAnimation(MyInteractiveLineGraphView.this);
#### 阅读源码的正确姿势
比如我想学习他的自定义View的写法，那么我就只关注写法规范，其他的算法什么的本身就很复杂，直接忽略
不要对自己要求太高，要求自己每一行代码都能看懂，那是不可能的。别人研究很久的东西，你一下就看懂了，那你不是神人了。
#### 不要让一个方法太长，能够抽取的尽量抽取出来
#### 常用属性的get和set
```java
public int getDataColor() {
        return mDataColor;
    }

    public void setDataColor(int dataColor) {
        mDataColor = dataColor;
    }
```
#### 常用操作的封装
```java
/**
     * Smoothly zooms the chart in one step.
     */
    public void zoomIn() {
        mScrollerStartViewport.set(mCurrentViewport);
        mZoomer.forceFinished(true);
        mZoomer.startZoom(ZOOM_AMOUNT);
        mZoomFocalPoint.set(
                (mCurrentViewport.right + mCurrentViewport.left) / 2,
                (mCurrentViewport.bottom + mCurrentViewport.top) / 2);
        ViewCompat.postInvalidateOnAnimation(this);
    }
```
#### 视图状态的持久化，用来恢复视图状态（如屏幕旋转）
```java
 ////////////////////////////////////////////////////////////////////////////////////////////////
    //
    //     Methods and classes related to view state persistence.
    //
    ////////////////////////////////////////////////////////////////////////////////////////////////

    @Override
    public Parcelable onSaveInstanceState() {
        Parcelable superState = super.onSaveInstanceState();
        SavedState ss = new SavedState(superState);
        ss.viewport = mCurrentViewport;
        return ss;
    }

    @Override
    public void onRestoreInstanceState(Parcelable state) {
        if (!(state instanceof SavedState)) {
            super.onRestoreInstanceState(state);
            return;
        }

        SavedState ss = (SavedState) state;
        super.onRestoreInstanceState(ss.getSuperState());

        mCurrentViewport = ss.viewport;
    }

    /**
     * Persistent state that is saved by InteractiveLineGraphView.
     */
    public static class SavedState extends BaseSavedState {
        private RectF viewport;

        public SavedState(Parcelable superState) {
            super(superState);
        }

        @Override
        public void writeToParcel(Parcel out, int flags) {
            super.writeToParcel(out, flags);
            out.writeFloat(viewport.left);
            out.writeFloat(viewport.top);
            out.writeFloat(viewport.right);
            out.writeFloat(viewport.bottom);
        }

        @Override
        public String toString() {
            return "InteractiveLineGraphView.SavedState{"
                    + Integer.toHexString(System.identityHashCode(this))
                    + " viewport=" + viewport.toString() + "}";
        }

        public static final Creator<SavedState> CREATOR
                = ParcelableCompat.newCreator(new ParcelableCompatCreatorCallbacks<SavedState>() {
            @Override
            public SavedState createFromParcel(Parcel in, ClassLoader loader) {
                return new SavedState(in);
            }

            @Override
            public SavedState[] newArray(int size) {
                return new SavedState[size];
            }
        });

        SavedState(Parcel in) {
            super(in);
            viewport = new RectF(in.readFloat(), in.readFloat(), in.readFloat(), in.readFloat());
        }
    }
```
#### OnSizeChange中确定要画的视图大小
```java
@Override
    protected void onSizeChanged(int w, int h, int oldw, int oldh) {
        super.onSizeChanged(w, h, oldw, oldh);
        mContentRect.set(
                getPaddingLeft() + mMaxLabelWidth + mLabelSeparation,
                getPaddingTop(),
                getWidth() - getPaddingRight(),
                getHeight() - getPaddingBottom() - mLabelHeight - mLabelSeparation);
    }
```
#### 漂亮的注释分割线
```java
////////////////////////////////////////////////////////////////////////////////////////////////
//
//     Methods and objects related to drawing
//
////////////////////////////////////////////////////////////////////////////////////////////////
```