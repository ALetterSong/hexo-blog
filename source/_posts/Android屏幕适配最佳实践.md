title: Android屏幕适配最佳实践
date: 2016-02-02 11:09:52
tags:
---

原文: [http://developer.android.com/training/multiscreen/index.html][1]
翻译: [http://hukai.me/android-training-course-in-chinese/ui/multiscreen/index.html][2]

## 支持不同屏幕尺寸
### 使用“wrap_content”和“match_parent”
如果你使用了wrap_content，view的宽和高会被设置为该view所包含的内容的大小值。如果是match_parent（在API 8之前是fill_parent）则会匹配该组件的父控件的大小。
### 使用相对布局（RelativeLayout）
如果你需要你的子view不只是简简单单的排成行的排列，更好的方法是使用RelativeLayout，它允许你指定你布局中控件与控件之间的关系，比如，你可以指定一个子view在左边，另一个则在屏幕的右边。
### 使用尺寸限定词
在编写布局文件时，将布局文件放在加上类似large，sw600dp等这样限定词的文件夹中，以此来告诉系统根据屏幕选择对应的布局文件
### 使用最小宽度限定词
最小宽度限定词允许你根据设备的最小宽度（dp单位）来指定不同布局。比如，传统的7寸平板最小宽度为600dp，如果你希望你的UI能够在这样的屏幕上显示两个窗格（不是一个窗格显示在小屏幕上），你可以使用上节中提到的使用同样的两个布局文件。不同的是，使用sw600来指定两个方框的布局使用在最小宽度为600dp的设备上。
### 使用布局别名
为了避免这些重复的文件，你可以使用别名文件。

 - res/values-large/layout.xml：

```
<resources>
    <item name="main" type="layout">@layout/main_twopanes</item>
</resources>
```

 - res/values-sw600dp/layout.xml：

```
<resources>
  <item name="main" type="layout">@layout/main_twopanes</item>
</resources>
```
### 使用方向限定词
有一些布局不管是在横向还是纵向的屏幕配置中都能显示的非常好，但是更多的时候，适当的调整一下会更好。

### 使用.9.png图片
如果你在使用组件时可以改变图片的大小，你很快就会发现这是一个不明确的选择。因为运行的时候，图片会被拉伸或者压缩（这样容易造成图片失真）。避免这种情况的解决方案就是使用点9图片，这是一种能够指定哪些区域能够或者不能够拉伸的特殊png文件。
你可以通过sdk中的draw9patch程序（位于tools/directory目录下）来画点9图片。通过沿左侧和顶部边框绘制像素来标记应该被拉伸的区域。也可以通过沿右侧和底部边界绘制像素来标记。我们可以利用右下边来控制内容与背景图边缘的padding。在右下边两个点距离上下左右四个方向的距离就是padding的距离。

## 支持不同的屏幕密度

### 使用密度独立像素（dp）
在指定单位的时候，通常使用dp或者sp。一个dp代表一个密度独立像素，也就相当于在160 dpi的一个像素的物理尺寸，sp也是一个基本的单位，不过它主要是用在文本尺寸上（它也是一种尺寸规格独立的像素），所以，你在定义文本尺寸的时候应该使用这种规格单位（不要使用在布尺寸上）。

### 提供可供选择的图片

为了提供更好的用户体验，你应该使用以下几种规格来缩放图片大小，为不同的屏幕密度提供相应的位图资源：

    xxxhdpi:4.0
    xxhdpi:3.0
    xhdpi:2.0
    hdpi:1.5
    mdpi:1.0(标准线)
    ldpi:0.75
    
## 实现自适应UI流
### 确定当前布局
由于每种布局的实现会略有差别，首先你可能要确定用户当前可见的布局是哪一个。比如，你可能想知道当前用户到底是处于“单窗格”的模式还是“双窗格”的模式。你可以通过检查指定的视图（view）是否存在和可见来实现：

    public class NewsReaderActivity extends FragmentActivity {
        boolean mIsDualPane;
    
        @Override
        public void onCreate(Bundle savedInstanceState) {
            super.onCreate(savedInstanceState);
            setContentView(R.layout.main_layout);
    
            View articleView = findViewById(R.id.article);
            mIsDualPane = articleView != null &&
                            articleView.getVisibility() == View.VISIBLE;
        }
    }
另一个关于如何适配不同组件是否存在的例子，是在组件执行操作之前先检查它是否是可用的。比如，在News Reader示例中，有一个按钮点击后打开一个菜单，但是这个按钮仅仅只在Android3.0之后的版本中才能显示（因为这个功能被ActionBar代替，在API 11+中定义）。所以，在给这个按钮添加事件之间，你可以这样做：

    Button catButton = (Button) findViewById(R.id.categorybutton);
    OnClickListener listener = /* create your listener here */;
    if (catButton != null) {
        catButton.setOnClickListener(listener);
    }

### 根据当前布局响应

    @Override
    public void onHeadlineSelected(int index) {
        mArtIndex = index;
        if (mIsDualPane) {
            /* display article on the right pane */
            mArticleFragment.displayArticle(mCurrentCat.getArticle(index));
        } else {
            /* start a separate activity */
            Intent intent = new Intent(this, ArticleActivity.class);
            intent.putExtra("catIndex", mCatIndex);
            intent.putExtra("artIndex", index);
            startActivity(intent);
        }
    }
    
### 在其他Activity中复用Fragment
当你在设计fragment的时候，非常重要的一点：不要为某个特定的activity设计耦合度高的fragment。通常的做法是，通过定义抽象接口，并在接口中定义需要与该fragment进行交互的activity的抽象方法，然后与该fragment进行交互的activity实现这些抽象接口方法。

    public class HeadlinesFragment extends ListFragment {
        ...
        OnHeadlineSelectedListener mHeadlineSelectedListener = null;
    
        /* Must be implemented by host activity */
        public interface OnHeadlineSelectedListener {
            public void onHeadlineSelected(int index);
        }
        ...
    
        public void setOnHeadlineSelectedListener(OnHeadlineSelectedListener listener) {
            mHeadlineSelectedListener = listener;
        }
    }
然后，当用户选择了一个headline item之后，fragment将通知对应的activity指定监听事件（而不是通过硬编码的方式去通知）：

    public class HeadlinesFragment extends ListFragment {
        ...
        @Override
        public void onItemClick(AdapterView<?> parent,
                                View view, int position, long id) {
            if (null != mHeadlineSelectedListener) {
                mHeadlineSelectedListener.onHeadlineSelected(position);
            }
        }
        ...
    }
    
### 处理屏幕配置变化

    /**
     * 返回当前屏幕是否为竖屏。
     * @param context
     * @return 当且仅当当前屏幕为竖屏时返回true,否则返回false。
     */
     public static boolean isScreenOriatationPortrait(Context context) {
     return context.getResources().getConfiguration().orientation == Configuration.ORIENTATION_PORTRAIT;
     }




  [1]: http://developer.android.com/training/multiscreen/index.html
  [2]: http://hukai.me/android-training-course-in-chinese/ui/multiscreen/index.html
