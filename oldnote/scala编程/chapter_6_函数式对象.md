---
title: 第六章 函数式对象
categories: scala   
tags: [scala,scala编程]
---



# 1.类ration的规格说明书
```
有理数是一种可以表达式比率 n/d 的数字,这里n和d是数字,其中d不能为零,n被称为分子,d被称为分母

```


<!--more-->


# 2.创建rational
```
class Rational(n:Int, d:Int)
/*
这行代码里首先应当注意到的是如果类没有主体,就不需要指定一对空的花括号(当然,如果你想指定也是可以的),在类名Rational之后的括号里的n和d,被称为类参数,scala编译器会收集这两个参数并创造出同样的两个参数的主构造器


Java类具有可以带参数的构造器,而scala类可以直接带参数,scala的写法更简洁,类参数可以直接在类的主体中使用
没有必要定义字段然后写赋值函数把构造器的参数复制到字段里,这里无形省略了很多固定写法,尤其是对小类

scala编译器将把类内部的任何既不是字段也不是方法定义的代码编译至主机构造器中,例如:
*/
class Rational(n:Int, d:Int) {
  println("Create "+n+"/"+d)    //主构造器中的内容
}
/*
scala编译器把这段代码的println调用放进Rational的主构造器,因此,println调用将在每次创建新的Rational实例时打印这条信息
*/


```

 不可变对象的权衡

```
/*
不可变对象提供了若干可变对象的优点和一个潜在的缺点
1.不可变对象常常比可变对象更易理清头绪,因为他们没有回随着时间变化的复杂的状态空间
2.其次,你可以很自由的传递不可变对象,但是对于可变对象而言,传递给其他代码之前,需要先建造一个以防万一的副本
3.一旦不可变对象完成构造之后,就不会有线程因为并发访问而破坏对象内部状态,因为根本没有线程可以改变不可变对象的状态
4.不可变对象让哈希表键值更安全,比方说,如果可变对象在进入HashSet之后被改变,那么你下一次查找这个HashSet时就找不到这个对象了

#缺点
有时需要复制很大的对象表而可变对象的更新可以在原址发生,有些情况下这会变成难以快速完成而可能产生性能瓶颈
*/

```

# 3.重新实现toString方法
```
class Rational(n:Int, d:Int) {
  override def toString = n + "/" + d //override修饰符说明这是对原有方法定义的重载
}

```


# 4.检查先决条件
```
//确保分母不能为零,两种做法:
//方式一
class Rational(n:Int, d:Int) {
  require(d != 0)        //require方法定义在对象Predef中
  override def toString = n + "/" + d 
}
/*
require方法带有一个布尔型参数,如果传入的值为真,require将正常返回,反之,require将抛出llegalArgumentException阻止对象被构造
*/
```


# 5.添加字段
```
class Rational(n:Int, d:Int) {
  require(d != 0)
  override def toString = n + "/" + d
  def add(that: Rational): Rational ={
    new Rational(n*that.d + that.n*d, d*that.d) 
    //n和d是构造器中的n和d,add方法中是拿不到该属性的,但是对象可以拿到
  }
}

/*尽管类参数n和d都在add代码可引用的范围内,但是add方法仅能访问调用对象自身的值,并不能访问that对象的d和n,因为that并不是调用add的Rational对象,所以上述代码回报下面的异常:
*/
Error:(5, 25) value d is not a member of Rational
    new Rational(n*that.d + that.n*d, d*that.d)
Error:(5, 46) value d is not a member of Rational
    new Rational(n*that.d + that.n*d, d*that.d)


/*如果想要访问that的d和n,需要把他们放在字段中,如下*/
class Rational(n:Int, d:Int) {
  require(d != 0)
  val number = n
  val denom = d

  override def toString = number + "/" + denom
  def add(that: Rational): Rational ={
    new Rational(n*that.denom + that.number*d, d*that.denom)
  }
}

/*
尽管n和d在类范围内有效,但因为他们只是构造器的一部分,所以scala编译器不会为他们自动构造字段,所以我们要手动添加字段(即number和denom)
*/

/*
我们之前不能在对象外部直接访问有理数的分子和分母,现在可以了,只要访问公共的number,denom字段即可
*/
val r = new Rational(1, 2)
r.number
r.denom

```

# 6.自指向
```
/*
关键字this指向当前执行方法被调用的对象实例
*/
//下面的方法测试有理数是否小于传入的参数
def lessThan(that:Rational) = {
  this.number * that.denom < that.number * this.denom
}

//下面的方法返回有理数和参数中的较大者
def max(that: Rational) = {
  if (this.lessThan(that)) that else this    //如果返回的是this,表示返回的是当前对象
}
```

# 7.辅助构造器
```
/*
有时候一个类里需要多个构造器,scala里主构造器之外的构造器被称为辅助构造器,比如:分母为1的有理数只写分子的话就更为简洁,写成这样Rational(5), 这就需要给Rational添加只传分子的辅助构造器并预设分母为1,如下:
*/

class Rational(n:Int, d:Int) {
  require(d != 0)
  val number = n
  val denom = d

  def this(n:Int) = this(n,1)    //辅助构造器

  override def toString = number + "/" + denom
  def add(that: Rational): Rational ={
    new Rational(n*that.denom + that.number*d, d*that.denom)
  }
  def lessThan(that:Rational) = {
    this.number * that.denom < that.number * this.denom
  }
  def max(that: Rational) = {
    if (this.lessThan(that)) that else this
  }
}
/*
scala里的每个辅助构造器的第一个动作都是调用同类的别的构造器,换句话说,每个scala类的每个辅助构造器都是以 this(...) 形式开头的,被调用的构造器就可以是主构造器,也可以是源文件中早于调用构造器定义的其他辅助构造器,规则的根本结果就是每个scala的构造器调用最终结束于对主构造器的调用,因此主构造器是类的唯一入口点

scala的类里面只有主构造器可以调用超类的构造器
*/

```

# 8.私有字段和方法
```
class Rational(n:Int, d:Int) {
  require(d != 0)
  private val g = gcd(n.abs, d.abs)
  val number = n / g
  val denom = d / g

  def this(n:Int) = this(n,1)

  //计算传入的两个Int的最大公约数
  private def gcd(a: Int, b: Int):Int = {
    if (b==0) a else gcd(b, a%b)
  }
}

```

# 9.定义操作符
```
/*
有理数(分数)的加法写成如下的形式:
x + y
而不用写成:
x.add(y)
*/

class Rational(n:Int, d:Int) {
  require(d != 0)
  private val g = gcd(n.abs, d.abs)
  val number = n / g
  val denom = d / g

  def this(n:Int) = this(n,1)

  //计算传入的两个Int的最大公约数
  private def gcd(a: Int, b: Int):Int = {
    if (b==0) a else gcd(b, a%b)
  }

  def +(that: Rational): Rational = {
    new Rational(
      number * that.denom + that.number * denom,
      denom * that.denom
    )
  }

  def *(that: Rational): Rational = {
    new Rational(number*that.number,denom * that.denom)
  }

  override def toString = number + "/" + denom
}

/*测试*/
object Rational{
  def main(args: Array[String]): Unit = {
    val x = new Rational(3,4)
    val y = new Rational(1,2)
    val z = x + y
    //也可以写成
    x.+(y)//不过这样写可读性不佳

    //按照scala的优先级规则,Rational的*方法比+方法优先级更高
    x + x * y
    //等同于
    x + (x * y)
  }
}
```


# 10.scala的标识符和命名规范
```
#字母数字标识符:以字母或下划线开始,之后可以跟字母,数字,或下划线
/*
scala遵循Java的驼峰式标识符习惯,例如:toString和HashSet,,尽管下划线在标识符内是合法的,但是scala程序里并不常用,部分原因是为了保持与Java一致,同样也由于下划线在scala代码里有许多其他非标识符用法,因此,最好避免使用像to_string, _init_ 这样的标识符
字段/方法参数/本地变量/还有函数的驼峰式名称,应该以小写字母开始,如:length, flatMap
类和特质的驼峰式名称应该以大写字母开始,如:BigInt, List 

scala与Java的习惯不一致的地方在于常量名,scala里constant这个词并不等同于val,尽管val被初始化之后的确保持不变,但他仍然是变量

在Java里,习惯上常量名称全都是大写的,用下划线分割单词,如:MAX_VALUE或PI,在scala里,习惯只是第一个字母必须大写,因此Java风格的常量名在scala里也可以用,但是scala的习惯是常数也用驼峰式风格,如:XOffset


操作符标识符由一个或多个操作符字符组成,操作符字符是一些+, :, ?, ~, # 

混合标识符:由字母数字组成,后面跟着下划线和一个操作符标识,如: unary_+  被用作定义一元的"+" 操作符的方法名,或 myvar_= 被用作定义赋值操作符的方法名,

字面量标识符,使用反引号 `....` 包括的任意字符串,如:
`x`         `<clinit>`        `yield`
在Java的Thread类中访问静态的yield方法是他的典型用例,你不能写Thread.yield() ,因为yield是scala的保留字,然而可以在反引号里引用方法的名称,例如:Thread.`yield`()

*/

```
# 11.方法 重载
```
/*
上述 * 的操作符数只能是有理数,所以对于有理数r不能写r * 2 ,只能写成r* new Rational(2) ,这样写很不美观,为了让Rational用起来方便,可以在类上增加能够执行有理数和整数之间的加法和乘法的新方法
*/

class Rational(n:Int, d:Int) {
  require(d != 0)
  private val g = gcd(n.abs, d.abs)
  val number = n / g
  val denom = d / g

  def this(n:Int) = this(n,1)

  //计算传入的两个Int的最大公约数
  private def gcd(a: Int, b: Int):Int = {
    if (b==0) a else gcd(b, a%b)
  }

  def +(that: Rational): Rational = {
    new Rational(
      number * that.denom + that.number * denom,
      denom * that.denom
    )
  }

  def + (i: Int): Rational = {
    new Rational(number + i*denom, denom)
  }

  def *(that: Rational): Rational = {
    new Rational(number*that.number,denom * that.denom)
  }

  def * (i: Int):Rational = {
    new Rational(number * i, denom)
  }
  override def toString = number + "/" + denom
}

```

# 12.隐式转换
```
/*
上面的做法可以对  r*2 进行计算了,但是 2 * r 还是不能进行,因为2*r等同于2.*(r),因此这是在整数2上的方法调用,但Int类没有带Rational参数的乘法
不过scala有另外的方法解决这个问题,可以创建在需要的时候自动把整数转换为有理数的隐式转换,如下:
*/
implicit def intToTational(x: Int) = new Rational(x)
/*
上述代码定义了从Int到Rational的转换方法,方法前面的implicit修饰符告诉编译器可以在一些情况下自动调用
*/
val x = new Rational(3,4)
val z = 2 * x

```


