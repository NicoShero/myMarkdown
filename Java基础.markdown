# 参考资料
* [Primitive Data Types](https://docs.oracle.com/javase/tutorial/java/nutsandbolts/datatypes.html)
* [The Java® Virtual Machine Specification](https://docs.oracle.com/javase/specs/jvms/se8/jvms8.pdf)
* [Autoboxing and Unboxing](https://docs.oracle.com/javase/tutorial/java/data/autoboxing.html)
* [StackOverflow : Differences between new Integer(123), Integer.valueOf(123) and just 123](https://stackoverflow.com/questions/9030817/differences-between-new-integer123-integer-valueof123-and-just-123)
* [Program Creek : Why String is immutable in Java?](https://www.programcreek.com/2013/04/why-string-is-immutable-in-java/)
* [StackOverflow : String, StringBuffer, and StringBuilder](https://stackoverflow.com/questions/2971315/string-stringbuffer-and-stringbuilder)
* [StackOverflow : What is String interning?](https://stackoverflow.com/questions/10578984/what-is-string-interning)
* [深入解析 String#intern](https://tech.meituan.com/in_depth_understanding_string_intern.html)
* [StackOverflow: Is Java “pass-by-reference” or “pass-by-value”?](https://stackoverflow.com/questions/40480/is-java-pass-by-reference-or-pass-by-value)
* [StackOverflow : Why don't Java's +=, -=, *=, /= compound assignment operators require casting?](https://stackoverflow.com/questions/8710619/why-dont-javas-compound-assignment-operators-require-casting)
* [StackOverflow : Why can't your switch statement data type be long, Java?](https://stackoverflow.com/questions/2676210/why-cant-your-switch-statement-data-type-be-long-java)
* [Abstract Methods and Classes](https://docs.oracle.com/javase/tutorial/java/IandI/abstract.html)
* [深入理解 abstract class 和 interface](https://www.ibm.com/developerworks/cn/java/l-javainterface-abstract/)
* [When to Use Abstract Class and Interface](https://dzone.com/articles/when-to-use-abstract-class-and-intreface)
* [Java 9 Private Methods in Interfaces](https://www.journaldev.com/12850/java-9-private-methods-interfaces)
* [Using the Keyword super](https://docs.oracle.com/javase/tutorial/java/IandI/super.html)
* [Trail: The Reflection API](https://docs.oracle.com/javase/tutorial/reflect/index.html)
* [深入解析 Java 反射（1）- 基础](http://www.sczyh30.com/posts/Java/java-reflection-1/)
* [Java Exception Interview Questions and Answers](https://www.journaldev.com/2167/java-exception-interview-questions-and-answersl)
* [Java提高篇——Java 异常处理](https://www.cnblogs.com/Qian123/p/5715402.html)
* [Java 泛型详解](https://www.cnblogs.com/Blue-Keroro/p/8875898.html)
* [10 道 Java 泛型面试题](https://cloud.tencent.com/developer/article/1033693)
* [注解 Annotation 实现原理与自定义注解例子](https://www.cnblogs.com/acm-bingzi/p/javaAnnotation.html)
* [Difference between Java 1.8 and Java 1.7?](http://www.selfgrowth.com/articles/difference-between-java-18-and-java-17)
* [Java 8 特性](http://www.importnew.com/19345.html)
* [What are the main differences between Java and C++?](http://cs-fundamentals.com/tech-interview/java/differences-between-java-and-cpp.php)

# 数据类型
## 基本类型
* byte / 8 位 / 1 字节
* char / 16 / 2
* short / 16 / 2
* int / 32 / 4
* float / 32 / 4
* long / 64 / 8
* double / 64 / 8
* boolean / * / *

## 包装类型
自动装修 (Integer.valueOf(2))
自动拆箱 (Integer.intValue())

## 缓存池
* new Integer(123) 每次都会新建对象
* Integer.valueOf(123) 会从缓存池获取对象，多次调用只会取得对象引用

valueOf() 方法实现简单，判断值是否存在缓存池中，是则返回对象引用，否则新建对象。

    public static Integer valueOf(int i) {
        if (i >= IntegerCache.low && i <= IntegerCache.high)
            return IntegerCache.cache[i + (-IntegerCache.low)];
        return new Integer(i);
    }

Java 8 中，Integer 缓存池大小默认为 -128 ~ 127

    static final int low = -128;
    static final int high;
    static final Integer cache[];

    static {
        // high value may be configured by property
        int h = 127;
        String integerCacheHighPropValue =
            sun.misc.VM.getSavedProperty("java.lang.Integer.IntegerCache.high");
        if (integerCacheHighPropValue != null) {
            try {
                int i = parseInt(integerCacheHighPropValue);
                i = Math.max(i, 127);
                // Maximum array size is Integer.MAX_VALUE
                h = Math.min(i, Integer.MAX_VALUE - (-low) -1);
            } catch( NumberFormatException nfe) {
                // If the property cannot be parsed into an int, ignore it.
            }
        }
        high = h;

        cache = new Integer[(high - low) + 1];
        int j = low;
        for(int k = 0; k < cache.length; k++)
            cache[k] = new Integer(j++);

        // range [-128, 127] must be interned (JLS7 5.1.7)
        assert IntegerCache.high >= 127;
    }

**基本类型缓冲区如下**：
* boolean values true and false
* all byte values
* short values between -128 ~ 127
* int values between -128 ~ 127
* char in the range \u0000 ~ \u007F

在使用这些基本类型对应的包装类型时，如果该数值范围在缓冲池范围内，就可以直接调用对象引用。   
JDK 1.8 中，可以通过 -XX:AutoBoxCacheMax=<Size> 来指定缓冲池的大小，该选项在 JVM 初始化的时候会设定一个名为 java.lang.IntegerCache.high 系统属性，然后 IntegerCache 初始化的时候就会读取该系统属性来决定上限。

# String
Java 8 中，String 内部使用 char 数组存储数据  

    public final class String
        implements java.io.Serializable, Comparable<String>, CharSequence {
        /** The value is used for character storage. */
        private final char value[];
    }

Java 9 以后，String 改用 byte 数组存储，同时使用 coder 标识使用了那种编码

    public final class String
        implements java.io.Serializable, Comparable<String>, CharSequence {
        /** The value is used for character storage. */
        private final byte[] value;

        /** The identifier of the encoding used to encode the bytes in {@code value}. */
        private final byte coder;
    }

value 数组被声明为 final ， 这意味着 value 数组初始化后不能再引用其他数组。并且 String 内部没有改变 value 数组的方法，因此可以保证 String 不可变。

## 不可变的好处
### 可以缓存 hash 值
因为 String 的 hash 值经常被使用，例如 String 当做 HashMap 的 key。

### String Pool 的需要
如果一个 String 对象已经建立过了，那么会从 String Pool 处获得引用。

![](assets/Java基础-90a7dba7.png)

### 安全性
String 经常作为参数， String 不可变性可以保证参数不可变。例如作为网络连接参数的情况下如果 String 是可变的，那么网络连接过程中, String 被改变，改变 String 的那一方以为现在连接的是其他主机，而实际情况不一定是。

### 线程安全
String 不可变性天生具备线程安全，可以再多个线程中安全使用。

## String，StringBuffer，StringBuilder
### 1. 可变性
* String 不可变
* StringBuffer 和 StringBuilder 可变

### 2. 线程安全
* String 是不可变，线程安全
* StringBuilder 线程不安全
* StringBuffer 是线程安全的，内部使用 synchronized 关键字同步

## String Pool
字符串常量池 (String Pool) 保存着所有字符串字面量 (literal strings)，这些字面量在编译时期就确定。不仅如此，还可以使用 String 的 intern() 方法在运行过程将字符串添加到 String Pool 中。   
当一个字符串调用 intern() 方法时，如果 String Pool 中已经存在一个字符串和该字符串值相等 (使用 equals() 方法进行确定)，那么就会返回 String Pool 中字符串的引用；否则，就会在 String Pool 中添加一个新的字符串，并返回这个新字符串的引用。   
s1 和 s2 采用 new String() 的方式建立两个不同字符串，s3 和 s4 通过 s1.intern() 和 s2.intern() 方法取得同一个字符串引用。 intern() 首先把 "aaa" 放入 String Pool 中，然后返回字符串引用，因此 s3 和 s4 是同一个引用。

    String s1 = new String("aaa");
    String s2 = new String("aaa");
    System.out.println(s1 == s2);           // false
    String s3 = s1.intern();
    String s4 = s2.intern();
    System.out.println(s3 == s4);           // true

如果是使用 "bbb" 这种字面量的形式创建字符串，会将其自动放入 String Pool 中。

    String s5 = "bbb";
    String s6 = "bbb";
    System.out.println(s5 == s6);  // true

Java 7 之前，String Pool 被放在运行时常量池中，它属于永久代。而在 Java 7 中，String Pool 被移到堆中，因为永久代空间有限，大量使用字符串会出现 OutOfMemoryError 错误。

## new String("abc")
使用这种方式一共会创建两个字符串对象 (如果池中没有该对象)。
* "abc"属于字符串字面量，会在 String Pool 中创建对象指向 "abc" 这个字面量
* 使用 new 的方式会在堆中创建一个字符串对象


    public class NewStringTest {
        public static void main(String[] args) {
            String s = new String("abc");
        }
    }

使用 javap -verbose 反编译：

    // ...
    Constant pool:
    // ...
       #2 = Class              #18            // java/lang/String
       #3 = String             #19            // abc
    // ...
      #18 = Utf8               java/lang/String
      #19 = Utf8               abc
    // ...

      public static void main(java.lang.String[]);
        descriptor: ([Ljava/lang/String;)V
        flags: ACC_PUBLIC, ACC_STATIC
        Code:
          stack=3, locals=2, args_size=1
             0: new           #2                  // class java/lang/String
             3: dup
             4: ldc           #3                  // String abc
             6: invokespecial #4                  // Method java/lang/String."<init>":(Ljava/lang/String;)V
             9: astore_1
    // ...

在 Constant Pool 中， #19 存储着字符串字面量 "abc" ， #3 是 String Pool 的字符串对象， 它指向 #19 这个字符串字面量。   
main 方法中 0: 行使用 new #2 在堆中创建了一个字符串对象， ldc #3 将 StringPool 中的字符串对象作为 String 构造函数的参数。

以下是 String 构造函数源码，用已有对象作为入参时，内容会指向已有对象的数组。

    public String(String original) {
        this.value = original.value;
        this.hash = original.hash;
    }

# 运算
## 参数传递
Java 参数是以值传递的形式，不是引用传递。本质上是将对象的地址以值的形式传递到形参中。   

## float 和 double
Java 不能隐式执行向下转型，会使得精度降低。    
比如 1.1 属于 double 类型，不能将 1.1 直接赋值给 float ，1.1f 字面量才是 float 类型。

    float f = 1.1f;

## 隐式类型转换
比如 1 是 int 类型，精度比 short 要高，但是 += 或者 ++ 会执行隐式类型转换。

    short s1 = 1；
    ✖ s1 = s1 + 1
    ✔ s1 += 1  (s1 = (short)(s1 + 1))

## switch
从 Java 7 开始，可以在 switch 条件判断语句使用 String 对象

    String s = "a";
    switch (s) {
        case "a":
            System.out.println("aaa");
            break;
        case "b":
            System.out.println("bbb");
            break;
    }

switch 不支持 long、float、double，因为 switch 初衷是只对少数几个值的类型进行判断。    


# 关键字
## final
### 1. 数据
声明数据为常量，可以是编译时常量也可以是初始化后不可变的常量。
* 对于基本类型，final 使数值不变
* 对于引用类型，final 使引用不变，但是被引用对象本身可变

### 2. 方法
声明方法不能被子类重写，private 方法隐式地被指定为 final ，如果在子类中定义的方法和基类中的一个 privte 方法签名相同，此时子类是定义了一个新的方法

### 3. 类
声明类不允许被继承


## static
### 1. 静态变量
* 静态变量：又称为类变量，也就是说这个变量是属于类的，类的所有实例共享静态变量，可以直接通过类名访问它。静态变量内存中只有一份。


    public class A {

        private int x;         // 实例变量
        private static int y;  // 静态变量

        public static void main(String[] args) {
            // int x = A.x;  // Non-static field 'x' cannot be referenced from a static context
            A a = new A();
            int x = a.x;
            int y = A.y;
        }
    }

### 2. 静态方法
静态方法在类加载的时候就存在了，不依赖于任何实例。所以静态方法必须有实现，他不能是抽象方法。

    public abstract class A {
        public static void func1(){
        }
        // public abstract static void func2();  
        // Illegal combination of modifiers: 'abstract' and 'static'
    }

只能访问所属类的静态字段和静态方法，方法中不能有 this 和 super 关键字，因为这两个关键字与具体对象关联。

    public class A {

        private static int x;
        private int y;

        public static void func1(){
            int a = x;
            // int b = y;  
            // Non-static field 'y' cannot be referenced from a static context
            // int b = this.y;     
            // 'A.this' cannot be referenced from a static context
        }
    }

### 3. 静态语句块
只会在类初始化时执行一次。   

    public class A {
        static {
            System.out.println("123");
        }

        public static void main(String[] args) {
            A a1 = new A();
            A a2 = new A();
        }
    }
.

    123

### 4. 静态内部类
非静态内部类依赖于外部类的实例，也就是说需要先创建外部类实例，才能使用实例去创建非静态内部类。而静态内部类在类初始化时一同初始化。

    public class OuterClass {

        class InnerClass {
        }

        static class StaticInnerClass {
        }

        public static void main(String[] args) {
            // InnerClass innerClass = new InnerClass(); // 'OuterClass.this' cannot be referenced from a static context
            OuterClass outerClass = new OuterClass();
            InnerClass innerClass = outerClass.new InnerClass();
            StaticInnerClass staticInnerClass = new StaticInnerClass();
        }
    }

静态内部类不能访问外部类和非静态的变量和方法。

### 5. 静态导包
在使用静态变量和方法时不用指明 className ，从而简化代码，但是可读性降低

    import static com.xxx.ClassName.*

### 6. 初始化顺序
静态变量和静态语句块优先于实例变量和普通语句块，静态变量和静态语句块的初始化顺序取决于代码的顺序

    public static String staticField = "静态变量";
.

    static {
        System.out.println("静态语句块");
    }
.

    public String field = "实例变量";
    {
        System.out.println("普通语句块");
    }
最后才是构造函数的初始化。

    public InitialOrderTest() {
        System.out.println("构造函数");
    }

存在继承的情况下，初始化孙旭为：
*  父类（静态变量、静态语句块）
*  子类（静态变量、静态语句块）
*  父类（实例变量、普通语句块）
*  父类（构造函数）
*  子类（实例变量、普通语句块）
*  子类（构造函数）

# Object 通用方法
## equals
### 1. 等价关系
两个对象具有等价关系，需要满足五个条件：

**Ⅰ 自反性**
> x.equals(x)

**Ⅱ 对称性**
> x.equals(y) == y.equals(x)

**Ⅲ 传递性**
> if (x.equals(y) == y.equals(z))   
x.equals(z);

**Ⅳ 一致性**
> x.equals(y) == x.equals(y)

**Ⅴ 与 null 的比较**
> x.equals(null) // false , 除非 x 也是 null

### 2. 等价与相等
* 对于基本类型， == 判断两个值是否相等，基本类型没有 equals() 方法
* 对于引用类型， == 判断两个变量是否引用同一个对象，而 equals() 判断引用的对象是否等价。


    Integer x = new Integer(1);
    Integer y = new Integer(1);
    System.out.println(x.equals(y)); // true
    System.out.println(x == y);      // false

### 3. 实现
*    检查是否为同一个对象的引用，如果是直接返回 true；
*    检查是否是同一个类型，如果不是，直接返回 false；
*    将 Object 对象进行转型；
*    判断每个关键域是否相等。


    public class EqualExample {
        private int x;
        private int y;
        private int z;
        public EqualExample(int x, int y, int z) {
            this.x = x;
            this.y = y;
            this.z = z;
        }

        @Override
        public boolean equals(Object o) {
            if (this == o) return true;
            if (o == null || getClass() != o.getClass()) return false;

            EqualExample that = (EqualExample) o;

            if (x != that.x) return false;
            if (y != that.y) return false;
            return z == that.z;
        }
    }

## hashCode()
hashCode() 返回哈希值，而 equals() 是用来判断两个对象是否等价。等价的对象哈希值一定相同，但是哈希值相同的对象不一定等价，这是由于计算哈希值具有随机性，两个值不同的对象可能计算出相同的哈希值。    
在覆盖 equals() 方法时应当总是覆盖 hashCode() 方法，保证等价的两个对象哈希值也相等。   
HashSet 和 HashMap 等集合类使用了 hashCode() 方法来计算对象应该存储的位置，因此要将对象添加到这些集合类中，需要让对应的类实现 hashCode() 方法。    
下面的代码中，新建了两个等价的对象，并将它们添加到 HashSet 中。我们希望将这两个对象当成一样的，只添加一个对象到集合。但是 EqualExample 没有实现 hashCode() 方法，因此两个哈希值不相同，所以导致集合添加了两个等价的对象。

    EqualExample e1 = new EqualExample(1, 1, 1);
    EqualExample e2 = new EqualExample(1, 1, 1);
    System.out.println(e1.equals(e2)); // true
    HashSet<EqualExample> set = new HashSet<>();
    set.add(e1);
    set.add(e2);
    System.out.println(set.size());   // 2

理想的哈希函数具有均匀性，即不相等的对象应当均匀分布到所有可能的哈希值上。这就要求哈希函数要把所有域的值都考虑进来。可以将每个域都当成 R 进制的某一位，然后组成一个 R 进制的整数。    
R 一般取 31 ,因为它是一个奇素数，如果是偶数的话，当发生乘法溢出，信息会丢失，相当于相乘后左位移一位，导致最左位的数值丢失。   
一个数和 31 相乘可以转换成移位和减法：31*x == (x<<5) - x， 编译器会自动进行该优化。

    @Override
    public int hashCode() {
        int result = 17;
        result = 31 * result + x;
        result = 31 * result + y;
        result = 31 * result + z;
        return result;
    }

## toString()
默认返回 ToStringExample@4554617c 这种形式，其中 @ 后面为哈希值的无符号十六进制表示。

    public class ToStringExample {

        private int number;

        public ToStringExample(int number) {
            this.number = number;
        }
    }
.

    ToStringExample example = new ToStringExample(123);
    System.out.println(example.toString());
.

    ToStringExample@4554617c

## clone()
### 1. cloneable
clone() 是 Object 的 protected 方法，不是 public ，一个类不显式去重写 clone(), 其它类就不能直接去调用该类实例的 clone() 方法。

    public class CloneExample {
        private int a;
        private int b;
    }
.

    CloneExample e1 = new CloneExample();
    // CloneExample e2 = e1.clone(); // 'clone()' has protected access in 'java.lang.Object'

重写 clone() 得到以下实现：

    public class CloneExample {
        private int a;
        private int b;

        @Override
        public CloneExample clone() throws CloneNotSupportedException {
            return (CloneExample)super.clone();
        }
    }
.

    CloneExample e1 = new CloneExample();
    try {
        CloneExample e2 = e1.clone();
    } catch (CloneNotSupportedException e) {
        e.printStackTrace();
    }
.

    java.lang.CloneNotSupportedException: CloneExample
以上抛出了 CloneNotSupportedException，这是因为 CloneExample 没事实现 Cloneable 接口。   
应该注意的是，clone() 方法并不是 Cloneable 接口的方法，它是一个规定。一个类没有实现 Cloneable 接口又调用了 clone() 方法就会抛出异常。

    public class CloneExample implements Cloneable {
        private int a;
        private int b;

        @Override
        public Object clone() throws CloneNotSupportedException {
            return super.clone();
        }
    }

### 2. 浅拷贝
拷贝对象和原始对象的引用类型引用同一个对象。
    public class ShallowCloneExample implements Cloneable {

        private int[] arr;

        public ShallowCloneExample() {
            arr = new int[10];
            for (int i = 0; i < arr.length; i++) {
                arr[i] = i;
            }
        }

        public void set(int index, int value) {
            arr[index] = value;
        }

        public int get(int index) {
            return arr[index];
        }

        @Override
        protected ShallowCloneExample clone() throws CloneNotSupportedException {
            return (ShallowCloneExample) super.clone();
        }
    }
.

    ShallowCloneExample e1 = new ShallowCloneExample();
    ShallowCloneExample e2 = null;
    try {
        e2 = e1.clone();
    } catch (CloneNotSupportedException e) {
        e.printStackTrace();
    }
    e1.set(2, 222);
    System.out.println(e2.get(2)); // 222

### 3. 深拷贝
拷贝对象和原始对象的引用类型引用不同对象。
    public class DeepCloneExample implements Cloneable {

        private int[] arr;

        public DeepCloneExample() {
            arr = new int[10];
            for (int i = 0; i < arr.length; i++) {
                arr[i] = i;
            }
        }

        public void set(int index, int value) {
            arr[index] = value;
        }

        public int get(int index) {
            return arr[index];
        }

        @Override
        protected DeepCloneExample clone() throws CloneNotSupportedException {
            DeepCloneExample result = (DeepCloneExample) super.clone();
            result.arr = new int[arr.length];
            for (int i = 0; i < arr.length; i++) {
                result.arr[i] = arr[i];
            }
            return result;
        }
    }
.

    DeepCloneExample e1 = new DeepCloneExample();
    DeepCloneExample e2 = null;
    try {
        e2 = e1.clone();
    } catch (CloneNotSupportedException e) {
        e.printStackTrace();
    }
    e1.set(2, 222);
    System.out.println(e2.get(2)); // 2

### 4. clone() 的代替方案
使用拷贝构造函数或者拷贝工厂来拷贝一个对象。    

    public class CloneConstructorExample {

        private int[] arr;

        public CloneConstructorExample() {
            arr = new int[10];
            for (int i = 0; i < arr.length; i++) {
                arr[i] = i;
            }
        }

        public CloneConstructorExample(CloneConstructorExample original) {
            arr = new int[original.arr.length];
            for (int i = 0; i < original.arr.length; i++) {
                arr[i] = original.arr[i];
            }
        }

        public void set(int index, int value) {
            arr[index] = value;
        }

        public int get(int index) {
            return arr[index];
        }
    }
.

    CloneConstructorExample e1 = new CloneConstructorExample();
    CloneConstructorExample e2 = new CloneConstructorExample(e1);
    e1.set(2, 222);
    System.out.println(e2.get(2)); // 2

# 继承
## 访问权限
Java 中有三个访问权限修饰符：private、protected 以及 public，如果不加修饰符代表包内可见。     
类、类中的字段或方法都可以加修饰符。
* 类可见表示其他类可以用这个类创建实例对象    
* 成员可见表示其他类可以用这个类的实例对象访问到该成员

protected 用于修饰成员，表示在继承体系中成员对于子类可见，但是这个访问修饰符对于类没有意义。   
设计良好的模块会隐藏所有的实现细节，把它的 API 与它的实现清晰地隔离开。模块之间只通过 API 进行通信，一个模块不需要知道其他模块的内部工作情况，这个概念被称为信息隐藏或封装，因此访问权限应当尽可能地使每个类或者成员不被外界访问。    
如果子类的方法重写了父类的方法，那么子类中该方法的访问级别不能低于父类访问级别，这是为了确保父类的实例都可以使用子类去代替，就是里氏替换原则。   
字段决不能是 public 的，否则就失去了对字段修改的控制权。但如果是包级别私有的嵌套类，那暴露也无所谓。

## 抽象类与接口
### 1. 抽象类
抽象类和抽象方法用 abstract 关键字声明，如果一个类包含抽象方法，那么类必须声明为抽象类。抽象类只能被继承不可以实例化。

### 2. 接口
接口是抽象类的延伸，Java 8 之前可以看做是完全抽象的类，不能有方法实现。
Java 8 开始，接口也可以有默认的方法实现。    
接口的成员默认都是 public 的，Java 9 开始允许将方法定义为 private 这样能定义某些复用的代码又不会把方法暴露出去。    
接口的字段默认是 static 和 final 的。    

### 3. 比较
* 从设计层面上看，抽象类提供一种 IS-A 关系，需要满足里氏替换原则，即子类对象必须可以替换所有父类对象。而接口更像是 LIKE-A 关系，它提供一种方法实现契约，让实现它的类都支持它的方法并实现。
* 使用上，一个类可以实现多个接口，但只能继承一个类
* 接口字段只能是 static 和 final 的，抽象类的字段没有限制
* 接口的成员只能是 public 的，抽象类的成员可以有多重访问权限

### 4. 使用选择
接口：
* 需要让不相关的类都实现一个方法，比如 compareTo() 方法
* 需要使用多重继承的时候

抽象类：
* 需要在相关的类中共享代码
* 需要能控制继承来的的成员的访问权限，因此不能都是 public
* 需要继承非静态和非常量字段

多数情况下，接口优先于抽象类。因此接口没有抽象类严格的类层次结构的要求，可以灵活地为一个类添加行为。    
Java 8 开始，接口也可以有默认的方法实现，使得接口的修改成本降低。

## super
* 访问父类的构造函数：使用 super() 访问父类构造函数，子类一定会调用父类的构造函数来完成初始化工作。
* 访问父类的成员： 如果子类重写了父类的某个方法，可以通过使用 super 关键字来引用父类的方法实现 (super.action())。


## 重写和重载
### 1. 重写 (Override)
子类实现了一个声明和父类完全相同的方法。为了满足里氏替换原则：
* 子类方法的访问权限必须宽松于父类 (public > protected > private)
* 子类方法的返回类型必须是父类方法的返回类型或其子类
* 子类方法抛出的异常类型必须是父类抛出的异常类型或其子类型

使用 @Override 注解可以让编译器帮忙检验是否满足条件。    
调用方法后，先从本类寻找对应的方法，没有则从父类中寻找。优先级为：
* this.action(this);
* super.action(this);
* this.action(super);
* super.action(super);


    class A {
        public void show(A obj) {
            System.out.println("A.show(A)");
        }
        public void show(C obj) {
            System.out.println("A.show(C)");
        }
    }

    class B extends A {
        @Override
        public void show(A obj) {
            System.out.println("B.show(A)");
        }
    }

    class C extends B {
    }

    class D extends C {
    }
.

    public static void main(String[] args) {
        A a = new A();
        B b = new B();
        C c = new C();
        D d = new D();

        // 在 A 中存在 show(A obj)，直接调用
        a.show(a); // A.show(A)
        // 在 A 中不存在 show(B obj)，将 B 转型成其父类 A
        a.show(b); // A.show(A)
        // 在 B 中存在从 A 继承来的 show(C obj)，直接调用
        b.show(c); // A.show(C)
        // 在 B 中不存在 show(D obj)，但是存在从 A 继承来的 show(C obj)，将 D 转型成其父类 C
        b.show(d); // A.show(C)
        // 引用的还是 B 对象，所以 ba 和 b 的调用结果一样
        A ba = new B();
        ba.show(c); // A.show(C)
        ba.show(d); // A.show(C)
    }

### 2. 重载 (Overload)
同一个类中，方法名称相同，但是参数的类型、个数、顺序上至少有一个不同。   
应该注意的是，返回值不同，其他都相同不算重载。

# 反射
每个类都有一个 Class 对象，包含了类相关的信息，编译一个新类的时候，都会产生一个同名的 .class 文件，该文件内容保存着 Class 对象。   
类加载相当于 Class 对象的加载，类在第一次使用时才动态加载到 JVM 中。也可以使用 Class.forName("com.mysql.jdbc.Driver") 这种方式来控制类的加载，方法会返回一个 Class 对象。    
反射可以提供运行时的类信息，可以在运行时加载，甚至编译时期不存在的 .class 也可以记载进来。   
Class 和 Java.lang.reflect 一起对反射提供了支持，Java.lang.reflect 类库主要包含以下三个类：
* Filed：使用 get() 和 set() 读取和修改 Filed 对象关联字段；
* Method：使用 invoke() 方法调用和 Method() 对象关联的方法；
* Constructor：使用 Constructor 的 newInstance() 创建新对象。

## 反射优点：
* 可扩展性：应用程序可以利用全限定名创建可扩展对象的实例，来使用后外部用户的自定义类
* 类浏览器和可视化开发环境：一个类浏览器需要枚举类的成员。可视化开发环境利用反射中可用的类型信息中收益，帮助程序员开发
* 调试器和测试工具：调试器需要能够检查一个类里的私有成员。测试工具可以利用反射来自动调用类里定义的可被发现的 API 定义，以确保代码覆盖率

## 反射缺点：
* 性能开销：设计动态类型解析，所以 JVM 无法对这些代码进行优化，因此，反射操作的效率要比哪些非反射操作低的多。
* 安全限制：反射要求程序必须在没有安全限制的环境中运行。
* 内部暴露：由于反射允许代码执行一些在正常情况下不被允许的操作(访问私有成员)，所以使用反射可能导致意外的副作用，导致代码功能失调并破坏可移植性，和抽象性。

# 异常
Throwable 可以用来表示任何可以作为异常抛出的类，分为 Error 和 Exception。    
Error 代表 JVM 无法处理的错误，Exception 则分为：
* 受检异常：通过 try...catch... 抛出的异常，可控并能继续运行
* 非受检异常：即运行时错误，发生后程序崩溃

# 泛型

    public class Box<T> {
        // T stands for "Type"
        private T t;
        public void set(T t) { this.t = t; }
        public T get() { return t; }
    }


# 注解
在工具编译，运行时进行解析和使用，起到说明、配置的功能。

# 特性
各版本新特性
## JAVA SE 8
* Lambda Expressions
* Pipelines and Streams
* Date and Time API
* Default Methods
* Type Annotations
* Nashhorn JavaScript Engine
* Concurrent Accumulators
* Parallel operations
* PermGen Error Removed

## JAVA SE 7
* Strings in Switch Statement
* Type Inference for Generic Instance Creation
* Multiple Exception Handling
* Support for Dynamic Languages
* Try with Resources
* Java nio Package
* Binary Literals, Underscore in literals
* Diamond Syntax

## Java 与 C++ 的区别
* Java 是纯粹的面向对象语言，所有的对象都继承自 java.lang.Object，C++ 为了兼容 C 即支持面向对象也支持面向过程。
* Java 通过虚拟机从而实现跨平台特性，但是 C++ 依赖于特定的平台。
* Java 没有指针，它的引用可以理解为安全指针，而 C++ 具有和 C 一样的指针。
* Java 支持自动垃圾回收，而 C++ 需要手动回收。
* Java 不支持多重继承，只能通过实现多个接口来达到相同目的，而 C++ 支持多重继承。
* Java 不支持操作符重载，虽然可以对两个 String 对象执行加法运算，但是这是语言内置支持的操作，不属于操作符重载，而 C++ 可以。
* Java 的 goto 是保留字，但是不可用，C++ 可以使用 goto。

## JRE or JDK
* JRE：Java Runtime Environment，Java 运行环境的简称，为 Java 的运行提供了所需的环境。它是一个 JVM 程序，主要包括了 JVM 的标准实现和一些 Java 基本类库。
* JDK：Java Development Kit，Java 开发工具包，提供了 Java 的开发及运行环境。JDK 是 Java 开发的核心，集成了 JRE 以及一些其它的工具，比如编译 Java 源码的编译器 javac 等。
