### 一、主线程更新UI操作
1. Handler handler = new Handler();
   这个会默认用**当前线程**的Looper。
   
2. 如果在**其他线程**，也要满足这个功能的话，使用
	Handler handler = new Handler(Looper.getMainLooper());
	或：
   1. Looper.prepare();
   2. Handler handler = new Handler();
   3. Looper.loop();
   4. PS：小心内存泄漏
### 二、Git撤回commit
撤回commit，未push，修改的代码仍保留
git reset --soft HEAD^,(windows 会显示more)
git reset --soft HEAD~1（也可2、3...）

--mixed 
意思是：不删除工作空间改动代码，撤销commit，并且撤销git add . 操作
这个为默认参数,git reset --mixed HEAD^ 和 git reset HEAD^ 效果是一样的。
--soft  
不删除工作空间改动代码，撤销commit，不撤销git add . 
--hard
删除工作空间改动代码，撤销commit，撤销git add . 
注意完成这个操作后，就恢复到了上一次的commit状态。
顺便说一下，如果commit注释写错了，只是想改一下注释，只需要：
git commit --amend

此时会进入默认vim编辑器，修改注释完毕后保存就好了。

**git配置**
$ git config --lsit #显示配置信息
$ git config --global user.name "wuyoupeng"
$ git config --global user.email "1327756702@qq.com"


### 三、window层级属性

```
<item name="android:windowIsTranslucent">true</item>
设置后，
 <item name="android:windowBackground">@color/red_bg</item>不起作用
 启动App看见的黑屏
 <style name="LogoTheme" parent="android:style/Theme.NoTitleBar.Fullscreen">
<item name="android:windowBackground">@android:color/transparent</item>//背景是透明的
        <item name="android:windowIsTranslucent">true</item>//这里设置了半透明
        <item name="android:windowNoTitle">true</item>
        <item name="android:windowActionBar">false</item>
    </style>
```





























