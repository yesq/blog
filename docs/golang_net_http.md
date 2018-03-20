# Golang `net/http`

整理一下 `net/http` 的简单用法。

## client

```golang
    resp, _ := http.Get("http://example.com/")
    defer resp.Body.Close()
    bs, _ := ioutil.ReadAll(resp.Body)
    log.Println(string(bs))
```

```golang
resp, err := http.Post("http://example.com/upload", "image/jpeg", &buf)

resp, err := http.PostForm("http://example.com/form", url.Values{"key": {"Value"}, "id": {"123"}})
```

如果想更详细的控制请求，需要生成一个 Client 对象。

上边的用法实际上是用了默认的 `var DefaultClient = &Client{}`

```golang
	client := &http.Client{}
	resp, _ := client.Get("http://example.com")
	defer resp.Body.Close()
	bs, _ := ioutil.ReadAll(resp.Body)
    log.Println(string(bs))
```

可以在定义 Client 的时候，自定义一些参数。比如超时时间，下层设置。

或者用 `http.NewRequest` 返回一个 request 。
该 request 在被 `client.Do(req)` 之前，可以自定义一些东西。

## server

ListenAndServe 指定地址和 handler 之后，可以起一个服务器。
handler nil 指使用 DefaultServeMux 。
Handle, HandleFunc 可以向 DefaultServeMux 添加 handlers 。

```golang
    http.Handle("/foo", fooHandler)

    http.HandleFunc("/bar", func(w http.ResponseWriter, r *http.Request) {
        fmt.Fprintf(w, "Hello, %q", html.EscapeString(r.URL.Path))
    })

    log.Fatal(http.ListenAndServe(":8080", nil))
```

自定义 server

```golang
s := &http.Server{
	Addr:           ":8080",
	Handler:        myHandler,
	ReadTimeout:    10 * time.Second,
	WriteTimeout:   10 * time.Second,
	MaxHeaderBytes: 1 << 20,
}
log.Fatal(s.ListenAndServe())
```

TODO: 更复杂的用法。
TODO: 跟框架相比。
