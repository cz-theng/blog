#使用Golang的官方mock工具--gomock

在Golang的官方Repo(https://github.com/golang/)中有一个单独的工程叫"mock"(https://github.com/golang/mock),虽然star不是特别多，但它却是Golang官方放出来的mock工具，充这这点我们也需要使用下，虽然并不是官方的就是最好（比如比标准库http更快的fasthttp）。

不同场景mock的对象互相不同，那么gomock主要是mock哪些内容呢？

> mockgen has two modes of operation: source and reflect. Source mode generates mock interfaces from a source file.
> Reflect mode generates mock interfaces by building a program that uses reflection to understand interfaces.

通过gomock的辅助工具我们知道，gomock主要是针对我们go代码中的接口进行mock的。

gomock主要包含两个部分：

* gomock库
* 辅助代码生成工具
