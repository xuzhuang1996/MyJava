1. 用处的话：Java新特性出来的时候，可以对比分析原理
2. 查看class字节码反编译文件：`javap -p -v BaseRsp.class`
3. 常量池解析：
   - Methodref表示类中方法的符号引用；后面依次是父类、方法原型。
   - Fieldref表示类中字段的符号引用；后面依次是所属类，字段原型。
   - String表示字符串类型的字面量。
   - Utf8表示utf-8的字符串。
   - Class表示类的符号引用。
   - NameAndType表示方法的符号引用；后面依次是方法名，方法描述符（入参、返回）
4. 方法表：

        public void hello();
            descriptor: ()V
            flags: ACC_PUBLIC
            Code:
              stack=2, locals=2, args_size=1
                 0: iconst_1
                 1: istore_1
                 2: getstatic     #3                  // Field java/lang/System.out:Ljava/io/PrintStream;
                 5: ldc           #4                  // String hello world
                 7: invokevirtual #5                  // Method java/io/PrintStream.println:(Ljava/lang/String;)V
                10: return
              LineNumberTable:
                line 12: 0
                line 13: 2
                line 14: 10
              LocalVariableTable:
                Start  Length  Slot  Name   Signature
                    0      11     0  this   LDemo2;
                    2       9     1     i   I
6. JVM将code称属性，局部变量表也称属性。根据上面的方法表，相关定义为：
	 - stack 最大操作数栈；JVM运行时会根据这个值来分配栈帧(Frame)中的操作栈深度
   - locals: 局部变量所需的存储空间。单位为Slot，4个字节大小。
   - args_size: 方法参数的个数。这里是1，因为每个实例方法都会有一个隐藏参数this。
7. LineNumberTable 行号表，该属性的作用是描述源码行号与字节码行号(字节码偏移量)之间的对应关系。每一行class文件行数，对应Code表下的代码行数。如12行对应code表下的0: iconst_1
8. LocalVariableTable 该属性的作用是描述帧栈中局部变量与源码中定义的变量之间的关系。hello()方法中，两个局部变量this，i。对于上诉的Code表，局部变量I,从第2行开始生效，生效长度为9，即到第11行。与this变量一致。

