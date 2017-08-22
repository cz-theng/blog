#使用Golang的官方mock工具--gomock

在Golang的官方Repo(https://github.com/golang/)中有一个单独的工程叫"mock"(https://github.com/golang/mock),虽然star不是特别多，但它却是Golang官方放出来的mock工具，充这这点我们也需要使用下，虽然并不是官方的就是最好（比如比标准库http更快的fasthttp）。

不同场景mock的对象互相不同，那么gomock主要是mock哪些内容呢？

> mockgen has two modes of operation: source and reflect. Source mode generates mock interfaces from a source file.
> Reflect mode generates mock interfaces by building a program that uses reflection to understand interfaces.

通过gomock的辅助工具我们知道，gomock主要是针对我们go代码中的接口进行mock的。

## 安装

gomock主要包含两个部分：" gomock库"和“ 辅助代码生成工具mockgen”

他们都可以通过`go get`来获取：

    go get github.com/golang/mock/gomock
    go get github.com/golang/mock/mockgen

如何你设置过$GOPATH/bin到你的$PATH变量中，那么这里就可以直接运行mockgen命令了，否则需要使用绝对路径或者相当于$GOPATH的目录。

## 示例
gomock的repo中带了一个官方的例子,但是这个例子过于强大和丰富，反而不适合尝鲜，下面我们写个我们自己的例子(https://www.github.com/cz-it/blog/blog/Go/testing/gomock/example)，一个获取当前Golang最新版本的例子：


    tree .
    .
    ├── go_version.go
    ├── main.go
    └── spider
        └── spider.go

目录结构如上。这里spider.go作为接口文件，定义了spider包的接口：

    package spider

    type Spider interface {
	    GetBody() string
    }

这里假设接口`GetBody`直接可以抓取"https://golang.org"首页的“Build version”字段来得到当前Golang发布出来的版本。

这里在go_version.go中对这个接口进行使用：


    import (
	    "github.com/cz-it/blog/blog/Go/testing/gomock/example/spider"
    )

    func GetGoVersion(s spider.Spider) string {
	    body := s.GetBody()
	    return body
    }

直接返回表示版本的字符串。正常情况下我们会写出如下的单元测试代码：

    func TestGetGoVersion(t *testing.T) {
	    v := GetGoVersion(spider.CreateGoVersionSpider())
	    if v != "go1.8.3" {
		    t.Error("Get wrong version %s", v)
	    }
	}

这里`spider.CreateGoVersionSpider()`返回一个实现了`Spider`接口的用来获得Go版本号的爬虫。

这个单元测试其实既测试了函数GetGoVersion也测试了`spider.CreateGoVersionSpider`返回的对象。
而有时候，我们可能仅仅想测试下`GetGoVersion`函数，或者我们的`spider.CreateGoVersionSpider`爬虫实现还没有写好，那该如何是好呢？

此时Mock工具就显的尤为重要了。

这里首先用gomock提供的mockgen工具生成要mock的接口的实现：

    mockgen -destination spider/mock_spider.go -package spider github.com/cz-it/blog/blog/Go/testing/gomock/example/spider Spider

这里生成了文件:

    └── spider
        ├── mock_spider.go
        └── spider.go


这里注意的是，要预先创建好spider/mocks目录。这样我们的mock代码就生成好了，在"spider/mocks/mock_spider.go"文件中。具体的内容可以先不管。这里先看例子中怎么使用：

    import (
	    "github.com/cz-it/blog/blog/Go/testing/gomock/example/spider"
	    "github.com/golang/mock/gomock"
	    "testing"
    )

    func TestGetGoVersion(t *testing.T) {
	    mockCtl := gomock.NewController(t)
		mockSpider := spider.NewMockSpider(mockCtl)
		mockSpider.EXPECT().GetBody().Return("go1.8.3")
		goVer := GetGoVersion(mockSpider)

        if goVer != "go1.8.3" {
		    t.Error("Get wrong version %s", goVer)
        }
    }

这里在单元测试中再也不用先去实现一个`Spider`接口了，而通过gomock为我们直接生成，然后再集成到我们的单元测试里面。可以看到gomock和testing单元测试框架可以紧密的结合起来工作。

## mockgen工具
在生成mock代码的时候，我们用到了mockgen工具，这个工具是gomock提供的用来为要mock的接口生成实现的。它可以根据给定的接口，来自动生成代码。这里给定接口有两种方式：接口文件和实现文件

### 接口文件
如果有接口文件，则可以通过：

* -source： 指定接口文件
* -destination: 生成的文件名
* -package:生成文件的包名
* -imports: 依赖的需要import的包
* -aux_files:接口文件不止一个文件时附加文件
* -build_flags: 传递给build工具的参数

比如mock代码使用

    mockgen -destination spider/mock_spider.go -package spider -source spider/spider.go

就是将接口spider/spider.go中的接口做实现并存在 spider/mock_spider.go文件中，文件的包名为"spider"。


### 实现文件
在我们的上面的例子中，并没有使用"-source",那是如何实现接口的呢？mockgen还支持通过反射的方式来找到对应的接口。只要在所有选项的最后增加一个包名和里面对应的类型就可以了。其他参数和上面的公用。

### 通过注释指定mockgen

如上所述，如果有多个文件，并且分散在不同的位置，那么我们要生成mock文件的时候，需要对每个文件执行多次mockgen命令（假设包名不相同）。这样在真正操作起来的时候非常繁琐，mockgen还提供了一种通过注释生成mock文件的方式，此时需要借助go的"go generate "工具。

在接口文件的注释里面增加如下：

    //go:generate mockgen -destination mock_spider.go -package spider github.com/cz-it/blog/blog/Go/testing/gomock/example/spider Spider

这样，只要在spider目录下执行

    go generate
命令就可以自动生成mock文件了。
	

## gomock的接口使用

在生成了mock实现代码之后，我们就可以进行正常使用了。这里假设结合testing进行使用（当然你也可考虑使用GoConvey）。我们就可以
在单元测试代码里面首先创建一个mock控制器：

    mockCtl := gomock.NewController(t)

将`* testing.T`传递给gomock生成一个"Controller"对象，该对象控制了整个Mock的过程。在操作完后还需要进行回收，所以一般会在New后面defer一个Finish

    defer mockCtl.Finish()

然后就是调用mock生成代码里面为我们实现的接口对象：

    mockSpider := spider.NewMockSpider(mockCtl)

这里的"spider"是mockgen命令里面传递的报名，后面是`NewMockXxxx`格式的对象创建函数"Xxx"是接口名。这里需要传递控制器对象进去。返回一个接口的实现对象。

有了实现对象，我们就可以调用其断言方法了:`EXPECT()`

这里gomock非常牛的采用了链式调用法，和Swfit以及ObjectiveC里面的Masonry库一样，通过"."连接函数调用，可以像链条一样连接下去。

    mockSpider.EXPECT().GetBody().Return("go1.8.3")

这里的每个"."调用都得到一个"Call"对象，该对象有如下方法：

    func (c *Call) After(preReq *Call) *Call
    func (c *Call) AnyTimes() *Call
    func (c *Call) Do(f interface{}) *Call
    func (c *Call) MaxTimes(n int) *Call
    func (c *Call) MinTimes(n int) *Call
    func (c *Call) Return(rets ...interface{}) *Call
    func (c *Call) SetArg(n int, value interface{}) *Call
    func (c *Call) String() string
    func (c *Call) Times(n int) *Call

这里`EXPECT()`得到实现的对象，然后调用实现对象的接口方法，接口方法返回第一个"Call"对象，
然后对其进行条件约束。

上面约束都可以在文档中或者根据字面意思进行理解，这里列举几个例子：

### 指定返回值
如我们的例子，调用Call的`Return`函数，可以指定接口的返回值：

    mockSpider.EXPECT().GetBody().Return("go1.8.3")

这里我们指定返回接口函数`GetBody()`返回"go1.8.3"。


### 指定执行次数
有时候我们需要指定函数执行多次，比如接受网络请求的函数，计算其执行了多少次。

    mockSpider.EXPECT().Recv().Return(nil).Times(3)

执行三次Recv函数，这里还可以有另外几种限制：

* AnyTimes() ： 0到多次
* MaxTimes(n int) ：最多执行n次，如果没有设置
* MinTimes(n int) ：最少执行n次，如果没有设置

### 指定执行顺序
有时候我们还要指定执行顺序，比如要先执行Init操作，然后才能执行Recv操作。

    initCall := mockSpider.EXPECT().Init()
    mockSpider.EXPECT().Recv().After(initCall)

## 再来回望官方Sample

Sample的结构如下：

    sample/
    ├── README.md
	├── imp1
	│   └── imp1.go
	├── imp2
	│   └── imp2.go
	├── imp3
	│   └── imp3.go
	├── imp4
	│   └── imp4.go
	├── mock_user
	│   └── mock_user.go
	├── user.go
	└── user_test.go

这里，user.go是包含要mock的接口函数的目标文件，而imp1-4是user.go里面接口依赖的文件用来模拟"-imports"和"-aux_files"选项。

user_test.go 文件如同我们的test文件，是对gomock的调用。

而mock_user是生成mock文件的目录。里面的mock_user.go是通过mockgen生成的。

这里我们看到user.go有generate的注释：

    //go:generate mockgen -destination mock_user/mock_user.go github.com/golang/mock/sample Index,Embed,Embedded

这里指定了同一个包里面的三个接口。然后定义了三个接口，里面方法有依赖impx四个目录中的文件：


    type Embed interface {
	    ...
    }

    type Embedded interface {
	    ...
	}

    type Index interface {
        ...
        ForeignOne(imp1.Imp1)
		ForeignTwo(renamed2.Imp2)
		ForeignThree(Imp3)
		ForeignFour(imp_four.Imp4)
		...
    }

以及其他函数。

最后来看调用，在user_test.go中首先创建控制器并调用其Finish函数：

    ctrl := gomock.NewController(t)
    defer ctrl.Finish()

然后就是如上面我介绍的，这里分开在几个不同Test函数中，流程基本上，依次创建mock对象：

    mockIndex := mock_user.NewMockIndex(ctrl)

然后调用其mock的方法：

    mockIndex.EXPECT().Put("a", 1)

    boolc := make(chan bool)
    mockIndex.EXPECT().ConcreteRet().Return(boolc)

最后运行`go test`就可以进行测试了。

    $ go test
    PASS
    ok  	github.com/golang/mock/sample	0.013s
