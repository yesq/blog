# io.Reader in depth

翻译一篇博客。深入了解 `io.Reader`

[https://medium.com/@matryer/golang-advent-calendar-day-seventeen-io-reader-in-depth-6f744bb4320b](https://medium.com/@matryer/golang-advent-calendar-day-seventeen-io-reader-in-depth-6f744bb4320b)

并不会逐字逐句翻译，基本上算是个笔记...

---

1. `io.Reader` 是什么
1. 常用的哪些对象、类型是 `io.Reader`
1. 如何操作 `io.Reader`
1. 为什么以及如何把 `io.Reader` 放到自己的设计中

## `io.Reader` 是什么

字面上是如下定义的 `interface`。从语义上来说也就是实现了 `Read` 方法的对象。

```golang
type Reader interface {
  Read(p []byte) (n int, err error)
}
```

读到的内容填进 `p []byte`。成功时，n 是成功数量。一个都读不到的时候，返回 EOF。

所以拿到 Reader 就可以统一执行 .Read 动作。

## 哪些常见的对象是 `io.Reader`

```golang
var r io.Reader
var err error
r, err = os.Open("file.txt")
```

`os.Open` 返回的是一个 `os.File` 对象。他实现了 `Read` 方法。
所以他也可以是 `Reader` 。

```golang
var r io.Reader
r = strings.NewReader("aaaa")
```

```golang
var r io.Reader
r = request.Body
```

```golang
var r io.Reader
var buf bytes.Buffer
r = &buf
```

## Readers 都有哪些操作

1. 直接调用 .Read 方法，向 slice 里灌数据。但是要自己控制 buff
1. 用 ioutil.ReadAll 将内容全部读出来。
1. 往另一个 writer 里拷贝
1. 从 reader 里 decode json 对象。
1. 解压缩


```golang
p := make([]byte, 256)
n, err := r.Read(p)
```

一下全读了 

`b, err := ioutil.ReadAll(r)`

拷贝到一个 writer `n, err := io.Copy(w, r)`

json 解码 `err := json.NewDecoder(r).Decode(v)`

解压缩 `r = gzip.NewReader(r)`

## 把 `io.Reader` 放到自己的设计中

比如一个反转字符串的函数。

`func Reverse(s string) (string, error)`

`func Reverse(r io.Reader) io.Reader`

如果用第一种方式。在调用 `Reverse` 的地方，需要判断数据来源。
然后用不同方式，转换成 `string` 。最终会有一坨 if/switch 语句。

如果用第二种方式。传进来的 `Reader` 自己定义好`.Read` 方法。
只管往型参里仍 `Reader` 。代码简洁清晰。

```golang
r = Reverse(strings.NewReader("Make me backwards"))
```

```golang
f, err := os.Open("file.txt")
if err != nil {
  log.Fatalln(err)
}
r = Reverse(f)
```

```golang
func handle(w http.ResponseWriter, r *http.Request) {
  rev := Reverse(r.Body)
  // etc...
}
```