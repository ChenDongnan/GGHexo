每周 Swift 社区问答 2016-02-24"


作者：[shanks](http://codebuild.me)

不知不觉，每周问答已经到了10期了，在stackoverflow上浏览 swift 相关问题，也让我收益不少。这个栏目会一直办下去，慢慢会对一些共性的问题进行分类和整理。本周共整理了 5 个问题。涉及问题有：字典元素的删除；optional存在的理由；可失败构造器的属性初始化问题；数组到字典的转换问题；for循环步长问题。


本周整理问题如下:

* [Swift Dictionary: Can't completely remove entry](#Q1)
* [What advantage of using optional in Swift over nil in Objective C?](#Q2)
* [Using guard to check parameters to inits in Swift 2.2](#Q3)
* [static Dictionary DictionaryGenerator?](#Q4)
* [How can I do a Swift for-in loop with a step? ](#Q5)




对应的代码都放到了 github 上，有兴趣的同学可以下载下来研究：[点击下载](https://github.com/SwiftGGTeam/SwiftCommunityWeeklyQA/tree/master/20160224)

<!--more-->

<a name="Q1"></a>

## Question1: Swift Dictionary: Can't completely remove entry

[Q1链接地址](http://stackoverflow.com/questions/35494051/swift-dictionary-cant-completely-remove-entry)

### 问题描述


以下代码中，并没有按照楼主的想法输出:

    import UIKit
    
    var questions: [[String:Any]] = [
        [
            "question": "What is the capital of Alabama?",
            "answer": "Montgomery"
        ],
        [
            "question": "What is the capital of Alaska?",
            "answer": "Juneau"
        ]
    ]
    
    var ask1 = questions[0]
    var ask2 = ask1["question"]
    
    print(ask2!) // 输出：What is the capital of Alabama?
    
    questions[0].removeAll()
    
    ask1 = questions[0] // ask1 这时候为：[:]
    ask2 = ask1["question"] //ask2=nil  楼主想得到：Should be "What is the capital of Alaska?"

楼主认为使用removeAll是把index=0的元素从字典中删除，所以再次使用index = 0 获取字典的元素时候，就会获取到下一个元素。

### 问题解答

实际上，removeAtIndex才是楼主想要的方法，而 removeAll 仅仅是将当前元素清空为一个空元素。见以下代码：

    questions.removeAtIndex(0) // 删除 index = 0 的元素
    
    ask1 = questions[0] // 得到：["answer": "Juneau", "question": "What is the capital of Alaska?"]
    ask2 = ask1["question"] // 得到："What is the capital of Alaska?"

<a name="Q2"></a>

## Question2: What advantage of using optional in Swift over nil in Objective C?

[Q2链接地址](http://stackoverflow.com/questions/35546224/what-advantage-of-using-optional-in-swift-over-nil-in-objective-c)

### 问题描述

这是一个纯讨论问题，楼主是想直到，Swift 的引入 optional 特性比起 OC 当中的 nil。有啥好处？


### 问题解答


optional 的引入问题，让我们首先来看看事实，就是如何在 Swift 中使用可选类型：
[官方教程](http://wiki.jikexueyuan.com/project/swift/chapter2/01_The_Basics.html#optionals)

关于理解 Swift optional 的用法，请查看这篇文章：[Understanding Optionals in Swift](http://blog.teamtreehouse.com/understanding-optionals-swift),知乎上面有一个翻译的版本：[Swift为什么定义Optional类型？](http://www.zhihu.com/question/26512698)

不过我觉得对 Swift Optional 最好的来源解释，还是在[Functional Swift](https://www.objc.io/books/functional-swift/)一书中的Optional章节。此书较难，需要细细品位。





<a name="Q3"></a>

## Question3: Using guard to check parameters to inits in Swift 2.2

[点击打开问题原地址](http://stackoverflow.com/questions/35545521/using-guard-to-check-parameters-to-inits-in-swift-2-2)

### 问题描述


以下代码，如果注释掉 guard 语句后面的赋值语句，会报错。楼主不理解这是为什么。

    import Foundation
    
    struct Foo{
        
        var url : NSURL
        
        init?(urlString: String){
            
            guard let url = NSURL(string: urlString) else{
                return nil
            }
            
            //self.url = url // 此处注释后， 会报错：return from initializer without initializing all stored properties
        }
    }



### 问题解答


即使是可失败构造器，要么返回nil，要么确保所有存储属性有初始值。而上例中，guard 里面的 url 只是一个临时变量，本身的存储属性没有在init中赋值，所以不能注释掉最后一个语句。

关于可失败构造器，可以查看[官方文档](http://wiki.jikexueyuan.com/project/swift/chapter2/14_Initialization.html#failable_initializers)




### Question4: static Dictionary DictionaryGenerator?

[点击打开问题原地址](https://forums.developer.apple.com/thread/38694)
### 问题描述


楼主想要一一对应2个数组，生成一个新的字典，于是出现以下代码：

    enum ItemKind : Int {
        case    All
        case    Mine
        case    InUse
        case    Archive
        
        
        static let all = [ItemKind.All, ItemKind.Mine, ItemKind.InUse, ItemKind.Archive]
        static let strings = ["All", "Mine", "In Use", "Archive"]
        static let values:[String:ItemKind] = [
            strings[ItemKind.All.rawValue]: ItemKind.All
            ,   strings[ItemKind.Mine.rawValue]: ItemKind.Mine
            ,   strings[ItemKind.InUse.rawValue]: ItemKind.InUse
            ,   strings[ItemKind.Archive.rawValue]: ItemKind.Archive
        ]
        
        
        var description: String {
            return ItemKind.strings[self.rawValue]
        }  
    }

这样的写法，楼主感觉有点繁琐，不好扩展，于是想通过 map 来实现功能，但是使用map时候出现编译错误：


    enum ItemKind1 : Int {
        case    All
        case    Mine
        case    InUse
        case    Archive
        
        
        static let all = [ItemKind.All, ItemKind.Mine, ItemKind.InUse, ItemKind.Archive]
        static let strings = ["All", "Mine", "In Use", "Archive"]
        static let values:[String:ItemKind] = all.map { //报错
            /// ???
            return ($0.description, $0)
            /// ??? or something
            
        }
        
        
        var description: String {
            return ItemKind.strings[self.rawValue]
        }
    }

### 问题解答


事实上，map是从一个数组，变换为另外一个数组，而不能由数组变换为一个字典。正确做法有2种，第一种是用forEach来对目标字典进行赋值。见下面的代码：


    enum ItemKind2 : Int {
        case    All
        case    Mine
        case    InUse
        case    Archive
        
        static let all = [ItemKind.All, ItemKind.Mine, ItemKind.InUse, ItemKind.Archive]
        static let values:[String:ItemKind] = {
            var result: [String: ItemKind] = [:]
            all.forEach {
                result[$0.description] = $0
            }
            return result
        }()
        
        var description: String {
            return String(self)
        }
    }
    print(ItemKind2.values)


另外一种做法，既然数组不支持到字典的转换，那我们实现一个即可：


    extension Array {
        func mapThatReturnsADictionary<KeyType:Hashable>(lambdaThatCreatesOneDictEntryFrom:(Element)->(KeyType,Element))->[KeyType:Element] {
            var result: [KeyType:Element] = [:]
            self.forEach {elt in
                let (key,value) = lambdaThatCreatesOneDictEntryFrom(elt)
                result[key] = value
            }
            return result
        }
    }
    enum ItemKind4 : Int {
        case    All
        case    Mine
        case    InUse
        case    Archive
        
        static let all: [ItemKind] = [.All, .Mine, .InUse, .Archive]
        static let values: [String: ItemKind] = all.mapThatReturnsADictionary{($0.description,$0)}
        
        var description: String {
            return String(self)
        }
    }
    
    print(ItemKind4.values)



<a name="Q5"></a>

### Question5: How can I do a Swift for-in loop with a step? 

[点击打开问题原地址](http://stackoverflow.com/questions/35556850/how-can-i-do-a-swift-for-in-loop-with-a-step)

### 问题描述


楼主的问题是：Swift 中的 for 循环，如何按照一个特定的步长递增或者递减，比如 c 语言中可以这样做：

    for (i = 1; i < max; i+=2) {
        // Do something
    }

### 问题解答

需要配合方法stride来设置步长，有以下几种写法：


    //(i = 1; i < 10; i+=2)
    for index in 10.stride(to: 10, by: 2) {
        print(index)
    }
    
    1.stride(to: 10, by: 2).forEach { i in
        print(i)
    }
    //(i = 1; i <= 10; i+=2)
    for i in 1.stride(through: 10, by: 2) {
        print(i)
    }