# Making concurrent HTTP requests in Go programming language
  [https://blog.narenarya.in/concurrent-http-in-go.html](https://blog.narenarya.in/concurrent-http-in-go.html)

  貌似没啥写的...

  每个请求开一个 goroutines 。每个请求把响应往一个没有缓冲的 channle 里塞。

  逻辑上就是所有请求同时等待响应。

  谁先拿到响应，谁先把响应塞进 channle 。main 中读取这个响应之前，其他已经拿到响应的 goroutines 等待。