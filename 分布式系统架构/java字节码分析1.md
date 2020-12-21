java代码：

```java
public class Demo {
    public static void main(String[] args) {
        int a=1;
        int b=3;
        System.out.println(a);
        System.out.println(b);
    }
}
```

反编译字节码：

```java
D:\JavaWorkSpace\demo>javap -v target/classes/shuailee/Demo.class
Classfile /D:/JavaWorkSpace/demo/target/classes/shuailee/Demo.class
  Last modified 2019-3-28; size 553 bytes
  MD5 checksum 4a02a76aba7590c22382edd40beb0ecc
  Compiled from "Demo.java"
public class shuailee.Demo
  minor version: 0
  major version: 49
  flags: ACC_PUBLIC, ACC_SUPER
//常量池
Constant pool:
   #1 = Methodref          #5.#22         // java/lang/Object."<init>":()V
   #2 = Fieldref           #23.#24        // java/lang/System.out:Ljava/io/PrintStream;
   #3 = Methodref          #25.#26        // java/io/PrintStream.println:(I)V
   #4 = Class              #27            // shuailee/Demo
   #5 = Class              #28            // java/lang/Object
   #6 = Utf8               <init>
   #7 = Utf8               ()V
   #8 = Utf8               Code
   #9 = Utf8               LineNumberTable
  #10 = Utf8               LocalVariableTable
  #11 = Utf8               this
  #12 = Utf8               Lshuailee/Demo;
  #13 = Utf8               main
  #14 = Utf8               ([Ljava/lang/String;)V
  #15 = Utf8               args
  #16 = Utf8               [Ljava/lang/String;
  #17 = Utf8               a
  #18 = Utf8               I
  #19 = Utf8               b
  #20 = Utf8               SourceFile
  #21 = Utf8               Demo.java
  #22 = NameAndType        #6:#7          // "<init>":()V
  #23 = Class              #29            // java/lang/System
  #24 = NameAndType        #30:#31        // out:Ljava/io/PrintStream;
  #25 = Class              #32            // java/io/PrintStream
  #26 = NameAndType        #33:#34        // println:(I)V
  #27 = Utf8               shuailee/Demo
  #28 = Utf8               java/lang/Object
  #29 = Utf8               java/lang/System
  #30 = Utf8               out
  #31 = Utf8               Ljava/io/PrintStream;
  #32 = Utf8               java/io/PrintStream
  #33 = Utf8               println
  #34 = Utf8               (I)V
{
  public shuailee.Demo();
    descriptor: ()V
    flags: ACC_PUBLIC
    Code:
      stack=1, locals=1, args_size=1
         0: aload_0
         1: invokespecial #1                  // Method java/lang/Object."<init>":()V
         4: return
      LineNumberTable:
        line 9: 0
      LocalVariableTable:
        Start  Length  Slot  Name   Signature
            0       5     0  this   Lshuailee/Demo;

  public static void main(java.lang.String[]);
    descriptor: ([Ljava/lang/String;)V
    flags: ACC_PUBLIC, ACC_STATIC
    Code:
      stack=2, locals=3, args_size=1
         0: iconst_1  //int型1入栈 ->栈顶=1
         1: istore_1  //将栈顶的int型数值存入部变量 
         2: iconst_3
         3: istore_2
         4: getstatic     #2                  // Field java/lang/System.out:Ljava/io/PrintStream;
         7: iload_1
         8: invokevirtual #3                  // Method java/io/PrintStream.println:(I)V
        11: getstatic     #2                  // Field java/lang/System.out:Ljava/io/PrintStream;
        14: iload_2
        15: invokevirtual #3                  // Method java/io/PrintStream.println:(I)V
        18: return
      LineNumberTable:
        line 11: 0
        line 12: 2
        line 13: 4
        line 14: 11
        line 15: 18
      LocalVariableTable:
        Start  Length  Slot  Name   Signature
            0      19     0  args   [Ljava/lang/String;
            2      17     1     a   I
            4      15     2     b   I
}


```

