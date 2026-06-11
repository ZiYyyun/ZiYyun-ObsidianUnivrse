# Swing

## 窗口

### 1.新建窗口

```java
JFrame jf = new JFrame();//新建一个窗口对象
jf.setVisible(true);//使窗口呈现可见状态
jf.setDefaultCloseOperation(3);//窗口在被关闭时结束进程
```

### 2.窗口的尺寸与位置

```java
jf.setSize(500,400);//窗口的宽和高
jf.setLocation(300,400);//窗口的位置（x坐标，y坐标）
jf.setLocationRelativeTo(null);//窗口位于屏幕正中
jf.setAlwaysOnTop(true);//窗口始终置顶

```

### 3.窗口的标题和图标

```java
jf.setTitle("用户登录");//设置窗口标题
jf.setIconImage();//设置窗口图标
```

### 4.关闭时的操作

```java
frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
```

> frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE)是设置窗口关闭操作的方法。它告诉程序在用户关闭窗口时要执行的操作。
>
> JFrame.EXIT_ON_CLOSE是一个常量，表示在关闭窗口时退出应用程序。当用户点击窗口的关闭按钮时，程序将终止运行并退出。
>
> 还有其他几个可选的关闭操作：
>
> - JFrame.HIDE_ON_CLOSE：隐藏窗口，但不终止程序的运行。
> - JFrame.DISPOSE_ON_CLOSE：释放窗口资源，但不终止程序的运行。
> - JFrame.DO_NOTHING_ON_CLOSE：不执行任何操作，需要通过编程来处理窗口关闭事件。
>
> 通常情况下，我们使用JFrame.EXIT_ON_CLOSE来确保在关闭窗口时退出应用程序。

## 按钮

> ​	JButton（按钮）是一个可点击的组件，通常用于触发某些操作或事件。可以通过设置按钮的文本、图标、背景色等属性来自定义按钮的外观。

### 1.按钮的创建与添加

```java
JButton jb = new JButton("登录");//创建一个带文本的按钮
jb.setVisible(true);//使按钮可见
jb.setText("按钮");//添加按钮文字
 jf.add(jb);//添加按钮到jb窗口
```

### 2.按钮的尺寸与位置

```java
jb.setBounds(10,20,4,5);//x,y,weight,height
```

## 事件监听器

> 当用户点击按钮时，可以通过添加ActionListener来监听按钮的动作，并在相应的事件处理方法中执行所需的操作。

```java
jb.addActionListener(new ActionListener() {
            @Override
            public void actionPerformed(ActionEvent e) {
                System.out.println("Clicked!!!");
            }
        });
```

## 标签

> JLabel（标签）是一个用于显示文本或图像的组件。可以通过设置标签的文本、字体、前景色等属性来自定义标签的外观。标签通常用于显示静态信息，而不会触发任何事件。

### 1.JLable的创建

```java
 JLabel label = new JLabel("Hello, World!");
```

### 2.JLable的添加

```java
frame.getContentPane().add(label);
```

## 文本框

> JTextField是Java Swing库中的一个文本输入框组件，用于接收用户的文本输入。
>
> 通过JTextField，用户可以在图形用户界面（GUI）中输入和编辑文本。可以设置文本框的初始文本、大小、字体等属性，并可以通过添加ActionListener来监听文本框的动作事件，例如按下回车键或失去焦点。

### 1.文本框的创建

```java
 JTextField textField = new JTextField();
```

### 2.文本框的事件

```java
 textField.addActionListener(new ActionListener() {
            @Override
            public void actionPerformed(ActionEvent e) {
                // 处理文本框的动作事件的代码
                String text = textField.getText();
                System.out.println("Entered text: " + text);
            }
        });
```

## 复选框





## 单选按钮





## 下拉列表





## 列表
