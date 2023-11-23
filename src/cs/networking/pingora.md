# Pingora：Cloudflare 使用 Rust 构建的 HTTP 代理

### [Nginx](https://www.nginx.com) 的核心架构

- [worker 进程架构](https://www.nginx.com/blog/inside-nginx-how-we-designed-for-performance-scale/)
- [Load Balancing with NGINX](https://www.nginx.com/blog/load-balancing-with-nginx-plus/)
- [How We Designed for Performance & Scale](https://www.nginx.com/blog/inside-nginx-how-we-designed-for-performance-scale/#Inside-the-NGINX-Worker-Process)

### Nginx 带来的生成环境问题

- [Nginx 的请求处理方式导致负载不均衡](https://blog.cloudflare.com/the-sad-state-of-linux-socket-balancing/)
- [Keepalives considered harmful](https://blog.cloudflare.com/keepalives-considered-harmful/)
- [Nginx 导致的生产问题：CPU 繁重场景](https://blog.cloudflare.com/the-problem-with-event-loops/)
- [Nginx 导致的生产问题：阻止 IO 任务](https://blog.cloudflare.com/how-we-scaled-nginx-and-saved-the-world-54-years-every-day/)

最关键的问题是糟糕的连接重用。我们的机器与原始服务器建立 TCP 连接，以代理 HTTP 请求。连接重用通过重用之前从连接池建立的连接，跳过新连接所需的 TCP 和 TLS 握手，来加快请求的 TTFB（首字节时间）。

但是，NGINX 连接池与单个 worker 相对应。当请求到达某个 worker 时，它只能重用该 worker 内的连接。当我们添加更多 NGINX worker 以进行扩展时，我们的连接重用率会变得更差，因为连接分散在所有进程的更多孤立的池中。这导致更慢的 TTFB 以及需要维护更多连接，进而消耗我们和客户的资源（和金钱）。

### Cloudflare 做的连接优化及 Pingora 的优势

- http2 优化： https://blog.cloudflare.com/delivering-http-2-upload-speed-improvements/
- QUIC: https://blog.cloudflare.com/the-road-to-quic/

Pingora 的设计决策：
- 选择用 Rust 为项目语言，不影响性能的情况下以内存安全的方式完成 C 语言可以做的事情
- 工作负载调度系统： 选择多线程而不是多处理，以便轻松共享资源，尤其是连接池。Work-Stealing, [Tokio 异步运行时](https://tokio.rs/blog/2019-10-scheduler)
- 跨所有线程共享连接: 意味着更好的连接重用率，在 TCP 和 TLS 握手上花费的时间更少。
- 添加 HTTP/2 上游支持, gPRC, 及更多功能
- 消耗的 CPU 和内存大幅度减少（减少了新的连接。与仅通过已建立的连接发送和接收数据相比，TLS 握手成本显然更为高昂。）
- Rust 的内存安全语义保护我们免受未定义行为的影响，并让我们相信我们的服务将正确运行。


> Source：[Pingora 的构建方式](https://blog.cloudflare.com/zh-cn/how-we-built-pingora-the-proxy-that-connects-cloudflare-to-the-internet-zh-cn/)