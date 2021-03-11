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
<img src="\assets.md\1614181299(1).jpg" alt="1614181299(1)" style="zoom: 67%;" />

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

![自己的Java文档](\assets.md\生成JavaDoc.jpg)

## Java的输入类：Scanner 

**next()**和**nextLine()**的区别：next()遇到空格就停止，next()取到空格字符。nextLine()是遇到回车符才停止，能取到空格字符。

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
"输入的小数f：xxx"
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

![Decompile](\assets.md\IDEA反编译class文件.jpg)

方法2：

![Decompile2](\assets.md\IDEA获取程序Class文件.jpg)

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

<img src="\assets.md\命令行传递参数.jpg" alt="命令行传递参数" style="zoom:80%;" />

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
int[] i1 = new int[]{1,2,3};//静态初始化
int[] i2 = {1,2,3};//静态初始化省略格式
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
![a](\assets.md\Java运行的堆与栈1.png)

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

**一句话：私有属性 配合 getter/setter 方法。**优点：统一用set方法可以很好地对属性进行赋值将值限定到一定范围内，比如年龄（限制1-100），统一用set方法就可以在对年龄属性进行赋值时进行判断参数的范围。

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

> 重载(overloading) 是在一个类里面，方法名字相同，而参数不同。返回类型可以相同也可以不同。
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