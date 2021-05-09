# Tomcat9乱码

打开/bin/startup.bat发现启动日志就已经有乱码

![image-20210414160446458](.\assets.md\tomcat乱码01.png)

修改/conf/logging.properties中

![image-20210414160918762](.\assets.md\tomcat乱码02.png)

Tomcat启动就不会乱码了

## req.getParameter("参数名")乱码的问题

[上面的Tomcat乱码解决方案](#Tomcat9乱码)虽然解决了Tomcat启动的乱码问题，但是在form提交数据，servlet使用req.getParameter获取参数值的时候还是出现了乱码，使用网上的解决方法（如下）还是不能解决乱码问题。

1. 设置Servlet的编码

```java
 request.setCharacterEncoding("utf-8");
 response.setContentType("text/html;charset=utf-8");
```

2. 设置req.getParameter编码

```java
String str = new String(request.getParameter("参数名").getBytes("iso-8859-1"), "utf-8");
```

3. 设置Tocmat server.xml 增加URIEncoding属性

```xml
<Connector port="8080" protocol="HTTP/1.1"   
    connectionTimeout="20000"   
    redirectPort="8444"   
    URIEncoding="UTF-8" />
```

**正确的解决方案：**

IDEA - Help - Edit Custom VM Options中设置 `-Dfile.encoding=utf-8`如果以此法设置req.getParam()就不会出现乱码，但是Tomcat启动时控制台产生乱码要把上面[解决Tomcat启动乱码](#Tomcat9乱码)的方法回滚GBK设置为默认的UTF-8就可以了

## （参考）网上的一篇关于解决乱码的文章 [点此超链接](https://blog.csdn.net/newObject__/article/details/103339342)

### 1. 和 请求/响应无关 的中文乱码问题

这里要记录一下，使用 tomcat 在 servlet 中随便打印一行中文数据都出现乱码啊啊啊啊啊啊！
sout，serr这种情况，居然都有乱码？这明明和 请求/响应 无关啊！

面向百度编程------>
得到的结果是：这个问题要么修改 tomcat 的配置文件，要么在IDEA中tomcat的配置中
有个 VM Options，给他填充一个数据： 

```properties
-Dfile.encoding=utf-8
```

配置文件我没有改，懒得去找path了，直接改 VM Options 方便嘛！
控制我们自己打印的数据信息 中文乱码问题 完美解决

### 2. tomcat 启动时，还有日常的日志打印 中文乱码问题

这个不得已，必须得去改配置文件了，在 tomcat 的安装路径下，找到conf文件夹，里面有个 logging.properties 配置文件，我们用记事本打开它，然后在

```properties
java.util.logging.ConsoleHandler.level = FINE
java.util.logging.ConsoleHandler.formatter = org.apache.juli.OneLineFormatter
```
这两行的下面添加一行配置：

```properties
java.util.logging.ConsoleHandler.encoding = UTF-8
```
完美解决日志信息的中文乱码问题

### 3. servlet 中 请求/响应 中文乱码问题解决
这里请求的话就只分为 get/post 这两种最常用的方式 去解决中文乱码

#### get请求

get请求没有请求报文，只能在url上传输参数，那么出现乱码的第一个地方：在我们填写完url后，浏览器传给tomcat，tomcat解析的时候就不老实地按照utf-8解析，导致我们拿到数据就是乱码的然后 resp.getWrite().write(param) 打在页面上，自然而然就是乱码。
如何解决tomcat解析URI时出现的乱码问题呢？

改配置文件！
 还是 tomcat 的安装目录下的 cong 文件夹，找到server.xml,然后在 <Connector> 标签中，拼接参数： URIEncoding="UTF-8"  (注意：这里是URI不是URL)

**这个做法，仅适用于 tomcat 8 以前的版本！tomcat 8 以后的版本，自动设置了这个参数，如果我们强行添加，反而在 tomcat 解析 URI的时候 出现乱码**
tomcat 以utf-8正常解析参数后，我们使用 resp.getWrite().write() 给页面打印中文时候还出现乱码！
这种情况就要说我们没有给定响应的解析格式（暂时这么理解吧），我们也不知道它是按照哪儿种编码格式解析响应中的数据，我们就一劳永逸的直接设置响应的解析格式：

在doGet方法中加上这两行代码：

```java
resp.setContentType("text/html;charset=utf-8");// 这个是告诉浏览器，服务器返回的响应是什么格式的文件
resp.setCharacterEncoding("utf-8");// 这个是告诉浏览器，服务器返回的响应你要以什么编码格式解析
```

#### post请求

post请求是含有请求报文的，我们在doPost方法直接加上三行代码

```java
req.setCharacterEncoding("utf-8");// 这个是告诉服务器，请求报文以什么编码格式解析，这里是utf-8
resp.setContentType("text/html;charset=utf-8");// 这个是告诉浏览器，服务器返回的响应是什么格式的文件,这里是text/html
resp.setCharacterEncoding("utf-8");// 这个是告诉浏览器，服务器返回的响应你要以什么编码格式解析，这里是utf-8
```


综上，tomcat常见的中文乱码导致原因

1. 浏览器向服务器发送请求时，服务器解析请求出现乱码；

2. 服务器向浏览器返回响应时，出现乱码

3. 在 tomcat 的环境下，jvm出现中文乱码（-Dfile.encoding=utf-8）

知道了乱码原因，对于 tomcat + servlet + jsp 来说，挨个步骤排查就很好解决了，这里注意一点就是 tomcat 8 以后的版本，在浏览器get请求服务器的时候，URI上的编码格式，默认就是UTF-8，已经不需要我们修改配置文件了，但是 tomcat 8 之前，还是要在 server.xml 中添加 URIEncoding=“UTF-8”

# IDEA 快捷键

- Ctrl + Alt + L 格式化代码
- 自动补全Html标签 比如 输入`div`后按`TAB键`会自动生成`<div></div>`


# Day01：Markdown语法
## 标题

#+空格+文字    =>  一级标题

##+空格+文字  =>  二级标题

###+空格+文字  =>  三级标题

## 字体

*1个\*号为斜体*

**2个\*号为粗体**

***3个\*号为斜体+粗体***

~~2个英文波浪号为废弃字体~~

## 分割线 
三个-号

---

或者三个\*号
***

## 引用

> 1个\>号后接文字表示引用

## 超链接

**格式：\[文字提示\]\(超链接路径\) **

[点击我跳转到辉煌的未来](https://www.hpr.com)

**跳转到其他标题**

[6.1.AWS管理控制台](#6.1.AWS管理控制台)

语法为：`[链接显示的内容](#某个标题)`

注意：

()括号里面是#井号后面直接跟标题的名字，我发现只能跳转到文中某个标题的位置，不能跳转到文中某段文字的位置。

## 图片

**格式：\!\[图片标题\]\(图片网络或者本地路径\) **

![图片标题](https://img1.doubanio.com/lpic/s28969587.jpg)

## 表格

**格式： **

标题1|标题2|标题3

--|--|--

值1|值2|值3

*注意：每行之间不能有空行，可在源码中查看是否有空行*

名字|性别|年龄
--|--|--
张三|男|31

## 代码块

```java
//3个`号加java表示java代码块  tab键上面那个英文符号
public static void main(){
	...   
}
```

## 列表

1. 数字1+.号+空格
2. 敲下回车自动生成2
3. ...
4. ...



- -号+空格就生成了无数字列表
- 敲下回车自动生成
- ...

# Github

## github搜索

| 限定符              | 示例                                                         |
| :------------------ | :----------------------------------------------------------- |
| `in:name`           | [**jquery in:name**](https://github.com/search?q=jquery+in%3Aname&type=Repositories) 匹配仓库名称中含有 "jquery" 的仓库。 |
| `in:description`    | [**jquery in:name,description**](https://github.com/search?q=jquery+in%3Aname%2Cdescription&type=Repositories) 匹配仓库名称或说明中含有 "jquery" 的仓库。 |
| `in:readme`         | [**jquery in:readme**](https://github.com/search?q=jquery+in%3Areadme&type=Repositories) 匹配仓库自述文件中提及 "jquery" 的仓库。 |
| `repo:owner/name`   | [**repo:octocat/hello-world**](https://github.com/search?q=repo%3Aoctocat%2Fhello-world) 匹配特定仓库名称。 |
| `stars:n`           | [**stars:500**](https://github.com/search?utf8=✓&q=stars%3A500&type=Repositories) 匹配恰好具有 500 个星号的仓库,[**stars:10..20**](https://github.com/search?q=stars%3A10..20+size%3A<1000&type=Repositories) 匹配具有 10 到 20 个星号、小于 1000 KB 的仓库。[**stars:>=500 fork:true language:php**](https://github.com/search?q=stars%3A>%3D500+fork%3Atrue+language%3Aphp&type=Repositories) 匹配具有至少 500 个星号，包括复刻的星号（以 PHP 编写）的仓库。 |
| `language:LANGUAGE` | [**rails language:javascript**](https://github.com/search?q=rails+language%3Ajavascript&type=Repositories) 匹配具有 "rails" 字样、以 JavaScript 编写的仓库 |
| Awesome + 关键字    | 通过该关键字搜索出来的都是比较好的一些资源，排名靠前的项目人气都非常高。 |

# DayX：dos命令

```bash
#切换盘符
>d:
#查看当前目录所有目录
>dir
#切换目录
>cd 
>cd /d d:\dir1\dir2 #跨盘符需加/d参数
>cd .. #返回上一级
#创建、删除目录
>md dirName 
>rd dirName
#创建、删除文件
>cd>filename.txt
>del filename.txt
#清理屏幕
>cls  (clear screen)
```



# Day02：第一个程序HelloWorld

## 第一个简单程序

1. 创建一个**HelloWorld.java**文件代码如下：

```java
public class HelloWorld{
	public static void main(String[] args){
		System.out.println("HelloWorld");
	}
}
```

2. cmd下执行**javac HelloWorld.java**编译生成**HelloWorld.class**

3. **java HelloWorld ** 注意不用加.class后缀名

## IDEA中Project和Module
当新建一个Empty Project的时候需要再添加一个Module才能开始写代码文件，之后需配置Project的环境如下图
<img src=".\assets.md\1614181299(1).jpg" alt="1614181299(1)" style="zoom: 67%;" />

## 注释

JavaDoc（文档注释）

```java
/**
 * @Description
 * @Author
 */
//  先打/**然后回车就可以了
```

## 标识符的命名

* 所有的标识符只能以字母（A-Z或a-z）或者美元符号$或者下划线_开始

* 首字母之后可以是字母（A-Z或a-z）、美元符号$、下划线_、数字的任意字符组合

* 大小写敏感

 **合法的命名： name  _name   $name   $1   _1   name1**

 **非法命名： 1name  -name  #name **

# Day03：32位计算机 & 64位计算机

- **32位计算机的CPU一次最多能处理32位数据，也就是4个字节。**

  4个字节能表示的数范围0～2<sup>32</sup>-1对比于1个字节表示的范围是0～2<sup>8</sup>-1***（不只4倍！！！就像int类型能表示的数比byte类型大得多***）

- **32位计算机的CPU有32位的寻址能力。**

  有个问题：**32位计算机最多支持4G内存？**
  
  原因：**内存的地址表示的是一个字节Byte而不是一个位bit**，所以当有32位的寻址能力的时候就能寻址4\*2<sup>10</sup>\*2<sup>10</sup>\*2<sup>10</sup>B也就是4\*2<sup>10</sup>\*2<sup>10</sup>KB或者说4\*2<sup>10</sup>MB或者说4GB的内存地址。

## 数据类型

### 基本类型

1. 整型 byte short int long （long类型的数字的表示  1000L）

   ```java
   int a1 = 10; //10进制数
   int a1 = 010; //8进制数
   int a1 = 0x10; //16进制数
   ```

2. 浮点型 float double （float类型的表示 5.2F）

   ```java
   float f1 = 23564122311212F;
   float f2 = f1 + 1;
   System.out.println(f1==f2); //true
   float f3 = 0.1F;
   double f4 = 1.0/10; // f4=0.1
   System.out.println(f3==f4); //false
   ```

   **一定要避免使用浮点型比较**

   **一定要避免使用浮点型比较**

   **一定要避免使用浮点型比较**

   浮点型比较请交给专业的类**BigDecimal**

3. 字符类型 char （'A'  '中'）

   ```java
   //unicode编码 2个字节（表示范围U0000到UFFFF 即 0~65536）
   char c1 = '\u0061'; //一个unicode字符的表示方法，由4位16进制数
   System.out.println(c1); // ‘a’
   ```

4. 布尔型 boolean

### 引用类型

1. 类

   String类

   ```java
   String s1 = new String("hello world");
   String s2 = new String("hello world");
   System.out.println(s1==s2); // false
   String s3 = "hello world";
   String s4 = "hello world";
   System.out.println(s3==s4); // true
   s3 += "abc";
   System.out.println(s3); // hello worldabc
   System.out.println(s4); // hello world
   ```

## 类型转换

类型转换的高低byte short char => int => long => float => double

1. 自动转换，低类型会自动转成高类型

2. 强制转换，高类型转成低类型需进行强制转换

3. 转换过程 造成的内存溢出

4. 运算过程 转成同一类型进行运算

# Day04：变量、作用域&常量

## 变量

type varName [=value] [{,varName2[=value]}];

数据类型  变量名 = 值; 可以用逗号隔开来声明多个同类型变量，但是不建议这么做！

变量的命名采用驼峰命名法，例如lastName。

## 作用域：

1. 静态变量（类变量），作用域是整个类！也就是说一个类的static变量被改变会影响该类所有的对象！

2. 实例变量，作用域是单个对象，变量值的变化不改变其他对象的成员变量。

3. 局部变量

```java
public class DataTypeTransform {
    static String name = "XiaoY"; // 类变量，从属于类
    int age = 1; //实例变量，从属于对象
    public static void main(String[] args) {
        int i = 10; //局部变量
        DataTypeTransform dtf = new DataTypeTransform();
        DataTypeTransform dtf2 = new DataTypeTransform();
        System.out.println(DataTypeTransform.name); //XiaoY
        DataTypeTransform.name = "Zolo";
        System.out.println(dtf.name); //Zolo
        dtf.name="Gao";
        System.out.println(dtf2.name); //Gao
        System.out.println(DataTypeTransform.name); //Gao
    }
    void sum (){
        System.out.println(age);
        age = 1;
        System.out.println(age);
    }
}  
```

## 常量
被final修饰符修饰的变量称为常量（Constant），值初始化之后便不会再改变。
常量一般使用大写字符，单词间用_隔开，例如ROOM_WITDH；
```java
final static double PI = 3.14;
```

## 命名规则

- 类：所有单词首字母大写驼峰原则 OldPeople、Man

- 变量：首字母小写驼峰原则 lastName

- 方法名：首字母小写驼峰原则 run()、runCool()

- 常量：大写 MAX_WIDTH

## 基本运算

### 位运算

- & 位与  a & b 仅当a=1，b=1时结果为1，其他情况结果为0
- | 位或   a | b 当a=1或者b=1时结果为1，其他情况结果为0
- ^ 异或  a ^ b 当a和b是同一个值则为0，其他情况为1
- ! 取反
- <<  >> 左移 右移运算 相当于 *2  /2

```java
/**
  a = 0011 1100
  b = 0001 1010
  a&b=0001 1000
  a|b=0011 1110
  a^b=0010 0110
  !b =1110 0101
  2<<3 得到16
*/
```

### 连接符 + 

如果 + 号左右两侧出现字符串，则会将操作数转成字符串类型

```java 
int a = 1;
int b = 2;
""+a+b // 12
a+b+"" // 3
```

# Day05：包机制 & JavaDoc

一般用公司域名建包，例如：com.hpr.oa =》com/hpr/oa

```java
package com.hpr.oa;  //打包
import com.hpr.erp;  //导入包
```

## JavaDoc

参数信息

- @author 作者名
- @version 版本号
- @since 指明需要最早使用的jdk版本
- @param 参数名
- @return  返回值情况
- @throws 异常抛出情况

```java
package com.hpr.erp;
/**
 * Author Maiziii
 */
public class TestJavaDoc {
    /**
     *
     * @param s1 第1个字符串
     * @param s2 第2个字符串
     * @return 拼接2个字符串
     * @throws Exception 可能的未知异常
     */
    public String concat(String s1,String s2) throws  Exception{
        return s1+s2;
    }
}
```

使用CMD输入命令行

```shell
>javadoc -encoding UTF-8 -charset UTF-8 TestJavaDoc.java
```

即可生成Java文档

![自己的Java文档](.\assets.md\生成JavaDoc.jpg)

## Java的输入类：Scanner 

**next()**和**nextLine()**的区别：next()遇到空格就停止，next()取不到空格字符。nextLine()是遇到回车符才停止，能取到空格字符。

```java
Scanner scanner = new Scanner(System.in);
while(scanner.hasNextInt()){
    System.out.println("输入的整数i：" + scanner.nextInt());
}
scanner.close(); //一定要记得关闭，切记！切记！切记！
//当一直输入整数时，程序不会停止会一直输出"输入的整数i：xxx"的提示信息，当输入非Int型时程序停止
```

有个疑问，到底是Scanner.hasNext()还是Scanner.next()阻塞程序使得用户可以输入信息？如下

```java
Scanner scanner = new Scanner(System.in);
if (scanner.hasNextInt()) {
    int i = scanner.nextInt();
    System.out.println("输入的整数i：" + i);
} else {
    System.out.println("不是整数");
}
if (scanner.hasNextFloat()) {
    float f = scanner.nextFloat();
    System.out.println("输入的小数f：" + f);
} else {
    System.out.println("不是小数");
}
scanner.close();
//当输入10时，程序输出信息 "输入的整数i：xxx" 然后让你再次输入，如果输入小数 程序输出 "输入的小数f：xxx"
//如果第一次输入10.2 程序会直接输出信息 
"不是整数"
"输入的小数f：10.2"
//如果输入的是 abc 程序输出
"不是整数"
"不是小数"
```

Scanner类的用户输入信息是存放于缓冲区的数据流，hasNext()的方法就是判断缓冲区是否有数据（hasNextInt判断缓冲区是否有数据并且是整数），而next()方法（nextInt()方法）是将输入信息从缓冲区取出，取出后缓冲区中就没有这个数据了。hasNext()和next()方法都会阻塞程序让用户进行输入，如果缓冲区为空则先让用户输入，如果缓冲区中有数据则不会先让用户输入而是直接对缓冲区进行判断或者进行取值。

所以上面这段代码，当你第一次输入了小数，因为判断不是Int类型所以不执行nextInt()方法，小数还保留在缓冲区中。而hasNextFloat()又去判断缓冲区中是否有小数，刚好是小数所以取出小数，并输出了 "输入的小数f：xxx"。

# Day06：Switch & IDEA反编译Class文件

## Switch结构

比较少用到好容易忘记怎么写switch结构判断

```java
switch(expression) {
    case value1:
        ...
        break;
    case value2:
        ...
        break;
    default:
        ...
}
```

**expression数据类型可以是byte、short、int、char；**<font color='red'>并且从jdk7开始支持字符串</font>;

**value必须为字符串常量或者字面量，不能是变量！**

## 反编译Class文件

方法1：

![Decompile](.\assets.md\IDEA反编译class文件.jpg)

方法2：

![Decompile2](.\assets.md\IDEA获取程序Class文件.jpg)

# Day07：while & do while & for

```java
do{
    ...
}while(expression)
//idea有个for循环的快捷键 100.for
for(;;;){
    
}
```

## 增强型for - foreach循环

```java
//增强型for主要用来循环数组、集合
//foreach没有下标，不适合对单个元素进行操作的场景
int[] arr = {1,2,3,4,5};  //注意是花括号！！！
for(int i : arr){ //注意是冒号，而不是in
    ...
}
```

java和javascript数组声明和循环for有区别

```javascript
var arr = [1,2,3,4,5]; 
for(var i in arr){ //这个方法主要不是用来遍历数组，而是用来获取对象key和value，
    
}
//或者
arr.forEach(e=>{...});
```

# Day08：方法重载 & 命令行传参 & 可变参数

## 方法重载

- 方法名称相同

- 参数个数或者类型或者顺序不同

**方法重载和方法的返回值、修饰符无关**，~~当一个重载方法被调用，JVM通过方法调用的实参类型、个数、顺序去寻找具体调用的那个方法。(这个说法待确认)~~

## 命令行传递参数

```java
package com.hpr.method;

public class CommandLine {
    public static void main(String[] args) {
        for (int i = 0; i < args.length; i++) {
            System.out.println(1+":"+args[i]);
        }
    }
}
```

**步骤**

1. javac编译.java文件生成.class文件，可以在src路径或.java当前路径执行javac进行编译。
2. java ClassName执行文件，注意要在src路径下执行，可以以包名.ClassName的方式执行，也可以路径的形式/.../.../ClassName的方式执行，如果后面跟着字符串便会被当成是传入main方法的参数。

<img src=".\assets.md\命令行传递参数.jpg" alt="命令行传递参数" style="zoom:80%;" />

## 可变参数

有时候定义的方法的参数个数可能是不固定的，可能是1个也有可能是2个也可能是5个，如果用重载方法去实现未免太繁琐。为了解决这个问题jdk5出现了一个新特性——可变参数。

在方法声明中，对指定参数类型后加省略号即表示该参数为可变参数，例如 **add(int... num)**

**一个方法只能定义一个可变参数，它必须是方法的最后一个参数，其他任何普通的参数要在它之前声明**

```java
package com.hpr.method;

public class VarArgs {
    public static void main(String[] args) {
        printMaxInt(1,3,9,45,7); //传入的可以是多个int整数
        printMaxInt(new int[]{2,6,4,0}); //也可以是int数组
    }
    public static void printMaxInt(int... i) { //形参i可看成是一个数组
        if (i.length == 0) {  //需要对i的长度进行判断，否则数据可能越界
            System.out.println("未传入参数!");
            return;
        }
        int max = i[0];
        for (int j = 0; j < i.length; j++) {
            if (i[j] > max) {
                max = i[j];
            }
        }
        System.out.println("最大值为:" + max);
    }
}
```

# Day 09-01：数组

- 数组是相同类型数据的有序集合。且只能有一种数据类型（基本数据类型或者引用类型）
- 数组的大小一旦确定就无法改变。
- 声明的数组变量是引用类型，<font color="red">但是数组本身是对象，Java中对象是存放于堆中</font>。

```java
int[] i; // 数组的声明
int i[]; // C语音对于数组的声明，Java中不提倡

Man[] mans = new Man[3]; //初始化数组的长度
mans[0] = new Man(); //动态初始化，必须先确定数组长度，否则会报错；数组长度可以用int变量指定

int[] i1 = new int[]{1,2,3};//静态初始化完整格式
int[] i2 = {1,2,3};//静态初始化省略格式，强烈建议不适用省略格式，静态初始化都用完整格式，可以使用匿名数组这一概念，如下
System.out.println(new int[] {1, 2, 4, 545, 11, 32, 13131, 4444}.length);
userFunction({1,2,3,4});//错误！作为实参传入的时候不能省略，应该写成userFunction(new int[]{1,2,3,4});
Girl[] girls = { new Girl(),new Girl() };//静态初始化省略格式
```

## 二维数组

```java
int[][] arr = new int[2][2];
arr[0] = new int[]{1};
for (int i = 0; i < arr.length; i++) {
    for (int j = 0; j < arr[i].length; j++) {
        System.out.print(arr[i][j]);
    }
    System.out.println("");
}
/**
 * 输出结果
 * 1
 * 00
 * 第一个元素数组长度为1，第二个元素数组自动补2个0
 */

int[][] arr2 = {{1,2},{1}};
for (int i = 0; i < arr2.length; i++) {
    for (int j = 0; j < arr2[i].length; j++) {
        System.out.print(arr2[i][j]);
    }
    System.out.println("");
}
/**
 * 输出结果
 * 12
 * 1
 * 第一个元素数组长度为2，第二个元素数组长度为1
 */

```

 ## java.util.Arrays 工具类

数组对象本身不提供什么方法，但Java提供了Arrays工具类，Arrays中有大量的静态方法可以对数组进行操作。常用的方法有

- `String Arrays.toString(varName[] a)` 返回数组a内容[, , ,]的字符串

- `void Arrays.sort(varName[] a)` 原数组进行升序排序

- `void Arrays.fill(varName[] a,varName val)`  对数组进行填充（赋值），例如Arrays(a,0); 数组a的所有元素将被赋值为0；`void Arrays.fill(varName[] a,int fromIndex,int toIndex,varName val)` 将数组a下标范围fromIndex到toIndex赋值为val

- `int binarySearch(int[] a, int key)` 用二分法查找指定元素b的下标；`binarySearch(Object[] a, int fromIndex, int toIndex, Object key)` 限定范围查找，<font color='red'>形参object[] a 是多态的应用？</font>

- `boolean equals(long[] a, long[] a2)` 比较a与a2是否元素都相同

## 稀疏数组

二维数组中有大量的重复值，可以用稀疏数组保存该数组

稀疏数组的处理方式是：

- 记录数组大小（行、列），有多少个不同值

- 把具体的不同值的元素和行列及值记录在一个小规模的数组中，从而缩小程序的规模

例如现有如下二维数组

```json
[
    [0,1,0,0,0],
	[0,0,2,0,0],
	[3,0,0,0,0]
]
```

用稀疏数组表示

```json
[
    [3,4,3], //第1行 第1、2元素记录数组的行数、列数，第3元素表示值为非0元素的个数
    [0,1,1], //第1个非0元素的坐标及值
    [1,2,2], //第2个非0元素的坐标及值
    [2,0,3]  //第3个非0元素的坐标及值
]
```

# Day09-02：类与对象

## 值传递 & 引用传递

值传递：调用方法传入的参数为基本类型变量，传入的变量的值不会被方法改变

引用传递：调用方法传入的参数为对象即引用类型变量（本质还是值传递），传入的对象的属性会被方法改变。

## 构造器

- 方法名必须与类名一样
- 方法无返回值
- 如果未显示定义类的构造器，则程序会自动添加一个无参构造器。
- 如果定义了有参数的构造器，就必须显示定义无参构造器，否则程序不会自动生成无参构造器

```java
class Person{
    String name;
    public Person(){

    }
    public Person(String name){
        this.name = name;
    }
}
```

**Idea快捷键： Alt + Insert 选择选择Constructor来快速生成有参/无参构造器**

## 堆&栈
![a](.\assets.md\Java运行的堆与栈1.png)

# Day10：封装 & 继承 

## 封装（Encapsulation）

> 在面向对象程式设计中，封装是指一种将抽象性函式接口的实现细节部分包装，隐藏起来的方法。
>
> 封装可以被认为是一个保护屏障，防止该类的代码和数据被外部类定义的代码随机访问，要访问该类的代码和数据，必须通过严格的接口控制。
>
> 封装最主要的功能在于我们只需修改自己的实现代码，而不用考虑那些调用我们代码的程序片段。
>
> 适当的封装可以让程式码更容易理解与维护，也加强了程式码的安全性。

封装的优点

1. 良好的封装能够减少耦合。
2. 类内部的结构可以自由修改。
3. 对成员变量进行更精确的控制。
4. 隐藏信息，实现细节。

程序应遵循**高内聚、低耦合**，应禁止直接访问对象的属性，而应该是通过某一接口来访问。***封装可以说就是将数据隐藏***

**一句话：私有属性 配合 getter/setter 方法。**优点：统一用setter方法可以很好地对属性进行赋值将值限定到一定范围内，比如年龄（限制1-100），统一用setter方法就可以在对年龄属性进行赋值时进行判断参数的范围。

**快捷键：Alt + Insert**

## 继承

> 继承就是子类继承父类的特征和行为，使得子类对象（实例）具有父类的实例域和方法，或子类从父类继承方法，使得子类具有父类相同的行为。

***继承的主要作用是对类进行扩充以及代码的重用！但是也提高了类之间的耦合性（继承的缺点，耦合度高就会造成代码之间的联系越紧密，代码独立性越差）。***

**快捷键：Ctrl + H**

~~private属性也能被子类继承，但是要用getter/setter方法进行访问~~

~~在进行继承的时候，子类会继承父类的所有结构（包括私有属性、构造方法、普通方法）
**显示继承**：所有非私有操作属于显示继承（可以直接调用）。
**隐式继承**：所有私有操作属于隐式继承（不可以直接调用，需要通过其它形式调用（get或者set））。
也有说法：子类不继承父类的私有属性、构造方法~~

**子类对象在进行实例化前首先调用父类构造方法，再调用子类构造方法实例化子类对象。实际在子类构造方法中，相当于隐含了一个语句super()，调用父类的无参构造。同时如果父类里没有提供无参构造，那么这个时候就必须使用super(参数)明确指明要调用的父类构造方法。**

> 需要注意的是 Java 不支持多继承，但支持多重继承。
>
> 可以用 implements 来伪实现多继承

final、String类不能被继承。

# Day11：方法的重写Override & 重载(Overload)

## 重写

> 重写是子类对父类的允许访问的方法的实现过程进行重新编写, 返回值和形参都不能改变。**即外壳不变，核心重写！**

规则：

- 参数列表与被重写方法的参数列表必须完全相同。
- 返回类型与被重写方法的返回类型可以不相同，但是必须是父类返回值的派生类（java5 及更早版本返回类型要一样，java7 及更高版本可以不同）。
- 访问权限不能比父类中被重写的方法的访问权限更低。例如：如果父类的一个方法被声明为 public，那么在子类中重写该方法就不能声明为 protected。
- 父类的成员方法只能被它的子类重写。
- 声明为 final 的方法不能被重写。
- 声明为 static 的方法不能被重写，但是能够被再次声明。
- 子类和父类在同一个包中，那么子类可以重写父类所有方法，除了声明为 private 和 final 的方法。
- 子类和父类不在同一个包中，那么子类只能够重写父类的声明为 public 和 protected 的非 final 方法。
- 重写的方法能够抛出任何非强制异常，无论被重写的方法是否抛出异常。但是，重写的方法不能抛出新的强制性异常，或者比被重写方法声明的更广泛的强制性异常，反之则可以。
- 构造方法不能被重写。
- 如果不能继承一个类，则不能重写该类的方法。

## 重载

> 重载(overload) 是在一个类里面，方法名字相同，而参数不同。返回类型可以相同也可以不同。
>
> 每个重载的方法（或者构造函数）都必须有一个独一无二的参数类型列表。
>
> 最常用的地方就是构造器的重载。

**重载规则:**

- 被重载的方法必须改变参数列表(参数个数或类型不一样)；
- 被重载的方法可以改变返回类型；
- 被重载的方法可以改变访问修饰符；
- 被重载的方法可以声明新的或更广的检查异常；
- 方法能够在同一个类中或者在一个子类中被重载。
- 无法以返回值类型作为重载函数的区分标准。

# 修饰符 ？

- public
- private
- protect
- default

# Day12 ：多态

> 多态是同一个行为具有不同表现形式或状态的能力。
>
> 多态就是同一个接口，使用不同的实例而执行不同操作。

### 多态的优点

- 消除类型之间的耦合关系
- 可替换性
- 使程序有良好的扩展性
- 接口性
- 灵活性
- 简化性

### 多态存在的3个必要条件

- 继承

- 重写

- 父类引用指向子类对象：

```java
Parent p = new Child();
p.cry();
//父类Parent与子类Child都必须包含此cry()方法，编译阶段程序能找到Parent中的cry()方法，程序编译通过，而运行时JVM实际执行的是Child类中的cry()方法，我们称之为动态编译。
/**
 * 如果Parent中没有cry()方法，而Child类中有cry()方法，程序则编译不通过，此时需要进行类型转换
 * 父类（高优先级） => 子类（低优先级）  
 * 和基本类型转换一样  低转高可以直接转。
 * 但高转低需进行强制类型转换，而且会有精度丢失
 * ((Child)p).cry(); //编译通过
```

## 多态的实现方式

- 重写：
- 接口
- 抽象类和抽象方法

# Day13：instanceof关键字

> instanceof 是 Java 的一个二元操作符，类似于 ==，>，< 等操作符。
>
> instanceof 是 Java 的保留关键字。它的作用是测试它左边的对象是否是它右边的类的实例，返回 boolean 的数据类型。

```java
Person p1 = new Person();
Person p2 = new Man();
Man m1 = new Man();
System.out.println(p1 instanceof Man);//false
System.out.println(p2 instanceof Man);//true  关键是看变量指向的对象类型
System.out.println(m1 instanceof Man);//true
```

# Day 14 ： static关键字

## 代码块

```java
public class Person {
    {
        //匿名代码块
        System.out.println("在实例化Person对象时执行");
    }

    static {
        //静态代码块
        //类加载时执行，只执行一次
        System.out.println("类加载时执行，只执行一次");
    }
}
```

# 抽象类

> 在面向对象的概念中，所有的对象都是通过类来描绘的，但是反过来，并不是所有的类都是用来描绘对象的，如果一个类中没有包含足够的信息来描绘一个具体的对象，这样的类就是抽象类。

> 抽象类除了不能实例化对象之外，类的其它功能依然存在，成员变量、成员方法和构造方法的访问方式和普通类一样。

> 由于抽象类不能实例化对象，所以抽象类必须被继承，才能被使用。也是因为这个原因，通常在设计阶段决定要不要设计抽象类。

## 抽象方法

> 如果你想设计这样一个类，该类包含一个特别的成员方法，该方法的具体实现由它的子类确定，那么你可以在父类中声明该方法为抽象方法。

> abstract 关键字同样可以用来声明抽象方法，抽象方法只包含一个方法名，而没有方法体。抽象方法没有定义，方法名后面直接跟一个分号，而不是花括号。

```java
public abstract void method();
```

声明抽象方法会造成以下两个结果：

- 如果一个类包含抽象方法，那么该类必须是抽象类。
- 任何子类必须重写父类的抽象方法，或者声明自身为抽象类。

## 抽象类总结规定

- 抽象类不能被实例化，如果被实例化，就会报错，编译无法通过。只有抽象类的非抽象子类可以创建对象。
- 抽象类中不一定包含抽象方法，但是有抽象方法的类必定是抽象类。
- 抽象类中的抽象方法只是声明，不包含方法体，就是不给出方法的具体实现也就是方法的具体功能。
- 构造方法，类方法（用 static 修饰的方法）不能声明为抽象方法。
- 抽象类的子类必须给出抽象类中的抽象方法的具体实现，除非该子类也是抽象类。

# 接口

接口（Interface），在JAVA编程语言中是一个抽象类型，是抽象方法的集合，接口通常以interface来声明。一个类通过继承接口的方式，从而来继承接口的抽象方法。

接口并不是类，编写接口的方式和类很相似，但是它们属于不同的概念。类描述对象的属性和方法。接口则包含类要实现的方法。

除非实现接口的类是抽象类，否则该类要定义接口中的所有方法。

接口无法被实例化，但是可以被实现。一个实现接口的类，必须实现接口内所描述的所有方法，否则就必须声明为抽象类。另外，在 Java 中，接口类型可用来声明一个变量，他们可以成为一个空指针，或是被绑定在一个以此接口实现的对象。

## 特性

- 接口不能用于实例化对象。
- 接口没有构造方法。
- 接口中所有的方法必须是抽象方法。接口中每一个方法也是隐式抽象的，接口中的方法会被隐式的指定为 public abstract（只能是 public abstract，其他修饰符都会报错）。
- 接口成员变量，只有 static 和 final 常量。接口中可以含有变量，但是接口中的变量会被隐式的指定为 public static final 变量（并且只能是 public，用 private 修饰会报编译错误）。
- 接口不是被类继承了，而是要被类实现。接口中的方法是不能在接口中实现的，只能由实现接口的类来实现接口中的方法。
- 接口支持多继承。

## 声明

```java
[可见度] interface 接口名称 [extends 其他的接口名] {
        // 声明变量  很少这么用
        // 抽象方法
}
//类使用implements关键字实现接口。
...implements 接口名称[, 其他接口名称, 其他接口名称..., ...] ...
//多继承
//在Java中，类的多继承是不合法，但接口允许多继承。
//在接口的多继承中extends关键字只需要使用一次，在其后跟着继承接口。 如下所示：
public interface Hockey extends Sports, Event
```

## 标记接口

最常用的继承接口是没有包含任何方法的接口。

标记接口是没有任何方法和属性的接口.它仅仅表明它的类属于一个特定的类型,供其他代码来测试允许做一些事情。

标记接口作用：简单形象的说就是给某个对象打个标（盖个戳），使对象拥有某个或某些特权。

例如：java.awt.event 包中的 MouseListener 接口继承的 java.util.EventListener 接口定义如下：

```java
package java.util; 
public interface EventListener {}
```

没有任何方法的接口被称为标记接口。标记接口主要用于以下两种目的：

- 建立一个公共的父接口：

  正如EventListener接口，这是由几十个其他接口扩展的Java API，你可以使用一个标记接口来建立一组接口的父接口。例如：当一个接口继承了EventListener接口，Java虚拟机(JVM)就知道该接口将要被用于一个事件的代理方案。

- 向一个类添加数据类型：

  这种情况是标记接口最初的目的，实现标记接口的类不需要定义任何接口方法(因为标记接口根本就没有方法)，但是该类通过多态性变成一个接口类型。

# 内部类

- 成员内部类
- 局部内部类
- 匿名内部类

```java
public class Outer {
    public void method(){
        //匿名内部类
        new UserService(){
           public void run(){                
           }
        }.run();        
        //局部内部类
        class Inner2{
        }
    }
    //成员内部类
    class Inner{
    }
}
interface UserService{
    void run();
}
```

# 异常

> 异常是程序中的一些错误，但并不是所有的错误都是异常，并且错误有时候是可以避免的。

所有的异常类是从 java.lang.Exception 类继承的子类。

Exception 类是 Throwable 类的子类。除了Exception类外，Throwable还有一个子类Error 。

Java 程序通常不捕获错误。错误一般发生在严重故障时，它们在Java程序处理的范畴之外。

Error 用来指示运行时环境发生的错误。

例如，JVM 内存溢出。一般地，程序不会从错误中恢复。

异常类有两个主要的子类：IOException 类和 RuntimeException 类。。

![](.\assets.md\java异常.jpg)

## 捕获异常

使用 try 和 catch 关键字可以捕获异常，IDEA快捷键 Ctrl + Alt + T

```java
try{
  // 程序代码
}catch(异常类型1 异常的变量名1){  //由小到大对异常进行捕获
  // 程序代码
}catch(异常类型2 异常的变量名2){  //由小到大对异常进行捕获
  // 程序代码
}finally{  
  // finally 关键字用来创建在 try 代码块后面执行的代码块。
  // 无论是否发生异常，finally 代码块中的代码总会被执行。在 finally 代码块中，可以运行清理类型等收尾善后性质的语句。
}
```

## throws/throw 关键字


# Day13：instanceof & 类型转换



# 数据库

## 三大范式

**范式（NF，NormalForm）是符合某一种级别的关系模式的集合**

### 第一范式（1NF）

**数据库表的每一列都是不可分割的原子数据项**。

![img](.\assets.md\第一范式.png)

在上面的表中，“家庭信息”和“学校信息”列均不满足原子性的要求，故不满足第一范式，调整如下：

![img](.\assets.md\第一范式改.png)

### 第二范式（2NF）

**第二范式需要确保数据库表中的每一列都和主键相关，而不能只与主键的某一部分相关（主要针对[联合主键](#复合主键&联合主键)而言）。**

举例说明：

![img](.\assets.md\第二范式.png)

在上图所示的情况中，同一个订单中可能包含不同的产品，因此主键必须是`订单号`和`产品号`联合组成，

但可以发现，`产品数量`、`产品折扣`、`产品价格`与`订单号`和`产品号`都相关，但是`订单金额`和`订单时间仅`与`订单号`相关，与`产品号`无关，

这样就不满足第二范式的要求，调整如下，需分成两个表：

 ![img](.\assets.md\第二范式1.png) ![img](.\assets.md\第二范式2.png)

### 第三范式（3NF）

**在2NF基础上，任何非主属性不依赖于其它非主属性（在2NF基础上消除传递依赖）**

**第三范式需要确保数据表中的每一列数据都和主键直接相关，而不能间接相关。**

举例说明：

![img](.\assets.md\第三范式1.png)

上表中，所有属性都完全依赖于`学号`，所以满足第二范式，但是`班主任性别`和`班主任年龄`直接依赖的是`班主任姓名`，而不是主键`学号`，所以需做如下调整：

![img](.\assets.md\第三范式2.png) ![img](.\assets.md\第三范式3.png)

**符合范式也就是规范，与效率有时候不可兼得，此时效率更重要**

## 复合主键&联合主键

### 复合主键

表的主键含有一个以上的字段组成 。 例如； 

```sql
create table test ( 
    name varchar(32), 
    id int, 
    value varchar(32), 
    primary key (id,name) 
) 
```

### 联合主键

由多个主键联合形成一个主键组合，体现在联合。如：主键A跟主键B组成联合主键。

**联合主键体现在多个表上，复合主键体现在一个表中的多个字段**

```sql
--学生表
create table student(
id mediumint  auto_increment comment '主键id',
name varchar(30) comment '姓名',
age smallint comment '年龄',
primary key(id,name)  --复合主键
)
engine = myisam,
charset = utf8,
comment = '学生'

--课程表
create table course(
id mediumint  auto_increment comment '主键id',
name varchar(30) comment '课程名称',
primary key(id)
)
engine = myisam,
charset = utf8,
comment = '课程'

--学生课程表
create table IF NOT EXISTS stu_cour(
id mediumint  auto_increment comment '主键id',
stu_id mediumint comment '学生表id',
cour_id mediumint comment '课程表id',
primary key(id)
)
engine = myisam,
charset = utf8,
comment = '学生课程表'

--此时stu_cour中id就表示联合主键，通过id可以获取学生和课程的一条记录
```

## 事务

访问并可能操作各种数据项的一个数据库操作单元。

### 四个特性：

#### 原子性（<font color="red">A</font>tomicity）

所有SQL作为原子工作单元执行，要么全部执行，要么全部不执行；如果事务崩溃，状态回到事务之前（事务回滚）。

#### 一致性（<font color="red">C</font>onsistency）

事务必须是使数据库从一个一致性状态变到另一个一致性状态。一致性与原子性是密切相关的。

#### 隔离性（<font color="red">I</font>solation）

如果有多个事务并发执行，每个事务作出的修改必须与其他事务隔离，也就是说如果2个事务 T1 和 T2 同时运行，事务 T1 和 T2 最终的结果是相同的，不管 T1和T2谁先结束。

#### 持久性（<font color="red">D</font>urability）

事务一旦提交，对数据的改变就将永久性保存。主机故障不应该对其有任何影响。

### 隔离等级

#### Read Uncommitted

是隔离级别最低的一种事务级别。在这种隔离级别下，一个事务会读到另一个事务更新后但未提交的数据，如果另一个事务回滚，那么当前事务读到的数据就是脏数据，这就是**脏读（Dirty Read）**。

#### Read Committed

一个事务可能会遇到不可重复读（Non Repeatable Read）的问题。**不可重复读**是指，在一个事务内，多次读同一数据，在这个事务还没有结束时，如果另一个事务恰好修改了这个数据，那么，在第一个事务中，两次读取的数据就可能不一致。

#### Repeatable Read

一个事务可能会遇到幻读（Phantom Read）的问题。

**幻读**是指，在一个事务中，第一次查询某条记录，发现没有，但是，当试图更新这条不存在的记录时，竟然能成功，并且，再次读取同一条记录，它就神奇地出现了。

#### Serializable

最严格的隔离级别，所有事务按照次序依次执行，因此，脏读、不可重复读、幻读都不会出现。

虽然Serializable隔离级别下的事务具有最高的安全性，但是，由于事务是串行执行，所以效率会大大下降，应用程序的性能会急剧降低。如果没有特别重要的情景，一般都不会使用Serializable隔离级别。

#### 默认隔离级别

如果没有指定隔离级别，数据库就会使用默认的隔离级别。在MySQL中，如果使用InnoDB，默认的隔离级别是Repeatable Read。

# MySQL的安装

### Windows

1. 下载mysql免安装版，下载地址`https://cdn.mysql.com//Downloads/MySQL-8.0/mysql-8.0.23-winx64.zip`（用迅雷下载比较快）

2. 设置环境变量，将mysql`\bin`文件目录加入Path变量中。

3. mysql根目录添加`my.ini`文件，内容如下：（注意不要新增这个`\Data`文件夹）

   ```ini
   [mysqld]
   port=3306
   basedir=D:\mysql\
   datadir=D:\mysql\Data
   #skip-grant-tables 为跳过密码验证 mysql8.0上有这一句mysql服务器无法启动，需要去掉
   ```

4. 使用管理员权限启动CMD，进入mysql\bin目录执行

   ```bash
   >mysqld -install #安装mysql服务  和 sc delete mysql 清空服务命令对应
   >mysqld --initialize-insecure --user=mysql #初始化mysql
   >net start mysql #启动mysql服务
   >mysql -uroot -p #连接登录mysql，注意是mysql不是mysqld
   #mysql8.0 password()函数不适用改用md5()
   >ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY '123456'; #修改root密码 
   >update mysql.user set authentication_string=md5('123456') where user='root' and host='hostlocal'; 
   >flush privileges; #刷新权限
   ```

## 安装SQLyog

[下载链接](\安装文件\SQLyog.zip)

名称：随意   注册码：8d8120df-a5c3-4989-8f47-5afc79c56e7c

## 创建数据库

```sql
CREATE DATABASE `school` CHARACTER SET utf8 COLLATE utf8_general_ci;
drop database [if exists] `school`;
--反引号``类似于SQL Server的[]用于区分关键字
```

![image-20210316114757403](.\assets.md\创建数据库.png)

## 创建表

![](.\assets.md\使用sqlyog创建mysql表.png)

```sql
--常用命令
show databases;  --显示所有数据库
use school; --切换数据库school
show tables; --查看数据库所有表
describe student; --查看表信息
create database XXX; --创建数据库
```

数据库xxx语言 CRUD    增加(Create)、检索(Retrieve)、更新(Update)和删除(Delete) 增删改查     

CRUD程序员（业务程序员）  CV程序员（面向百度） API程序员

DDL 数据库定义语言

DML操作语言

DQL查询

DCL 控制

## 字段类型

- 数值 

  **int  4个字节** 

  **decimal 字符串形式的浮点型（金融计算常见）**

- 字符串

  char 字符串固定大小 255

  **varchar 可变长字符串 65535**

  tinytext 微型文本 2^8-1

  **text  文本类型 2^16-1**

- 时间日期

  date YYYY-MM-DD

  time HH:mm:ss  

  **datetime YYYY-MM-DD HH:mm:ss**  

  **timestamp 时间戳**

- null

  <font color='red'>没有值，未知，不要用NULL进行运算，结果一定为NULL</font>

## 字段属性

- unsigned 无符号整数、声明该列不能为负数

- zerofill    0填充，不足的位使用0来填充 int(3)  5-->005

- 自增 可设置初始值、步长

- 非空 NOT NULL 字段必须有值

- 默认 设置默认的值

**阿里巴巴约定文档规定数据库表需有如下字段**

| id   | version | is_delete | gmt_create | gmt_update |
| ---- | ------- | --------- | ---------- | ---------- |
| 主键 | 乐观锁  | 逻辑删除  | 创建时间   | 更新时间   |

# JDBC

1. 导入jar

    新建lib目录（选择Directory），将`mysql-connector-java-8.0.23.jar`复制到lib目录下

    ![image-20210320155723312](.\assets.md\添加jar到lib.png)

2. 代码

```java
//加载驱动
Class.forName("com.mysql.cj.jdbc.Driver"); //mysql8.0和5.7不一样
//连接mysql，connection就是数据库，可以完成所有的数据库操作
Connection connection = DriverManager.getConnection("jdbc:mysql://localhost:3306/hpr?characterSet=uft8&useUnicode=true", "root", "123456");
//事务提交、回滚、取消自动提交
connection.commit(); //一旦执行了commit()，即使执行rollback()也回滚不了
connection.rollback();
connection.setAutoCommit(false);

//获取数据库执行类 Statement 和 prepareStatement
Statement statement = connection.createStatement();
//执行sql
ResultSet resultSet = statement.executeQuery("select username,nickname from t_user where is_alive=0;");
//获取结果集
while (resultSet.next()) {
    System.out.println("username:" + resultSet.getObject("username")+"；nickname:" + resultSet.getObject("nickname"));
}
//关闭
resultSet.close();
statement.close();
connection.close();
```

Statement执行sql会有SQL注入的情况，正常我们使用PreparedStatement

```java
Connection connection = null;
PreparedStatement st = null;
ResultSet rs = null;
try {
    connection = JdbcUtil.getConnection();
    st = connection.prepareStatement("update t_user set phonenumber=? where id=?");
    st.setString(1,"xxxxxxxx");
    st.setInt(2,0);
    st.executeUpdate();  
} catch (Exception e) {
    e.printStackTrace();
}finally {
    JdbcUtil.release(connection,st,rs);
}
```

## 自己封装的 JdbcUtil.java工具类

在`\src`目录下创建`db.properties`

```properties
#值不要用""引号包裹
driver=com.mysql.cj.jdbc.Driver
url=jdbc:mysql://localhost:3306/hpr?useUnicode=true&characterSet=utf8
username=root
password=123456
```

```java
package com.hpr.util;

import java.io.InputStream;
import java.sql.*;
import java.util.Properties;

public class JdbcUtil {
    private static String url = "";
    private static String username = "";
    private static String password = "";

    //类加载时被调用，只执行一次
    static {
        try {
            InputStream is = JdbcUtil.class.getClassLoader().getResourceAsStream("db.properties");
            Properties properties = new Properties();
            properties.load(is);
            driver = properties.getProperty("driver");
            url = properties.getProperty("url");
            username = properties.getProperty("username");
            password = properties.getProperty("password");
            Class.forName(driver);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    public static Connection getConnection() throws SQLException {
        return DriverManager.getConnection(url, username, password);
    }

    public static void release(Connection con, Statement st, ResultSet rs){
        if(con!=null){
            try {
                con.close();
            } catch (SQLException e) {
                e.printStackTrace();
            }
        }
        if(st!=null){
            try {
                st.close();
            } catch (SQLException e) {
                e.printStackTrace();
            }
        }
        if(rs!=null){
            try {
                rs.close();
            } catch (SQLException e) {
                e.printStackTrace();
            }
        }
    }
}
```

## 数据库连接池 

数据库连接 --- 执行完毕 --- 释放 十分浪费资源

**池化技术：预先准备一些资源，过来就连接预先准备好的连接。**

编写连接池：实现一个接口 DataSource

> 开源 DBCP、C3P0、Druid（阿里巴巴）

> DBCP

1. 导入dbcp的jar包 commons-dbcp2-2.8.0.jar  核心包 commons-pool2-2.9.0.jar 辅助包 commons-logging-1.2.jar

2. 创建配置文件，设置连接池参数(初始连接数，最大连接数，最大等待时间)

3. 创建连接池对象DataSource对象
4. DataSource获取连接对象(getConnection() 方法)

```java
package com.hpr.util;

import org.apache.commons.dbcp2.BasicDataSource;
import org.apache.commons.dbcp2.BasicDataSourceFactory;
import javax.sql.DataSource;
import java.io.InputStream;
import java.sql.*;
import java.util.Properties;

public class JdbcUtil_Dbcp {
    private static DataSource dataSource = null;
    static {
        try {
            InputStream is = JdbcUtil_Dbcp.class.getClassLoader().getResourceAsStream("dbcp.properties");
            Properties properties = new Properties();
            properties.load(is);
            dataSource = BasicDataSourceFactory.createDataSource(properties);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
    public static Connection getConnection() throws SQLException {
        return dataSource.getConnection();
    }
    public static void release(Connection con, Statement st, ResultSet rs) {
        if (con != null) {
            try {
                con.close();
            } catch (SQLException e) {
                e.printStackTrace();
            }
        }
        if (st != null) {
            try {
                st.close();
            } catch (SQLException e) {
                e.printStackTrace();
            }
        }
        if (rs != null) {
            try {
                rs.close();
            } catch (SQLException e) {
                e.printStackTrace();
            }
        }
    }
}

```







## IDEA连接MySQL

![image-20210321111346150](.\assets.md\IDEA连接mysql.png)

![image-20210321112430553](.\assets.md\mysql的serverTimezone参数.png)

# HTTP协议

超文本传输协议，一种简单的请求响应协议，通常运行于TCP之上。

文本指的是：html、字符串等等，超文本：视频、音频、图片、定位、地图

http/1.0  客户端与web服务器建立连接后只能获取一个web资源，然后断开连接

http/1.1  客户端与web服务器建立连接后可以获取多个web资源

## HTTP请求头

```xml
Accept :告诉浏览器，所支持的数据类型
Accept-Encoding  支持的编码格式 UTF8 GBK ISO8859-1
Accept-Language 语言环境
Cache-Control 缓存控制
Connection 告诉浏览器 请求完成是否保存连接
```

## HTTP请求体

当你在浏览器的地址栏输入地址并回车的一瞬间到 页面能够展示回来，经历了什么？

# Maven

**约定大于配置**

环境变量配置M2_HOME（maven的bin目录）和MAVEN_HOME变量，path变量加%MAVEN_HOME%\bin

Maven的JavaWeb应用

![image-20210326110755926](.\assets.md\Maven文件夹类型.png)

# Servlet

## Servlet简介

- sun公司推出的用于实现动态web的技术

- Sun在这些API中提供一个接口：Servlet，开发一个Servlet程序，只要2个步骤

  - 编写一个类，实现Servlet接口

  - 将开发好的java类部署到web服务器中


把实现了Servlet接口的java程序称为Servlet

![Servlet 架构](.\assets.md\servlet-arch.jpg)

## 第一个Servlet程序HelloServlet

1. 构建一个普通的Maven项目，删掉里面的src目录，以后我们的学习就在这个项目里面建立Module；这个空的工程就是Maven主工程

2. 父项目不使用模板只创建一个空的maven项目，子项目使用maven-archivetype-webapp模板创建，子项目会继承父项目的pom设置

   父项目的pom.xml

   ```xml
   <modules>
       <module>servlet-01</module>
   </modules>
   ```

   子项目中

   ```xml
   <parent>
       <groupId>com.hpr</groupId>
       <artifactId>javaweb-01-servlet</artifactId>
       <version>1.0-SNAPSHOT</version>
   </parent>
   ```
   修改web.xml文件

    ```xml
    <?xml version="1.0" encoding="UTF-8"?>
    <web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
             xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
             xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee
              http://xmlns.jcp.org/xml/ns/javaee/web-app_4_0.xsd"
             version="4.0">

        <welcome-file-list>
            <welcome-file>index.html</welcome-file>
            <welcome-file>index.htm</welcome-file>
            <welcome-file>index.jsp</welcome-file>
        </welcome-file-list>
    </web-app>
    ```


3. 编写一个普通类，实现Servlet接口，这里我们继承HttpServlet类

```java
public class HelloServlet extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        PrintWriter writer = resp.getWriter();
        writer.println("wo shi nibaba");
    }

    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        doGet(req,resp);
    }
}
```

4. 设置Servlet的映射

   为什么需要映射？因为我们写的是Java程序，浏览器需要通过web服务器才能访问。

   我们需要在web服务器中注册Servlet，并且配置一个浏览器能够访问的路径。

   在web.xml中配置如下

```xml
  <servlet>
    <servlet-name>hello</servlet-name>
    <servlet-class>com.hpr.servlet.HelloServlet</servlet-class>
  </servlet>
  <servlet-mapping>
    <servlet-name>hello</servlet-name>
    <url-pattern>/hello</url-pattern>
  </servlet-mapping>
```

## Servlet 三种方式

**1、实现 Servlet 接口**

因为是实现 Servlet 接口，所以我们需要实现接口里的方法。下面我们也说明了 Servlet 的执行过程，也就是 Servlet 的生命周期。

```java
//Servlet的生命周期:从Servlet被创建到Servlet被销毁的过程
//一次创建，到处服务
//一个Servlet只会有一个对象，服务所有的请求
/*
 * 1.实例化（使用构造方法创建对象）
 * 2.初始化  执行init方法
 * 3.服务     执行service方法
 * 4.销毁    执行destroy方法
 */
public class ServletDemo1 implements Servlet {

    //public ServletDemo1(){}

     //生命周期方法:当Servlet第一次被创建对象时执行该方法,该方法在整个生命周期中只执行一次
    public void init(ServletConfig arg0) throws ServletException {
                System.out.println("=======init=========");
        }

    //生命周期方法:对客户端响应的方法,该方法会被执行多次，每次请求该servlet都会执行该方法
    public void service(ServletRequest arg0, ServletResponse arg1)
            throws ServletException, IOException {
        System.out.println("hehe");

    }

    //生命周期方法:当Servlet被销毁时执行该方法
    public void destroy() {
        System.out.println("******destroy**********");
    }
//当停止tomcat时也就销毁的servlet。
    public ServletConfig getServletConfig() {
        return null;
    }

    public String getServletInfo() {
        return null;
    }
}
```

**2、Servlet有两个默认的实现类：HttpServlet、GenericServlet**

它实现了 Servlet 接口除了 service 的方法，不过这种方法我们极少用。

```java
public class ServletDemo2 extends GenericServlet {

    @Override
    public void service(ServletRequest arg0, ServletResponse arg1)
            throws ServletException, IOException {
        System.out.println("heihei");

    }
}
```

GenericServlet 是一个抽象类，实现了 Servlet 接口，并且对其中的 init() 和 destroy() 和 service() 提供了默认实现。在 GenericServlet 中，主要完成了以下任务：

-  将 init() 中的 ServletConfig 赋给一个类级变量，可以由 getServletConfig 获得；
-  为 Servlet 所有方法提供默认实现；
-  可以直接调用 ServletConfig 中的方法；

基本的结构如下：

```java
abstract class GenericServlet implements Servlet,ServletConfig{
 
   //GenericServlet通过将ServletConfig赋给类级变量
   private trServletConfig servletConfig;
 
   public void init(ServletConfig servletConfig) throws ServletException {

      this.servletConfig=servletConfig;

      /*自定义init()的原因是：如果子类要初始化必须覆盖父类的init() 而使它无效 这样
       this.servletConfig=servletConfig不起作用 这样就会导致空指针异常 这样如果子类要初始化，
       可以直接覆盖不带参数的init()方法 */
      this.init();
   }
   
   //自定义的init()方法，可以由子类覆盖  
   //init()不是生命周期方法
   public void init(){
  
   }
 
   //实现service()空方法，并且声明为抽象方法，强制子类必须实现service()方法 
   public abstract void service(ServletRequest request,ServletResponse response) 
     throws ServletException,java.io.IOException{
   }
 
   //实现空的destroy方法
   public void destroy(){ }
}
```

以上就是 GenericServlet 的大致实现思想，可以看到如果继承这个类的话，我们必须重写 service() 方法来对处理请求。

**继承 HttpServlet 方法（也是我们经常用的方法）**

```java
public class ServletDemo3 extends HttpServlet {

    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp)
            throws ServletException, IOException {
        System.out.println("haha");
    }

    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp)
            throws ServletException, IOException {
        System.out.println("ee");
        doGet(req,resp);
    }

}
```

HttpServlet 也是一个抽象类，它进一步继承并封装了 GenericServlet，使得使用更加简单方便，由于是扩展了 Http 的内容，所以还需要使用 HttpServletRequest 和 HttpServletResponse，这两个类分别是 ServletRequest 和 ServletResponse 的子类。代码如下：

```java
abstract class HttpServlet extends GenericServlet{
 
   //HttpServlet中的service()
   protected void service(HttpServletRequest httpServletRequest,
                       HttpServletResponse httpServletResponse){
        //该方法通过httpServletRequest.getMethod()判断请求类型调用doGet() doPost()
   }
 
   //必须实现父类的service()方法
   public void service(ServletRequest servletRequest,ServletResponse servletResponse){
      HttpServletRequest request;
      HttpServletResponse response;
      try{
         request=(HttpServletRequest)servletRequest;
         response=(HttpServletResponse)servletResponse;
      }catch(ClassCastException){
         throw new ServletException("non-http request or response");
      }
      //调用service()方法
      this.service(request,response);
   }
}
```

我们可以看到，HttpServlet 中对原始的 Servlet 中的方法都进行了默认的操作，不需要显式的销毁初始化以及 service()，在 HttpServlet 中，自定义了一个新的 service() 方法，其中通过 getMethod() 方法判断请求的类型，从而调用 doGet() 或者 doPost() 处理 get,post 请求，使用者只需要继承 HttpServlet，然后重写 doPost() 或者 doGet() 方法处理请求即可。

我们一般都使用继承 HttpServlet 的方式来定义一个 servlet。

## ServletContext

web容器启动的时候，会为每个web容器创建一个对应的ServletContext对象，它代表了当前的web应用：

- 共享数据

  ```java
  public class HelloServlet extends HttpServlet {
      @Override
      protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
          ServletContext context = this.getServletContext();
          context.setAttribute("username","曾思南");
          PrintWriter writer = resp.getWriter();
          writer.println("wo shi ni baba");
      }
  
      @Override
      protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
         doGet(req,resp);
      }
  }
  ```

  ```java
  public class ReadServlet extends HttpServlet {
      protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
          ServletContext context = this.getServletContext();
          Object attribute = context.getAttribute("username");
          //声明resp的content-type和编码，否则中文显示为问号 ???
          resp.setHeader("content-type","text/html;charset=UTF-8");
          // resp.setContentType("text/html");
          // resp.setCharacterEncoding("UTF-8");
          PrintWriter writer = resp.getWriter();
          writer.println("username:"+attribute);
      }
  
      @Override
      protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
          doGet(req,resp);
      }
  }
  ```

- 获取初始化参数

- 请求转发（RequestDispath）

  ```java
  public class ReadServlet extends HttpServlet {
      protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
          ServletContext context = this.getServletContext();
          context.getRequestDispatch("/hello").forward(req,resp);
      }
  
      @Override
      protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
          doGet(req,resp);
      }
  }
  ```

  转发（不会改变路径），http请求响应码为200而不是3xx

  ![image-20210329232500904](.\assets.md\请求转发.png)

  重定向

  ![image-20210329232554240](.\assets.md\请求重定向.png)

- 读取资源文件

  Properties

![image-20210404150321526](.\assets.md\servletcontext-properties.png)

如果properties放到了java目录下，资源导出会有问题，需要在该module的pom.xml配置build节点

```xml
<build>
    <!-- 配置build resources节点防止资源导出失败的问题 -->
    <resources>
        <resource>
            <directory>src/main/java</directory>
            <includes>
                <include>**/*.properties</include>
                <include>**/*.xml</include>
            </includes>
            <filtering>true</filtering>
        </resource>
    </resources>
</build>
```

![image-20210404151205176](.\assets.md\servlet-properties-02.png)

**关于classes  classpath路径，我们习惯称target/module/classes/目录为class path目录**


## 下载文件

1. 获取浏览器要下载文件的路径
2. 获取文件名称
3. 取得相应的OutputSteam输出流
4. 将FileInputStream输出到buffer缓冲区，
5. 设置响应头（content-disposition）

```java
public class FileDownLoadServlet extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        ServletContext servletContext = this.getServletContext();
        //1、获取要下载的文件路径以及文件名称
        //这个demo的文件放置于/resources目录下。将生成/target/项目名/WEB-INF/classes/文件名
        String filePath = "D:\\IntelliJ IDEA 2019.2.4\\IdeaSpace\\javaweb-01-servlet\\servlet-01\\src\\main\\resources\\QQ截图20210413145401.png";
        //也可以用getResourcePaths()方法获取classpath来获取资源
        //String filePath = servletContext.getResourcePaths("/WEB-INF/classes/QQ截图20210413145401.png");
        String fileName = filePath.substring(filePath.lastIndexOf("/")+1);
        //中文文件名URLEncoder.encode编码，否则有可能乱码
        resp.setHeader("Content-Disposition","attachment;filename="+ URLEncoder.encode(fileName,"UTF-8"));
        //4、获取下载文件的输入流
        FileInputStream in = new FileInputStream(filePath);
        //5、创建缓冲区
        int len = 0;
        byte[] buffer = new byte[1024];
        //6、获取OutputStream对象
        ServletOutputStream out = resp.getOutputStream();
        //7、将FileOutputStream流写入到buffer缓冲区，使用OutputStream将缓冲区中的数据输出到客户端
        while((len=in.read(buffer))>-1){
            out.write(buffer);
        }
        out.close();
        in.close();
    }

    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        doGet(req, resp);
    }
}
```

## 验证码的实现

```java
protected void doGet(HttpServletRequest req, HttpServletResponse resp) 
    										throws ServletException, IOException {
    ServletContext servletContext = this.getServletContext();
    //设置浏览器每3s刷新一次
    resp.setHeader("refresh","3");
    BufferedImage bufferedImage = 
        				new BufferedImage(200,30, BufferedImage.TYPE_INT_RGB);
    Graphics g = bufferedImage.getGraphics();
    g.setColor(Color.WHITE);
    g.fillRect(0,0,200,30);
    g.setColor(Color.BLACK);
    g.setFont(new Font(null,Font.BOLD,28));
    g.drawString(randNum(),0,29);
    //g.dispose();
    //告诉浏览器这个请求用图片的方式打开
    resp.setContentType("image/png");
    //网站有缓存，不让浏览器缓存
    resp.setDateHeader("expires",-1);
    resp.setHeader("Cache-Control","no-cache");
    resp.setHeader("Pragma","no-cache");
    //将图片写给浏览器
    ImageIO.write(bufferedImage,"png",resp.getOutputStream());
}
public static String randNum(){
    String s = "000"+ new Random().nextInt(9999);
    return s.substring(s.length()-4);
}
```



# HttpServletRequest

**常见应用：**

1. **获取请求参数**
2. **请求转发**

模拟一个登录页面以及登录后的跳转

1. 登录页面login.jsp，action中的`${pageContext.request.contextPath}`表示项目路径

```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<body>
<form action="${pageContext.request.contextPath}/login" method="post">
    用户名：<input type="text" name="username"/> <br />
    密码：<input type="password" name="password" /> <br />
    爱好：<input type="checkbox" name="hobbys" value="电影">电影
    	 <input type="checkbox" name="hobbys" value="音乐">音乐<br />
    <input type="submit" />
</form>
</body>
</html>
```

2. loginServlet.java

```java
public class LoginServlet extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        req.setCharacterEncoding("UTF-8");
        resp.setContentType("text/html;charset=utf-8");
        String username = req.getParameter("username") ;
        String password = req.getParameter("password");
        String[] hobbys = req.getParameterValues("hobbys");

        System.out.println("===========");
        System.out.println("username:"+username+"==password:"+password
                           +"==hobbys:"+ Arrays.toString(hobbys));
        System.out.println("===========");        
	    //请求转发 参数 /main.jsp前的 / 号代表当前web应用
        req.getRequestDispatcher("/main.jsp").forward(req,resp);
        /**
        也可以重定向 这边main.jsp前的 / 号并不是表示 当前web应用
        resp.sendRedirect(req.getContextPath()+ "/main.jsp");
        **/
    }

    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        doGet(req, resp);
    }
}
```

req.getParameter的[乱码问题](##req.getParameter(key)乱码的问题)

3. web.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee
          http://xmlns.jcp.org/xml/ns/javaee/web-app_4_0.xsd"
         version="4.0">

    <servlet>
        <servlet-name>login</servlet-name>
        <servlet-class>com.hpr.servlet.LoginServlet</servlet-class>
    </servlet>
    <servlet-mapping>
        <servlet-name>login</servlet-name>
        <url-pattern>/login</url-pattern>
    </servlet-mapping>

    <welcome-file-list>
      <welcome-file>index.html</welcome-file>
      <welcome-file>index.htm</welcome-file>
      <welcome-file>index.jsp</welcome-file>
    </welcome-file-list>


</web-app>
```

**注意：Maven创建的web默认是2.3的web.xml 当有jsp**

```jsp
<form action="${pageContext.request.contextPath}/login" method="post">
    <input type="text" name="username">
    <input type="submit">
</form>
```

提交的时候发生如下错误

![错误](.\assets.md\webxml25一下版本的action问题.png)

**web.xml需要2.5以上版本，模板如下**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee
          http://xmlns.jcp.org/xml/ns/javaee/web-app_4_0.xsd"
         version="4.0">
    
</web-app>
```

# Cookie

服务器响应给客户端cookie

```java
protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        req.setCharacterEncoding("utf-8");
        resp.setContentType("text/html;charset=utf-8");
        Cookie[] cookies = req.getCookies();
        PrintWriter out = resp.getWriter();
        if (cookies != null) {
            for (Cookie cookie : cookies) {
                if (cookie.getName().equals("lastlogintime")) {
                    String value = cookie.getValue();
                    out.write("您上一次登录时间：" 
                              + new Date(Long.parseLong(value)).toLocaleString());
                }
            }
        } else {
            System.out.println("第一次登录系统");
            out.write("第一次登录系统");
        }
        Cookie cookie = new Cookie("lastlogintime", System.currentTimeMillis() + "");
        //cookie.setMaxAge(86400);
        resp.addCookie(cookie);
    }
```

- 一个cookie只能保持一个信息；
- 一个web站点可以给浏览器发送多个cookie，最多20个cookie
- cookie大小限制4kb

## 删除cookie

- 不设置有效期，关闭浏览器cookie失效
- 设置有效期为0  `cookie.setMaxAge(0)`

# 编码&解码

```java
URLEncoder.encode("我是你爸爸","utf-8");
URLDecoder.decode(cookie.getValue(),"utf-8");
```

# Session

保存在服务端的信息，浏览器第一次访问系统，系统产生SessionId和其他Session的数据，然后将SessionId发给浏览器，之后浏览器发起请求时携带该SessionId服务器就能识别浏览器。

常见应用：

- 登录信息
- 购物车
- 系统应用常用的数据可以存在Session中

```java
HttpSession session = req.getSession();
Enumeration<String> attributeNames = session.getAttributeNames();//获取所有session key
session.setAttribute("name","value"); //设置session
session.removeAttribute("name"); //删除某session
session.invalidate(); //使session失效  类似登录中的注销
```

web.xml设置session失效时间

```xml
<session-config>
    <!-- 单位：分钟 -->
    <session-timeout>15</session-timeout>
</session-config>
```

# JSP 

Java Server Pages :Java服务器端页面，和Servlet一样，用于动态Web技术。

最大的特点：

- 写JSP就像是在写HTML
- 区别：
  - HTML只给用户提供静态数据
  - JSP可以嵌入JAVA代码，为用户提供动态数据

**浏览器访问服务器页面，本质上都是在访问Servlet**

JSP最终也会被转化成Servlet

Tomcat中有个work目录，IDEA的Tomcat work目录如下，可以找到jsp的转化之后的.java和.class文件，用户访问一个jsp页面其实最终服务器返回的时候由该jsp转化而成的Servlet的.calss文件

![image-20210429154020483](.\assets.md\IDEA中Tomcat的work路径.png)

![image-20210429154114459](.\assets.md\JSP转化成jsp点java文件.png)

![image-20210429154448895](.\assets.md\Jsp继承HttpJspBase继承HttpServlet.png)

JSP内置的一些对象

```java
final javax.servlet.jsp.PageContext pageContext; //页面上下文
javax.servlet.http.HttpSession session = null;  //Session
final javax.servlet.ServletContext application; //applicationContext作用域很高
final javax.servlet.ServletConfig config;
javax.servlet.jsp.JspWriter out = null;
final java.lang.Object page = this;
javax.servlet.jsp.PageContext _jspx_page_context = null;
```

**JSP本质是一个Servlet，JSP继承HttpJspBase类，HttpJspBase类继承HttpServlet**

```java
//HttpJspBase类
//初始化
public final void _jspInit(ServletConfig config) throws ServletException {
}
//销毁
public final void _jspDestroy() {
}
//JSP Service
public final void _jspService(HttpServletRequest req, HttpServletResponse resp){
}
```

（项目导入Tomcat的lib包，IDEA-Project Structure-Libraries-点击+号 - 选择Tomcat的lib文件 - 应用）

**Jsp的基础语法**

- jsp表达式

```jsp
<%= 变量或者表达式 %>

<!-- EL表达式 -->
${pageContext.request.contextPath}
```

- jsp脚本片段

```jsp
声明的变量和方法将被提高到类中而不是_jspService()方法<% for (int i = 0; i < 5; i++) {
    System.out.println(i);
}%>
<!-- -->
<% for (int i = 0; i < 5; i++) {%>
    <h1>序号：<%=i%></h1>
<%}%>
<!-- 结合El表达式 -->
<% for (int i = 0; i < 5; i++) {%>
    <h1>序号：${i}</h1>
<%}%>
```

- jsp声明

  jsp声明会被编译到jsp生成的类中，上面的2个方式会被生成到_jspService()方法中

```jsp
<%!
static {
    System.out.println("static");
}
String _ss = "sss";
public void Sum(){
    System.out.println("中");
}
%>
```

![image-20210430165119074](.\assets.md\JSP声明.png)

- 注释

```jsp
<%--  这个是jsp的注释，不会显示在客户端页面上  --%>
<!--  这个是html的注释，会显示在客户端页面上  --->
```

**JSP指令**

```jsp
<%@ page args.. %>
<!--例如-->
<%@ page import="java.util.*" %> <!--导入包-->
<%@ page errorPage="error.jsp" %> <!--错误页面-->

<!--include-->
<%@ include file="header.jsp" %> 
<!--
类似include的jsp标签，
区别：
	上面的include其实是一个页面，代码会有冲突；
	下面的jsp:include是多个页面，代码不会有冲突
-->
<jsp:include page="header.jsp" />
```

**web.xml设置错误页面**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee
          http://xmlns.jcp.org/xml/ns/javaee/web-app_4_0.xsd"
         version="4.0">
    ...
    <error-page>
        <error-code>404</error-code>
        <location>/error/404.jsp</location>
    </error-page>
</web-app>
```





