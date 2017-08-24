#[译] Go是面向对象的么？
[IS GO OBJECT ORIENTED?](https://flaviocopes.com/golang-is-go-object-oriented/)

有时候我会读到一些文章说“Go是面向对象的”,有时候又会看到一些文章说“Go不能实现面向对象，因为Go里面没有class”

所以我就Go是否面向对象，写下了这篇文章。

对于这个问题的答案，拥有不同开发语言背景的人可能会有不同的理解。比如，如果你之前是一个C程序猿，那么，很明显
Go已经携带了大量面向对象的特性了；而如果你之前是Java程序猿，可能你会觉得Go看上去不那么面向对象。

所以在考虑这个问题的时候，应该放下自己先前对“其他语言”的执念，而从Go本身的思想出发。

基于这样的条件，我得到的答案是：Go是面向对象的，并且是一种超凡脱俗的形式来面向对象。

在[GoFAQ](https://golang.org/doc/faq#Is_Go_an_object-oriented_language)中也有关于这个问题的探讨

> Is Go an object-oriented language?

>Yes and no. Although Go has types and methods and allows an object-oriented style of programming, there is no type hierarchy. The concept of “interface” in Go provides a different approach that we believe is easy to use and in some ways more general. There are also ways to embed types in other types to provide something analogous—but not identical—to subclassing. Moreover, methods in Go are more general than in C++ or Java: they can be defined for any sort of data, even built-in types such as plain, “unboxed” integers. They are not restricted to structs (classes).
>Also, the lack of a type hierarchy makes “objects” in Go feel much more lightweight than in languages such as C++ or Java.

> 【译】: Go是一门面向对象的语言么？
>
> 是也不是。尽管Go有types有methods并且也允许面向对象风格的编程方式，但是它却没有类型继承。Go提供的“interface”提供了一种另类的接口方法，但是我们认为这种方法更简单易懂也更容易理解。另外Go还通过在一个类型里面嵌入另一个类型方式提供一种相似但是又不一样的子类实现方式。然而，Go里面的方法比C++和Java来的更自然：你可以为任何数据类型甚至是内置数据类型定义一个sort的排序算法，而不仅仅只能为structs（classess）定义方法。
>
> 同样的，没有了类型继承，使得Go里面的“objects”比其他语言如C++、Java看上去更轻量一点。

## Cherry-picking 概念
Go吸取了很多其他编程语言的概念，如函数式编程、面向对象编程。并将这些特性都集成了，
这样使得开发者可以随性所欲的组合自己习惯的编程方式。

## 没有Class使用Structs
从传统概念上说，Go没有Class的定义，但是比C里面功能更强的Struct概念。Struct类型配合上
为其定义的方法，可以使得其具备和传统Class同等的功能。Struct通过组织数据结构提供了数据的状态信息，通过定义Struct的方法，赋予了其改变数据状态的行为。

## 封装
Go一个最大的特性就是：通过首字母的大写来表征成员、方法是否是公开的。所有其他的（首
字符非大写的）部分都是包内可见但是不对包外可见的。比较类似其他语言中的public和private，但是没有protected，因为Go中没有继承。

## 没有继承
从GoFAQ中，我们可以了解到Go是没有继承的：

> Object-oriented programming, at least in the best-known languages, involves too much discussion of the relationships between types, relationships that often could be derived automatically. Go takes a different approach.
>
> 【译】：面向对象的编程语言，至少在比较著名的语言中，都大量探讨了两个类型之间的关系，尤其是派生关系。而Go提供了另一种表达形式

## 不同于继承的组合模式

这篇[Composition over inheritance](https://en.wikipedia.org/wiki/Composition_over_inheritance)文章以及Gang of Four的书中（译者注：Go4是《设计模式》书的作者）都有提及的模式，而在Go的代码中会经常看到这样的情形。

当我们定义一个struct的时候，可以定义一个匿名的域，这样就会使得这个域类型的方法会透传
到我们新定义的struct中来。这就是所谓的"struct embedding"：

    package main

    import (
	    "fmt"
    )

    type Dog struct {
	    Animal
	}
		
    type Animal struct {
	    Age int
    }

    func (a *Animal) Move() {
	    fmt.Println("Animal moved")
	}
		
    func (a *Animal) SayAge() {
	    fmt.Printf("Animal age: %d\n", a.Age)
	}
		
    func main() {
	    d := Dog{}
    	d.Age = 3
	    d.Move()
	    d.SayAge()
	}

## 接口
请忘记Java和PHP风格的接口吧，Go同一种完全不同的方式来定义接口，其核心概念就是隐式关联。

GoFAQ中说：

> Rather than requiring the programmer to declare ahead of time that two types are related, in Go a type automatically satisfies any interface that specifies a subset of its methods.
>
> 【译】：不同于需要程序猿提前声明两个类型是相关的。Go中的类型对于实现了声明方法的类型进行隐式关联。

接口定义通常非常简洁，甚至很多都是只有一个方法，在Go的世界里面，很少有一大坨一大坨
方法定义在接口中。

接口很自然的提供了多态的性质：为了实现一个接口，你只要实现其声明过的方法就可以了。

## 方法
类型拥有方法，他们在类型的外面定义。语法定义有点想JavaScript的方法：

    function Person(first, last) {
        this.firstName = first;
        this.lastName = last;
	}
		
    Person.prototype.name = function() {
        return this.firstName + " " + this.lastName;
	};
		
    p = new Person("Flavio", "Copes")
    p.name() // Flavio Copes

在Go中表达是这样的：

    package main

    import (
	    "fmt"
	)
		
    type Person struct {
	    firstName string
    	lastName  string
	}
		
    func (p Person) name() string {
	    return p.firstName + " " + p.lastName
	}
		
    func main() {
	    p := Person{"Flavio", "Copes"}
	    fmt.Println(p.name())
    }

## 为类型增添方法
可以为任何类型增加方法，甚至是基础类型。由于类型的方法只能在其所属的包代码中进行添加
因此我们不能直接给基础类型增加一个方法的定义，当时我们可以先为其重新命个名，然后在为
这个新类型增加方法，使得其在拥有基础类型属性的同时拥有新方法的功能。

    package main

    import (
	    "fmt"
    )

    type Amount int

    func (a *Amount) Add(add Amount) {
	    *a += add
    }

    func main() {
	    var a Amount
	    a = 1
	    a.Add(2)
	    fmt.Println(a)
    }


## 函数

想想在传统的面向对象的语音如Java中，你有多少次为了实现一个静态方法二定义一个"Utils"
类。

这是因为在这些语言中，所有的东西都必须是个对象，因此函数的定义必须要附着在一个对象上
。而这样的情况，在Go上是不存在的。因为Go有函数类型，并不是所有东西都要求是一个对象。
虽然在真实世界中"Classes"和"object"非常重要，但是不是必要的。

在Go的世界里，不是所有东西都是对象（从技术的角度来看，其实没有对象的概念，只是人们
把类型值和变量称为"object"）。和C一样，Go可以在类型的外面定义函数。

所以，Go里面既存在方法也存在函数，并且函数作为"first class"的类型存在（函数可以存
在一个变量中，并可以被当成值进行传递或者从函数或者方法中返回）。


## 结语
总的来说Go的面向对象机制是直接而灵活的。抛弃了类和继承，可以减少大量的模板声明文件。
并且没有了这些需要进行修改的声明后。你可以自由的组合和拆解这些有关系的类型。

