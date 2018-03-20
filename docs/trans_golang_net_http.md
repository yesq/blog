# Don’t use Go’s default HTTP client (in production)

翻译/笔记 [https://medium.com/@nate510/don-t-use-go-s-default-http-client-4804cb19f779](https://medium.com/@nate510/don-t-use-go-s-default-http-client-4804cb19f779)

只用简单的 `http.Get` 就可以发送请求得到相应。但是并没有设置超时时间。当服务端挂起你的请求，你的程序将可能出现问题。

证明

```golang 
  svr := httptest.NewServer(http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
    time.Sleep(time.Hour)
  }))
  defer svr.Close()
  fmt.Println(“making request”)
  http.Get(svr.URL)
  fmt.Println(“finished request”)
```

服务端响应之前等待 1 小时。用默认 Client 发送的请求会一直等下去。

简单的解决方案。

```golang
	var netClient = &http.Client{
		Timeout: time.Second * 10,
    }
```

如果想更细粒度的控制整个请求生命周期。
可以自定义 `net.Transport`, `net.Dialer`。

```golang
    var netTransport = &http.Transport{
    Dial: (&net.Dialer{
        Timeout: 5 * time.Second,
    }).Dial,
    TLSHandshakeTimeout: 5 * time.Second,
    }
    var netClient = &http.Client{
    Timeout: time.Second * 10,
    Transport: netTransport,
    }
    response, _ := netClient.Get(url)
```