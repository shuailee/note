​	计算机是不能直接运行java代码的，必须要先运行java虚拟机，再由java虚拟机运行编译后的java代码。这个编译后的java代码，就是java字节码。 java字节码class文件本质上是一个以8位字节为基础单位的二进制流，各个数据项目严格按照顺序紧凑的排列在class文件中。jvm根据其特定的规则解析该二进制数据，从而得到相关信息;

#### 字节码文件

```java
//java代码在mvn compile 后会生成Demo.class字节码文件
public class Demo {
    public static void main(String[] args) {
        Integer a=1;
        Integer b=2;
        Integer c=a+b;
        System.out.println(c+"");
    }
}
```

下图为上面代码编译的字节码文件的16进制形式：

 ![1553769977644](D:\My File\blog\notes\img\1553769977644.png)

 在图中，前4个字节cafe babe就是魔数，紧接着魔数的4个字节代表的是Class文件的版本号，（第5,6个字节表示的是次版本号（minor version），在上图中为0000,说明class文件的次版本号为 0 ,第7,8个字节代表主版本号（major version），在上图中为0034,因为是16进制，计算可以得到该class文件的主版本号为52.）,以此类推，根据java字节码的规则，可以依次解析成该字节码文件的所有内容.

字节码文件的结构属性：

![1553770222872](D:\My File\blog\notes\img\1553770222872.png)

#### javap查看字节码文件 

​	 jdk中包含了一个可以将字节码文件“可视化”操作的命令javap，运行该命令，可以将java字节码文件解析为符合人类逻辑的文件。

```xml
D:\JavaWorkSpace\demo>javap -help
用法: javap <options> <classes>
其中, 可能的选项包括:
  -help  --help  -?        输出此用法消息
  -version                 版本信息
  -v  -verbose             输出附加信息
  -l                       输出行号和本地变量表
  -public                  仅显示公共类和成员
  -protected               显示受保护的/公共类和成员
  -package                 显示程序包/受保护的/公共类
                           和成员 (默认)
  -p  -private             显示所有类和成员
  -c                       对代码进行反汇编
  -s                       输出内部类型签名
  -sysinfo                 显示正在处理的类的
                           系统信息 (路径, 大小, 日期, MD5 散列)
  -constants               显示最终常量
  -classpath <path>        指定查找用户类文件的位置
  -cp <path>               指定查找用户类文件的位置
  -bootclasspath <path>    覆盖引导类文件的位置

```

**javap -c 命令**:   例如： javap -c target/classes/shuailee/Demo.class 

```java
D:\JavaWorkSpace\demo>javap -c  target/classes/shuailee/Demo.class
Compiled from "Demo.java"
public class shuailee.Demo {
  public shuailee.Demo();
    Code:
       0: aload_0
       1: invokespecial #1   // Method java/lang/Object."<init>":()V
       4: return
  public static void main(java.lang.String[]);
    Code:
       0: iconst_1
       1: invokestatic  #2  // Method java/lang/Integer.valueOf:(I)Ljava/lang/Integer;
       4: astore_1
       5: iconst_2
       6: invokestatic  #2  // Method java/lang/Integer.valueOf:(I)Ljava/lang/Integer;
...........
```

**javap -v 命令**:显示字节码文件的详细信息:

![1553769839659](D:\My File\blog\notes\img\1553769839659.png)

##### 常量池:

​	Constant pool意为常量池,可以理解成Class文件中的资源仓库。主要存放的是两大类常量：字面量(Literal)和符号引用(Symbolic References)。字面量类似于java中的常量概念，如文本字符串，final常量等，而符号引用则属于编译原理方面的概念，包括以下三种:
	1. 类和接口的全限定名(Fully Qualified Name)
	2. 字段的名称和描述符号(Descriptor)
	3. 方法的名称和描述符 
	不同于C/C++,JVM是在加载Class文件的时候才进行的动态链接，也就是说这些字段和方法符号引用只有在运行期转换后才能获得真正的内存入口地址。当虚拟机运行时，需要从常量池获得对应的符号引用，再在类创建或运行时解析并翻译到具体的内存地址中 .

反编译字节码文件：

```java
D:\JavaWorkSpace\demo>javap -v  target/classes/shuailee/Demo.class
Classfile /D:/JavaWorkSpace/demo/target/classes/shuailee/Demo.class
  Last modified 2019-3-28; size 936 bytes
  MD5 checksum eee8aed65d4835431d7636bf794de0c5
  Compiled from "Demo.java" //表示该字节码文件是由“Demo.java”编译而来
public class shuailee.Demo
  minor version: 0  //表示可以支持最小的jdk版本，java的jdk都是向下兼容的，所以该值一般都为0
  major version: 49 // 编译Demo.java的jdk版本，我使用的是jdk1.8,所以49代表的是jdk8
  flags: ACC_PUBLIC, ACC_SUPER //表示该类的访问属性
  
//常量池
Constant pool: 
//解读常量池中的内容:
   	// #1 表示常量池值的序号，该数字表示为第一个
    // #1 = Methodref 表示第一个值是一个方法的引用
    // #13.#31 表示该方法的完整属性还需要 #13和#22的来复合表示
   #1 = Methodref          #13.#31        // java/lang/Object."<init>":()V
   ...
   //#13 说明属性为class，而#45指明了该class为object类
   //将 #13和36结合一起，#13就的属性就是一个class类，且其具体的类为java.lang.Object，仔细观察 #13 后的注释，你发现了什么？其实在将java字节码可视化的时候，javap就已经将各个常量的属性都给关联好了
   #13 = Class              #45            // java/lang/Object
   #14 = Utf8               <init>
   #15 = Utf8               ()V
   ...
    //NameAndType 这一看知道是代表名字和类型
    //按照上述的方式解析，把14和15关联到一起会发现#31的名字是<init>，类型为()V表示无参数并且返回值为void
   #31 = NameAndType        #14:#15        // "<init>":()V
   ...
   //#45指明了该class为object类
   #45 = Utf8               java/lang/Object
   ...
   //根据上面的步骤分析可以得出：#1 引用的方法就是Test类的初始构造方法
```

方法表结合：在常量池之后的是对类内部的方法描述，在字节码中以表的集合形式表现 

```java
//接上面字节码文件
//这个Demo()方法就是我们Demo类中默认的构造方法，在我们没有重写或者没有指定对应的构造方法时，java编译的时候会默认生成一个空的构造方法。
public shuailee.Demo();
    descriptor: ()V  //descriptor· 表示该方法的描述，这里表示的是该方法的参数为空，且返回值为void
    flags: ACC_PUBLIC //表示该类的访问属性，同上表
    Code:
      stack=1, locals=1, args_size=1
          //表示运行方法时，各个指令的顺序
         0: aload_0
         1: invokespecial #1   // Method java/lang/Object."<init>":()V
         4: return
      LineNumberTable:
        line 9: 0
      LocalVariableTable:
        Start  Length  Slot  Name   Signature
            0       5     0  this   Lshuailee/Demo;

//main函数
  public static void main(java.lang.String[]);
    descriptor: ([Ljava/lang/String;)V
    flags: ACC_PUBLIC, ACC_STATIC
    Code:
      stack=3, locals=4, args_size=1
         //声明一个变量赋初始值的步骤  
         //Integer a=1;
         0: iconst_1
         1: invokestatic  #2   // Method java/lang/Integer.valueOf:(I)Ljava/lang/Integer;
         4: astore_1
         // Integer b=2;
         5: iconst_2
         6: invokestatic  #2   // Method java/lang/Integer.valueOf:(I)Ljava/lang/Integer;
         9: astore_2
        //一个计算并赋值的步骤：Integer c=a+b; 
        10: aload_1
        11: invokevirtual #3   // Method java/lang/Integer.intValue:()I
        14: aload_2
        15: invokevirtual #3   // Method java/lang/Integer.intValue:()I
        18: iadd
        19: invokestatic  #2    // Method java/lang/Integer.valueOf:(I)Ljava/lang/Integer;
        22: astore_3
        //一条打印命令 System.out.println(c+"");
        23: getstatic     #4    // Field java/lang/System.out:Ljava/io/PrintStream;
        26: new           #5    // class java/lang/StringBuilder
        29: dup
        30: invokespecial #6     // Method java/lang/StringBuilder."<init>":()V
        33: aload_3
        34: invokevirtual #7  // Method java/lang/StringBuilder.append:(Ljava/lang/Object;)Ljava/lang/StringBuilder;
        37: ldc           #8   // String
        39: invokevirtual #9    // Method java/lang/StringBuilder.append:(Ljava/lang/String;)Ljava/lang/StringBuilder;
        42: invokevirtual #10   // Method java/lang/StringBuilder.toString:()Ljava/lang/String;
        45: invokevirtual #11   // Method java/io/PrintStream.println:(Ljava/lang/String;)V
        48: return
        //结束         
      LineNumberTable:
        line 12: 0
        line 13: 5
        line 14: 10
        line 15: 23
        line 16: 48
      LocalVariableTable:
        Start  Length  Slot  Name   Signature
            0      49     0  args   [Ljava/lang/String;
            5      44     1     a   Ljava/lang/Integer;
           10      39     2     b   Ljava/lang/Integer;
           23      26     3     c   Ljava/lang/Integer;
}


```

code内的主要属性为: 

1. **stack** 最大操作数栈，JVM运行时会根据这个值来分配栈帧(Frame)中的操作栈深度,此处为1
2. **locals:** 局部变量所需的存储空间，单位为Slot, Slot是虚拟机为局部变量分配内存时所使用的最小单位，为4个字节大小。方法参数(包括实例方法中的隐藏参数this)，显示异常处理器的参数(try catch中的catch块所定义的异常)，方法体中定义的局部变量都需要使用局部变量表来存放。值得一提的是，locals的大小并不一定等于所有局部变量所占的Slot之和，因为局部变量中的Slot是可以重用的。
3. **args_size:** 方法参数的个数，这里是1，因为每个实例方法都会有一个隐藏参数this
4. **attribute_info** 方法体内容，0,1,4为字节码"行号"，该段代码的意思是将第一个引用类型本地变量推送至栈顶，然后执行该类型的实例方法，也就是常量池存放的第一个变量，也就是注释里的"java/lang/Object."":()V", 然后执行返回语句，结束方法。
5. **LineNumberTable** 该属性的作用是描述源码行号与字节码行号(字节码偏移量)之间的对应关系。可以使用 -g:none 或-g:lines选项来取消或要求生成这项信息，如果选择不生成LineNumberTable，当程序运行异常时将无法获取到发生异常的源码行号，也无法按照源码的行数来调试程序。
6. **LocalVariableTable** 该属性的作用是描述帧栈中局部变量与源码中定义的变量之间的关系。可以使用 -g:none 或 -g:vars来取消或生成这项信息，如果没有生成这项信息，那么当别人引用这个方法时，将无法获取到参数名称，取而代之的是arg0, arg1这样的占位符。 start 表示该局部变量在哪一行开始可见，length表示可见行数，Slot代表所在帧栈位置，Name是变量名称，然后是类型签名。

 

 //https://juejin.im/post/5aca2c366fb9a028c97a5609

http://www.importnew.com/13107.html

 

 