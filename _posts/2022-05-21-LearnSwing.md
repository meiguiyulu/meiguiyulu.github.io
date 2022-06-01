# Swing

## 1. GUI

图形用户界面（Graphical User Interface，简称 GUI，又称图形用户接口）是指采用图形方式显示的计算机操作用户界面。

**Java提供了三个主要包做GUI开发：**

- java.awt 包 – 主要提供字体/布局管理器
- javax.swing 包[商业开发常用] – 主要提供各种组件(窗口/按钮/文本框)
- java.awt.event 包 – 事件处理，后台功能的实现。



## 2. Swing组件

如图所示：swing组件主要可分为三个部分：

1. 顶层容器:：常用有JFrame，JDialog
2. 中间容器：JPanel，JOptionPane，JScrollPane，JLayeredPane 等，主要以panel结尾。
3. 基本组件：JLabel，JButton，JTextField，JPasswordField，JRadioButton 等

![img](https://img-blog.csdn.net/20180831162454432?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQyMDM1OTY2/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)





## 3. 顶层容器

| #    | 组件    | 描述                                                         |
| ---- | ------- | ------------------------------------------------------------ |
| 1    | JFrame  | 一个普通的窗口（绝大多数 Swing 图形界面程序使用 JFrame 作为顶层容器） |
| 2    | JDialog | 对话框                                                       |

### 3.1 JFrame类的常用方法

|      JFrame类的常用方法       |   类型   |                     描述                     |
| :---------------------------: | :------: | :------------------------------------------: |
|           JFrame()            | 构造方法 |            创建一个普通的窗体对象            |
|       JFrame(String a)        | 构造方法 |         创建一个窗体对象，并指定标题         |
| setSize(int width,int height) | 普通方法 |                 设置窗体大小                 |
|   setBackgorund(color.red)    | 普通方法 |               设置窗体背景颜色               |
|   setLocation(int x,int y)    | 普通方法 |              设置组件的显示位置              |
|     setLocation(point p)      | 普通方法 |        通过point来设置组件的显示位置         |
|    setVisible(true/false)     | 普通方法 |                显示或隐藏组件                |
|      add(Component comp)      | 普通方法 |               向容器中增加组件               |
| setLayout(LayoutManager mgr)  | 普通方法 |   设置局部管理器,如果设置为null表示不使用    |
|            pack()             | 普通方法 | 调整窗口大小，以适合其子组件的首选大小和局部 |
|       getContentpane()        | 普通方法 |             返回此窗口的容器对象             |

### 3.2 JFrame举例

```java
public class MyDemo {
    public static void main(String[] args) {
        JFrame frame = new JFrame("Swing Example");

        // 关闭窗口的时候退出整个程序
        frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

        // 窗口大小
        frame.setSize(400, 300);

        // 显示窗口
        frame.setVisible(true);
    }
}
```





## 4. 中间容器

| #    | 组件               | 描述                                       |
| ---- | ------------------ | ------------------------------------------ |
| 1    | JPanel (相当于div) | 一般轻量级面板容器组件(作为JFrame中间容器) |
| 2    | JScrollPane        | 带滚动条的，可以水平和垂直滚动的面板组件   |
| 3    | JSplitPane         | 分隔面板                                   |
| 4    | JTabbedPane        | 选项卡面板                                 |
| 5    | JLayeredPane       | 层级面板                                   |

### 4.1 JPanel

> 定义

JPanel 是 Java[图形用户界面](https://baike.baidu.com/item/图形用户界面/3352324)(GUI)工具包swing中的面板容器类，包含在javax.swing 包中，是一种[轻量级容器](https://baike.baidu.com/item/轻量级容器/3877340)，可以加入到[JFrame窗体](https://baike.baidu.com/item/JFrame窗体/15543587)中。JPanel默认的布局管理器是FlowLayout，其自身可以嵌套组合，在不同子容器中可包含其他组件(component),如JButton、JTextArea、JTextField 等，功能是对窗体上的这些控件进行组合。

> 常用方法

```java
JPanel panel = new JPanel(); // 初始化 创建面板
panel.setBorder(BorderFactory.createTitledBorder("标题"); // 给面板添加边框
panel.setLayout(); // 设置布局
panel.add(); //添加基本组件 例如按钮等。
```

> 简单例子

```java
public class MyDemo {
    public static void main(String[] args) {
        JFrame frame = new JFrame("Swing Example");

        // 关闭窗口的时候退出整个程序
        frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

        /*
        设置容器
         */
        JPanel rootPanel = new JPanel();
        frame.setContentPane(rootPanel);

        /*
        添加控件
         */
        JButton button = new JButton("测试");
        rootPanel.add(button);

        // 窗口大小
        frame.setSize(400, 300);

        // 显示窗口
        frame.setVisible(true);
    }
}
```





## 5. 基本组件

| #    |      组件      | 描述       |
| ---- | :------------: | ---------- |
| 1    |     JLabel     | 标         |
| 2    |    JButton     | 按         |
| 3    |  JRadioButton  | 单选按钮   |
| 4    |   JCheckBox    | 复选框     |
| 5    | JToggleButton  | 开关按钮   |
| 6    |   JTextField   | 文本框     |
| 7    | JPasswordField | 密码框     |
| 8    |   JTextArea    | 文本区域   |
| 9    |   JComboBox    | 下拉列表框 |
| 10   |     JList      | 列表       |
| 11   |  JProgressBar  | 进度条     |
| 12   |    JSlider     | 滑块       |

### 5.1 JLabel标签

> 定义

标签控件，用于显示文本。

> 例子

```java
JLabel label = new JLabel("JLabel标签");
rootPanel.add(label);

// 简化
rootPanel.add(new JLabel("JLabel标签"));
```

> 方法

```java
/*
 构造方法
*/
JLabel()// 创建无图像并且其标题为空字符串的 JLabel。
    
JLabel(Icon image)// 创建具有指定图像的 JLabel 实例。
    
JLabel(Icon image, int horizontalAlignment)//创建具有指定图像和水平对齐方式的 JLabel 实例。
    
JLabel(String text)// 创建具有指定文本的 JLabel 实例。

JLabel(String text, Icon icon, int horizontalAlignment)//创建具有指定文本、图像和水平对齐方式的 JLabel 实例。

JLabel(String text, int horizontalAlignment)//创建具有指定文本和水平对齐方式的 JLabel 实例。
    
/* JLabel中增加图片和文本 */
ImageIcon imageIcon = new ImageIcon("yourFile.gif");
JLabel label = new JLabel("Mixed", imageIcon, SwingConstants.RIGHT);
frame.add(label);

/* JLabel中增加HTML文本 */
JLabel label = new JLabel("<html>bold <br> plain</html>");
frame.add(label);

/*
常用方法
*/

void setText(String text) // 设置文本

void setIcon(Icon icon) // 设置图像

// 设置文本相对于图片的位置(文本默认在图片右边垂直居中)
void setHorizontalTextPosition(int textPosition)
void setVerticalTextPosition(int textPosition)

// 设置标签内容(在标签内)的对其方式(默认左对齐并垂直居中)
void setHorizontalAlignment(int alignment)
void setVerticalAlignment(int alignment)

// 设置文本的字体类型、样式 和 大小
void setFont(Font font)
```

### 5.2 JTextField

> 定义

单行文本框

> 例子

```java
/* 文本框 */
JTextField textField = new JTextField(20); // 20是设置文本框的长度
rootPanel.add(textField);
```

> 方法

```java
/* 构造函数 */
JTextField()    //用来创建一个默认的文本框
JTextField(String text)    //用来创建指定初始化信息(text)的文本框
JTextField(int columns)    //用来创建指定列数（colums）的文本框
JTextField(String text, int columns)    //结合上面两个，创建一个既有初始化信息，又指定列数的文本框

/* 设置文本 */
setText(str);
/* 获取文本 */
getText();
/* 设置字体 */
setFont();  // 例如 setFont(new Font("楷体", Font.BOLD, 0x12));
/* 设置文本框的水平对齐方式 */
setHorizontalAlignment() // 例如 setHorizontalAlignment(JTextField.CENTER);
/* 设置文本框的最多显示内容的列数 */
setColumns();
```

### 5.3 JCheckBox

> 定义

复选框

> 例子

```java
/* 复选框 */
JCheckBox checkBox = new JCheckBox("同意");
rootPanel.add(checkBox);
```

> 常用方法

```java
setSelected(true/false) // 设置是否选中
isSelected() // 是否选中
addActionListener() // 勾选或取消选中获取事件
    
/* 简单例子 */
JCheckBox checkBox = new JCheckBox("同意");
JButton button = new JButton("测试");
checkBox.addActionListener(new ActionListener() {
    @Override
    public void actionPerformed(ActionEvent e) {
        if (checkBox.isSelected())
            button.setEnabled(true);
        else button.setEnabled(false);
    }
});
/* 复选框 */
rootPanel.add(checkBox);
```

### JComboBox

> 定义

下拉列表

> 例子



> 方法

