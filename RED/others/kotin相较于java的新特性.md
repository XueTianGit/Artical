

>本文主要看一下kotlin相较于java有哪些新的feature

### 大多数控制结构都是表达式

比如： if/when/try 等等在kotlin中是表达式，而(if)在java中是语句。

>语句和表达式的区别在于，表达式有值，并且能作为另一个表达式的一部分使用；而语句总是包围着它的代码块中的顶层元素，并且没有自己的值。

### 目录和包

在java中，要求把类放到和包结构相匹配的文件与目录结构中。而在kotlin中定义一个包的package可以与当前目录不同，并且可以把多个类放到一个文件中

>kotlin是推荐使用者把一些特别小的有关联的类放到同一个文件中的。但是对于kotlin和java混用的项目，文件最好还是一个一个的按照java目录来组织比较好。

### when

kotlin中的when要比java中的switch强大的多，不仅可以使用代码块作为分支体， 并且在kotlin中when可以“不带参数”:

>如果没有给when表达式提供参数，分支条件就是任意的布尔表达式，这个用来替代多个if十分有效:

    fun test(value1:Int, value2:Int){
        when{
            value1 == 1 ->{ }
            value2 == 2 ->{}
            value1 > 10 && value2 == 0 ->{ }
            else ->{  }
        }
    }

### 异常

kotlin中并不区分受检查异常和未受检查异常，不用指定函数抛出的异常。

>这种设计是基于java中使用异常的实践做出的决定，经验显示java异常规则常常导致许多毫无意义的重新抛出或者忽略异常代码，而且这些规则不能总是保护你免受可能发生的错误。

### 顶层函数与顶层属性

kotlin中顶层函数的设计很大一部分能解决java中各种Utils类的问题，在kotlin我们可以不用创建这些类，直接把这些函数放到一个文件中就可以，在使用时使用import来导入这个函数，和顶层函数一样，属性也可以放到文件中的最顶层。

>其实无论是顶层函数还是顶层属性，在最终都还是被编译为一个静态类的方法或者属性运行在JVM中的。

### 扩展函数

kotlin中的扩展函数就是静态函数的一个高效的语法糖。比如下面这个函数：

    //这个类定义在 StringUitlsKt.kt文件中
    fun String.lastChar():Char = this.get(this.length - 1)

实际上的实现就是，生成一个这个类的静态函数，在调用时把调用对象this，作为函数的第一个参数传入。因此如果我们想在java中使用kotlin的扩展函数可以这样使用:

    char c = StringUtilsKt.lastChar("java");

- 扩展函数是不可以被重写的，没有多态性

>扩展函数并不是类的一部分，它是声明在类之外的。尽管可以给基类和子类都分别定义一个同名的扩展函数，但当这个函数被调用时，它是由该变量的静态类型所决定，而不是这个变量的运行时类型。

### 接口

kotlin中的接口与java8类似，它们可以包含抽象方法的定义以及非抽象方法的实现，但它们不能包含任何状态，比如下面这个接口:

    interface Clickable{
        fun click()
        fun showOff() = printlin("I'm clickable!")
    }

>kotlin是兼容到java6的，但是java6是不支持接口中的默认方法的，因此它会把每个带默认方法的接口编译成一个普通接口和一个将方法体作为静态函数的类的结合体。

### 少继承的思想

我们知道在java中类和方法默认是open的，而kotlin中默认都是final的，即如果你想允许创建一个类的子类，需要使用open修饰符来表示这个类，统一。对于可以被重写的属性或者方法需要添加open修饰符。
    
>脆弱基类的问题：对于继承，对于基类的更改是会导致子类不正确的行为的，因为基类代码的修改不再符合在其子类中的假设，如果类没有提供子类应该怎么实现的明确规则，当事人可能会有按基类作者预期之外的方式来重写方法的风险。

### 内部类

kotlin中的内部类不能访问外部类的实例，即内部类不持有外部类的引用，而在java中内部类就会持有外部类的引用。不过kotlin支持在类声明上添加`inner`,这样内部类就会持有外部类引用。

    class Outer{
        inner class Inner{
            fun getOuterReference():Outer = this@Outer
        }
    }

### 密封类 sealed

如果一个类添加了`sealed`修饰符，那么这个类的子类声明必须嵌套在父类中。(kotlin1.1解除了这个限制)密封类对于`when`表达式有特殊的效果：**如果你在when表达式中处理所有sealed类的子类，你就不再需要提供默认分支，并且如果增加了新的子类，编译带返回值的when表达式时会编译失败**

    sealed class Expr{
        class Num:Expr()
        class Sum:Expr()
    }

### getter 与 setter

我们知道kotlin中的属性，默认实现了`getter`与`setter`，不过我们可以重写他们：

    var address:String = "aa"
        set(value:String){
            field = "bbbb"  //在set方法中可以使用 支持字段`field`来对字段进行赋值
        }

### 数据类

如果为一个累添加了`data`修饰符，则编译器会默认生成 toString、equals、hashCode等方法。equals和hashCode方法会将所有在主构造方法中声明的属性纳入考虑。

### 类委托

为了避免继承的脆弱性，我们有时会使用`装饰模式`来对一个类进行扩展:这种设计模式的本质就是创建一个新类，实现与原始类一样的接口并将原来的类的实例作为一个字段保存。

在kotlin中实现`装饰模式`十分容易，无论实现什么接口，可以使用`by`关键字将接口的实现委托到另一个对象，比如下面这个例子:

class CountingSet<T>(val innerSet:MutableCollection<T> = HashSet()) : MutableCollectio<T> by innerSet

### object关键字

- 对象声明:创建单例对象

object Singleton

与类一样，一个对象声明可以包含属性、方法、初始化语句块等声明，可以实现接口，并且，对象声明在定义的时候就被立即创建了，对象声明不允许有构建方法。

- 伴生对象

在kotlin中不允许拥有静态成员，不过在kotlin中有顶层函数和伴生对象，使用`companion`修饰的对象声明类似于java中的静态方法:

    class A{
        companion object{
            fun bar(){}
        }
    }

    >> A.bar()







