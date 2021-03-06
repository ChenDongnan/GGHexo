Swift 2.1 函数类型转换：协变与逆变"

> 作者：uraimo，[原文链接](https://www.uraimo.com/2015/09/29/Swift2.1-Function-Types-Conversion-Covariance-Contravariance/)，原文日期：2015-09-29
> 译者：[Lanford3_3](http://lanfordcai.github.io)；校对：[shanks](http://codebuild.me/)；定稿：[Cee](https://github.com/Cee)
  









> 这篇 Swift 2.1 相关的文章需要使用 Xcode 7.1 beta 或者更新的版本， 你可以通过 [GitHub](https://github.com/uraimo/Swift-Playgrounds) 或者是 [zip 文件](https://www.uraimo.com/2015/09/29/Swift2.1-Function-Types-Conversion-Covariance-Contravariance/) 来获取相关 playground 文件。

在即将和 Xcode 7.1 一起到来的 Swift 2.1 中（译者注：原文发表于 2015 年 9 月=，=），函数类型将支持协变与逆变。让我们看看这意味着什么。 



在计算机科学及类型推断的语境中，型变（variance）这个词表示的是，两种类型之间的关系是如何影响他们派生出的复杂类型之间的关系的。复杂类型间的关系，根据原始类型间的关系来看，无外乎不变（invariance）、协变（covariance）与逆变（contravariance）。要高效地使用复杂类型，理解这种派生关系是如何定义的是非常重要的。

我们用伪代码来对此进行阐释。考虑这样一个复杂的参数类型 `List<T>` 和两个简单类型: `Car` 和 `Car` 的一个子类型（subtype） `Maserati`。

我们把已有的两种类型作为 `List<T>` 的 `T` 来获得两种新类型，之后就可以通过讨论新类型的关系，来对不变，协变和逆变加以解释：

* **协变**：如果 `List<Maserati>` 也是 `List<Car>` 的子类型，那么原始类型间的关系也存在于 `List` 中，这是因为，`List` 和他的原始类型是*协变*的。
* **逆变**：但倘若 `List<Car>` 是 `List<Maserati>` 的子类型，那么原始类型间的关系和 `List` 派生出的复杂类型间的关系是相反的，因为 `List` 相对于它的原始类型是*逆变*的。
* **不变**：`List<Car>` 不是 `List<Maserati>` 的子类型，反之亦然，则两种复杂类型间没有衍生关系。

每种语言都采用了一个特定的型变方法集，了解复杂类型间是如何相互联系的，有助于理解两个复杂类型是否兼容，是否能在一些情境下互换，就像是一种类型和他的子类型那样。

在函数类型（Function Types）的语境中，复杂类型的兼容问题可以归结为一个简单的问题：当你需要使用 A 类型的函数时，在什么情况下使用 B 类型的函数进行替代是安全的？

一个通用的规则是，能够兼容的函数类型有这样的特征：其参数是更加泛用的父类型（相比于 A 函数所声明的参数类型，A 的调用者也能够处理更特殊的参数），返回的结果则是一个更加特殊的子类型（A 的调用者会把返回值的类型当成 A 中声明的父类型的简化版）[^1]，参数是逆变的，而返回值是协变的。

> 译者注：如果 `T1` 是 `T2` 的子类型，则可以表示为 `T1 < T2`，那么上面的规则就可以表示为：对函数类型 `F1 = S1 -> T1` 和 `F2 = S2 -> T2` 来说，当且仅当 `S2 < S1` 且 `T1 < T2` 时，`F1` 是 `F2` 的子类型。
> `S1` 和 `S2` 在入参位置上，他们之间的关系和 `F1` 与 `F2` 间的关系是相反的，所以入参是逆变的，同时，`T1` 和 `T2` 在出参位置上，他们之间的关系和 `F1` 与 `F2` 间的关系是相同的，所以出参是协变的。

在 Swift 2.1 前的版本中，函数类型都是不变（invariance）的，如果你在 Playground 中尝试运行下面的代码，你会得到一些类似这样的警告：

    
    
    // Cannot convert value of type '(Int) -> Int' to expected argument type '(Int) -> Any 
    // (无法把 '(Int) -> Int' 转换为期望的参数类型 '(Int) -> Any')
    func testVariance(foo:(Int)->Any){foo(1)}
    
    func innerAnyInt(p1:Any) -> Int{ return 1 }
    func innerAnyAny(p1:Any) -> Any{ return 1 }
    func innerIntInt(p1:Int) -> Int{ return 1 }
    func innerIntAny(p1:Int) -> Any{ return 1 }
    
    testVariance(innerIntAny)
    testVariance(innerAnyInt)
    testVariance(innerAnyAny)
    testVariance(innerIntInt)

在 Swift 2.1 中情况发生了改变，Swift 已经支持函数类型转换，现在参数是逆变的，而返回值是协变的。

回到上面的示例代码，即便 `testVariance` 函数输入参数的类型是 `Int -> Any`，但现在传入 `Any -> Any`、`Any -> Int`、`Int -> Int` 三种类型的函数也都是允许的。 

> 译者注：上述这个 Int 和 Any 的例子其实并不合适，因为 Int 并不是 Any 的子类型。可以参考下面这个例子：
> 
>     
    >class Animal {}
    >class Cat: Animal {}
    
    >func innerAnimalCat(p1: Animal) -> Cat { return Cat() }
    >func innerAnimalAnimal(p1: Animal) -> Animal { return Cat() }
    >func innerCatCat(p1: Cat) -> Cat { return Cat() }
    >func innerCatAnimal(p1: Cat) -> Animal { return Cat() }
    
    >func testVariance(foo: (Cat) -> Animal) { foo(Cat()) }
    
    >testVariance(innerAnimalCat)
    >testVariance(innerAnimalAnimal)
    >testVariance(innerCatCat)
    >testVariance(innerCatAnimal)
    >
> 

说点什么？来 [Twitter](https://www.twitter.com/uraimo) 找我吧~

校者注：关于协变与逆变，还可以参考翻译组翻译的另外一篇文章，解释的更加详细：[Friday Q&A 2015-11-20：协变与逆变](http://swift.gg/2015/12/24/friday-qa-2015-11-20-covariance-and-contravariance/)

---

[^1]: 我并不太理解括号中的内容。对于这段话想表达的意思，举个例子来说明应该是，定义类型 `Animal` 及其子类型 `Cat`，对于函数 `test(catAnimalF: Cat -> Animal)` 中的函数类型 A `catAnimalF: Cat -> Animal` 来说，是可以使用函数 B `animalCatF: Animal -> Cat` 来替换的。因为 `animalCatF` 相较于 `catAnimalF`，其参数类型 `Animal` 是比 `Cat` 更加泛用的父类型，而其返回值类型则更加特殊。但作者在括号内的解释我却没看懂。第一个括号是想说，用 B 替代 A 之后，相较于 A 所声明的参数类型（`Cat`），A 的调用者（`test`）能够处理一个更加特殊的类型？感觉不对诶...第二个括号意思是，在用 B 替代 A 后，其调用者（`test`）会把返回的类型（`Cat`）作为 A 中声明的返回类型（`Animal`）的简化版处理？这个倒好像说的过去囧...（校者注：第一个括号的理解，A 的调用者，也就是函数 `test`，函数类型的入参可以是 `Cat` 的父类，也就是 `Animal`，译者理解是对的，第二个括号理解也是对的。）(定稿注：正好翻译了之前那篇「协变与逆变」，所以译者的理解是正确的。)

> 本文由 SwiftGG 翻译组翻译，已经获得作者翻译授权，最新文章请访问 [http://swift.gg](http://swift.gg)。