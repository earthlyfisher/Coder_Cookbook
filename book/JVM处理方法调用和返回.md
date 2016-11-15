## 方法调用

java程序语言提供了两种基本方法：实例方法和类(静态)方法.其不同点是：

1. 实例方法在调用前需要一个对象实例，而类方法不需要.
2. 实例方法使用动态绑定，而类方法使用静态绑定.

JVM用`invokevirtual`和`invokestatic`两种不同的指令来分别处理实例方法和类方法的调用.

## 动态链接

**符号引用** 是一簇唯一标识一个方法，包括类名，方法名，和方法描述(方法返回值，参数类型等)等信息的存放在常量池中的标识.当在运行时，遇到方法调用的指令时，就会解决符号引用.

为了解决符号引用，JVM定位到被符号引用的标识，并将其替换成直接引用.直接引用，比如一个指针或一个偏移量，能够使虚拟机调用方法更快(没有使用过).

## 验证

在将符号引用转成直接引用时，JVM会附加执行一些验证检查操作.这些检查确保java语言规则被遵守并且调用指令能被安全执行.比如，虚拟机首先确保符号引用的方法存在.如果存在，虚拟机检查确保当前类能合法进入到要调用的方法.比如，如果方法是`private`，它必须是当前类的成员.如果有一个检查失败，JVM将抛出异常.

## 对象引用和参数问题

在上面准备好了直接引用后，JVM将要准备参数，实例方法需要有对象引用.这些参数必须在调用指令之前必须通过字节码指令push到调用方法(发起方)的操作数栈中.

## Pushing and poping 栈帧

调用一个方法时，JVM将为被调用方法创建一个新的栈帧.栈帧包括方法的本地变量表、操作数栈和该虚拟机特殊实现需要的其他信息.本地变量表和操作数栈的大小在编译时就已经计算好了.

当方法调用时将在栈中增加一个新的栈帧就叫做`Pushing`一个栈帧.当方法返回时移除一个栈帧叫做`poping`一个栈帧.

## 调用java方法

当调用一个java方法时，JVM将push一个新的栈帧到当前的java栈中.

针对一个实例方法，VM 将pops调用方法栈帧中的对象引用和操作数栈中的参数.JVM创建一个新的栈帧，并且将新的栈帧中本地变量的0(this)替换为对象引用，其他的1、2替换为其他参数.

针对类方法，VM仅仅调用方法栈帧中的操作数栈中的参数，并用他们替换新的栈帧中的本地变量0,1,2....

一旦新的栈帧中的本地变量表替换完成，VM将新的栈帧作为当前栈帧并且设置程序计数器指向新方法的第一条指令.

> The JVM specification does not require a particular implementation for the Java stack. Frames could be allocated individually from a heap, or they could be taken from contiguous memory, or both. If two frames are contiguous, however, the virtual machine can just overlap them such that the top of the operand stack of one frame forms the bottom of the local variables of the next. In this scheme, the virtual machine need not copy objectref and args from one frame to another, because the two frames overlap. The operand stack word containing objectref in the calling method's frame would be the same memory location as local variable 0 of the new frame



## 调用本地方法

如果调用的方法是本地方法,JVM以一种依赖实现的方法来调用。VM不会为本地方法向java栈push一个新的栈帧，At the point at which the thread enters the native method, it leaves the Java stack behind. When the native method returns, the Java stack once again will be used.

## 其他方法调用形式

尽管实例方法一般通过`invokevirtual`来调用，然而`invokespecial`和`invokeinterface`这两个操作码被用来调用某种情形的特殊方法.

`invokespecial`用于基于引用类型的实例化方法.用于三种情形：

1. 调用实力初始化(<init>)方法
2. 调用私有方法
3. 调用使用`super`关键字的方法

`invokeinterface`用来调用一个接口引用的实例方法.

`invokespecial`与`invokevirtual`不同点：

`invokespecial`选择的方法是基于引用类型而不是对象的类类型.或者说，是静态绑定而不是动态绑定。

**invokespecial与私有方法**

```java
class Superclass {
    private void interestingMethod() {
        System.out.println("Superclass's interesting method.");
    }
    void exampleMethod() {
        interestingMethod();
    }
}
class Subclass extends Superclass {
    void interestingMethod() {
        System.out.println("Subclass's interesting method.");
    }
    public static void main(String args[]) {
        Subclass me = new Subclass();
        me.exampleMethod();
    }
}
```

以上将打印:

```
Superclass's interesting method.
```

由于`invokespecial`,VM将选择基于引用类型的方法,所以如此打印.

## 方法调用事例

```java
interface inYourFace {
    void interfaceMethod ();
}
class itsABirdItsAPlaneItsSuperClass implements inYourFace {
    itsABirdItsAPlaneItsSuperClass(int i) {
        super();                    // invokespecial (of an <init>)
    }
    static void classMethod() {
    }
    void instanceMethod() {
    }
    final void finalInstanceMethod() {
    }
    public void interfaceMethod() {
    }
}
class subClass extends itsABirdItsAPlaneItsSuperClass {
    subClass() {
        this(0);                    // invokespecial (of an <init>)
    }
    subClass(int i) {
        super(i);                   // invokespecial (of an <init>)
    }
    private void privateMethod() {
    }
    void instanceMethod() {
    }
    final void anotherFinalInstanceMethod() {
    }
    void exampleInstanceMethod() {
        instanceMethod();             // invokevirtual
        super.instanceMethod();       // invokespecial
        privateMethod();              // invokespecial
        finalInstanceMethod();        // invokevirtual
        anotherFinalInstanceMethod(); // invokevirtual
        interfaceMethod();            // invokevirtual
        classMethod();                // invokestatic
    }
}
class unrelatedClass {
    public static void main(String args[]) {
        subClass sc = new subClass(); // invokespecial (of an <init>)
        subClass.classMethod();       // invokestatic
        sc.classMethod();             // invokestatic
        sc.instanceMethod();          // invokevirtual
        sc.finalInstanceMethod();     // invokevirtual
        sc.interfaceMethod();         // invokevirtual
        inYourFace iyf = sc;
        iyf.interfaceMethod();        // invokeinterface
    }
  ｝
```

## 方法返回

JVM会使用针对每一种返回类型的操作来返回.返回值将从操作数栈pop并且push到调用方法的方法栈帧中.当前的栈帧pop，被调用方法的栈帧变成当前的.程序计数器将重置为调用这个方法的指令的下一条指令.

操作码描述：

1. ireturn none pop int, push onto stack of calling method and return

2. lreturn none pop long, push onto stack of calling method and return

3. freturn none pop float, push onto stack of calling method and return

4. dreturn none pop double, push onto stack of calling method and return

5. areturn none pop object reference, push onto stack of calling method and return

6. return none return void

   The ireturn instruction is used for methods that return int, char, byte, or short.

## 结论

1. 实例方法是动态绑定的,除了`<init>`、private和super关键字调用的方法,这三种特殊情况，实例方法是静态绑定的.
2. 类方法是静态绑定的.
3. 与接口引用相关的实例方法可能比同样的对象关联的方法慢.

## 参考链接

[how the java virtual machine handles method invocation and return](http://www.javaworld.com/article/2076949/learn-java/how-the-java-virtual-machine-handles-method-invocation-and-return.html)









