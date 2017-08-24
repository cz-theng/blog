#KSUID

![asset_ZMiKWCG5](./images/asset_ZMiKWCG5.png)

一个生成UUID并提供K-Sort功能的库

## KSUID官方介绍
KSUID是一个提供K-Sort排序功能的UUID库。其生成UUID的方式遵从[RFC 4122 ](https://www.ietf.org/rfc/rfc4122.txt)定义
的第四代UUID计算方式。同时为了方便按照其生成的时间先后进行排序，故加入了
时间戳信息。

KSUID生成了一段20字节的字符串。里面包含了32位的UTC时间戳和128位的随机数，时间戳采用了大端序（所以可以把这段解压出来，做时间排序）。字符串采用27个字母数字的base62编码.

## KSUID的设计初衷

1. 按照时间排序
KSUID生成的字符串中包含了一段时间戳信息，这样对于需要按照生产时间排序的需求如Twitter，可以很方便的使用。

2. 独立化
支持K-Sort的[Snowflak](https://blog.twitter.com/engineering/en_us/a/2010/announcing-snowflake.html)以及衍生物如[Flak](https://github.com/boundary/flake)都需要一定的协调工具，使得创建过程显得过重。RFC4122的UUIDv4虽然包含了时间戳，但是容易造成重复。KSUID使用128位的随机数，可以比RFC4122的UUIDv4高出64倍，并且因为加入了时间戳（另外的位，加随机数总共20byte），更不容易重复同时基本秒速创建。

3. 排序灵活易于移植
KSUID生成的UUID就是一个字符串，所以可以灵活的运用到各种K-Sort排序算法中，尤其是根据其时间戳特性的改造会大大提高排序效率。

另外，这些字符串是Base62编码的，所以可以对整体用Base62的数来表达，这样就变成了传统的整形数排序了。

## KSUID的使用
KSUID提供了一个cmd工具"ksuid"其实现在"https://github.com/segmentio/ksuid/blob/master/cmd/ksuid/main.go" 接口使用也可以参考这个例子。

执行:

    go get github.com/segmentio/ksuid/blob/master/cmd/ksuid

安装ksuid工具。然后执行

    ksuid
	0s8vez5ljdh6IptV1hVzBk1Jvda
	ksuid
	0s8vhBYOFZgpJp2yYnvpkS5t350
	ksuid
	0s8vhUQI58Ml97lGs2fYvfyxTwy
	ksuid
	0s8vhaZEtJ0EYrUZg3sNTDDzLye

这里会发现生成了一些UUID,并且因为时间戳相近，他们都有类似的前缀。

最简单的使用：

    uid := ksuid.New()
    fmt.Printf("UID:%s\n", uid.String())

这里New生成一个KSUID对象，然后调用其String进行Base62编码，得到一个字符串。

当然他还有其他方法比如：

* Time：获得UUID中的时间部分
* Timestamp： 获得UUID中的时间戳部分
* Payload： 获得随机数部分
* Bytes：获得20byte的数组
* String: 获得Base62的字符串

详细介绍可以参考[GoDoc](https://godoc.org/github.com/segmentio/ksuid)

## 参考
1. [ksuid](https://github.com/segmentio/ksuid)
2. [A Brief History of the UUID](https://segment.com/blog/a-brief-history-of-the-uuid/)
3. [ksuid godoc](https://godoc.org/github.com/segmentio/ksuid)
