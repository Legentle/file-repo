# UI开发
## 常用控件
- TextView
  在界面上显示一段文本信息
  使用android:gravity来指定文字的对齐方式，可选值有top、bottom、left、right、center等，可以用“|”来同时指定多个值
  通过android:textSize属性可以指定文字的大小，通过android:textColor属性可以指定文字的颜色，在Android中字体大小使用sp作为单位。
- Button
  用于和用户进行交互的一个重要控件
  系统会对Button中的所有英文字母自动进行大写转换，可以通过Android:textAllCaps="false"禁用
  对按钮点击事件监听，启动相依功能
- EditText
  允许用户在控件里输入和编辑内容，并可以在程序中对这些内容进行处理
  可以使用android:maxLines来限制最大显示行数，当输入的内容超过时，文本会向上滚动
  还可以结合使用EditText与Button来完成一些功能，比如通过点击按钮来获取EditText中输入的内容：首先通过findViewById()方法得到EditText的实例，然后在按钮的点击事件里调用EditText的getText()方法获取到输入的内容，再调用toString()方法转换成字符串，最后使用Toast将输入的内容显示出来。
- ImageView
  用于在界面上展示图片的一个控件
- ProgressBar
  用于在界面上显示一个进度条
  通过style属性可以将它指定成水平进度条
- AlertDialog
  AlertDialog可以在当前的界面弹出一个对话框，这个对话框是置顶于所有界面元素之上的，能够屏蔽掉其他控件的交互能力，因此AlertDialog一般都是用于提示一些非常重要的内容或者警告信息
- ProgressDialog
  ProgressDialog和AlertDialog有点类似，都可以在界面上弹出一个对话框，都能够屏蔽掉其他控件的交互能力。不同的是，ProgressDialog会在对话框中显示一个进度条，一般用于表示当前操作比较耗时，让用户耐心地等待。

## 4种基本布局
1 线性布局
2 相对布局
3 帧布局
4 百分比布局

## 自定义控件
## 最常用和最难用的控件——ListView
## 更强大的滚动控件——RecyclerView
