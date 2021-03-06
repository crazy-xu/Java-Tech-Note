## 二、Java基本程序设计结构
### 1、数据类型
    Java是一种强数据类型，必须为每一个变量声明一种类型。在Java中一共有8种基本数据类型（primitive type），其中4种整型、2种浮点类型、1种用于表示Unicode
    编码的字符单元的字符类型char和1种boolean类型。
    
#### 1.1、整型
    整型用于表示没有小数的数值，允许是负数。一共有4种，如下：
    
类型 | 存储需求 | 取值范围
---- | ------ | --------------
int | 4字节 | -2147483648 ~ 2147483647
short | 2字节 | -32768 ~ 32767
long | 8字节 | 好长好长
byte | 1字节 | -128 ~ 127
    在通常情况下int最常用（10位数），比较长的使用long类型，比如时间戳。byte和short类型底层文件处理或者需要控制占用存储空间量的大数组。
    
#### 1.2、浮点类型
    浮点类型用于表示有小数部分的数值。一共有2种，如下：
    
类型 | 存储需求 | 取值范围
---- | ------ | --------------
float | 4字节 | 大约 ± 3.402 823 47 E + 38 F ( 有效位数为 6 ~ 7 位 ）
double | 8字节 | 大约 ± 1.797 693 134 862 315 70 E + 308 （有效位数为 15 位）

    double表示的数值精度是float类型的两倍（一般称之为双精度数值）。绝大部分程序都采用double类型。
    float类型的数值有一个后缀F/f，没有后缀F的浮点数值，会被默认为double类型。
    浮点数值不适用于无法接受舍入误差的金融计算中。例如：2.0-1.1 不等于0.9，而是等于0.8999999999，出现这种舍入误差的原因是浮点数值采用二进制系统表示，
    而二进制系统中无法精确表示分数1/10，跟1/3类似。如果程序不允许有误差，需要配合BigDecimal类使用。
    
#### 1.3、char类型
    
    char类型原本是用于表示单个字符，现在有所改变，有些Unicode字符可以用一个char值表示，有的Unicode则需要两个char值表示。
    char类型的值可以表示为十六进制值，其范围从\u0000到\uffff。例如：\u2122表示注册符号（™），\u03C0表示希腊字母π。
    
转义序列 | 名称 | Unicode值
------ | ------- | --------
\b    | 退格 | \u0008
\t    | 制表 | \u0009
\n    | 换行 | \u000a
\r    | 回车 | \u000d
\"    | 双引号 | \u0022
\'    | 单引号 | \u0027
\\    | 反斜杠 | \u005c
    
#### 1.4、Unicode和char类型

    在Unicode出现之前，有许多不同的编码标准：
    美国的ASCII ((American Standard Code for Information Interchange): 美国信息交换标准代码）。
    西欧语言的ISO 8859-1，ISO-8859-1编码是单字节编码，向下兼容ASCII，其编码范围是0x00-0xFF，0x00-0x7F之间完全和ASCII一致，0x80-0x9F之间是控制字符，0xA0-0xFF之间是文字符号。
    俄罗斯的KOI-8，中国的GB 180030和GIG-5等。
    
#### 1.5、boolean类型

    boolean（布尔）类型有两个值：false和true，用来判断逻辑条件。整数值和布尔值之间不能进行相互转换。
    
### 2、变量

    在Java中，每个变量都有一个类型（type）。在声明变量时，变量的类型位于变量名之前。
    如果想要知道哪些Unicode字符属于Java中的“字母”，可以使用Character类的isJavaldentifierStart和isJavaldentifierPart方法来检查。
    
    利用关键字final指示常量，表示这个常量只能被赋值一次，一旦被赋值，就不能够在更改了。习惯上常量名使用全大写。
    可以使用static final设置一个类常量，在所有方法的外部，如果被声明public，则其它方法也可以使用这个常量。
    
### 3、运算符

    使用算术运算符+、-、*、/表示加减乘除运算，当参与/运算的两个操作数都是整数时，表示整数除法；否则表示浮点数除法。
    例如：15/2=7，15.0/2=7.5。需要注意的是，整数被0除将会产生一个异常，浮点数被0除将会得到无穷大和NaN结果。

    对于使用strictfp关键词标记的方法，必须使用严格的浮点计算来生成可再生的结果。如果一个类标记为strictfp，这个类中的所有方法都要使用严格的浮点计算。
    
#### 3.1、数学函数与常量

    Math类中，包含了各种各样的数学函数。
    计算一个数的平方根sqrt方法：Math.sqrt(X);
    还有一些常用的三角函数：Math.sin;Math.cos;Math.tan;Math.atan;Math.atan2;
    指数函数以及反函数：Math.exp;Math.log;Math.log10;
    用于表示π和e的两个常量：Math.PI;Math.E;
    
#### 3.2、数值类型之间的转换

    当两个数值进行二元操作时（例如：n + f，n是整数，f是浮点数），先将两个操作数转换为同一类型，然后在进行计算。
    如果两个操作数有一个是double类型，则另一个操作数就会转换为double类型。
    否则，如果其中一个操作数是float类型，另一个操作数将会转换为float类型。
    否则，如果其中一个操作数是long类型，另一个操作数将会转换为long类型。
    否则，两个操作数都将被转换为int类型。
    
#### 3.3、强制类型转换

    在Java中，允许进行数值之间的类型转换（int转double，double转int），当然，有可能会丢失一些信息。
    在这种情况下，需要通过强制类型转换（cast）实现这个操作。强制类型转换的语法格式在圆括号中给出要转换的目标类型。
    
    double x = 9.991; int nx = (int)x;
    变量nx的值为9，强制类型转换通过截断小数部分将浮点值转换为整型。
    
    如果想对浮点数进行舍入运算，以便得到最接近的整数（在很多情况下，这种操作更有用），那就需要使用Math.round方法。
    double x = 9.991;
    int nx = (int)Math.round(x);
    现在，变量 nx 的值为 10。
    当调用 round 的时候，仍然需要使用强制类型转换(int)。其原因是round方法返回的结果为long类型，由于存在信息丢失的可能性，
    所以只有使用显式的强制类型转换才能够将 long 类型转换成 int 类型。
    
### 4、字符串

