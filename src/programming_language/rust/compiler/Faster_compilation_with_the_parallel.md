# Faster compilation with the parallel

#### 编译过程

The compiler is split into two halves: the **front-end** and the **back-end**.

The front-end does many things, including：
- parsing / type checking / borrow checking.
- It uses Rayon to perform compilation tasks using **fine-grained parallelism**.

The back-end performs code generation：
- It generates code in chunks called "codegen units" and then LLVM processes these in parallel.
- This is a form of **coarse-grained parallelism**

Rust compilation has long benefited from interprocess parallelism, via Cargo, and from intraprocess parallelism in the back-end. It can now also benefit from **intraprocess parallelism in the front-end**

The compiler uses the jobserver protocol to limit the number of threads it creates.

#### How to use 

```bash
# 安装最新 nightly 版本：rustc 1.75.0-nightly 以上
rustup default nightly 

RUSTFLAGS="-Z threads=8" cargo build --release
```

或增加到配置文件：config.toml:

```toml
[build]
rustflags = ["-Z", "threads=8"]
```

---
- Source：[https://blog.rust-lang.org/2023/11/09/parallel-rustc.html](https://blog.rust-lang.org/2023/11/09/parallel-rustc.html)
- [Samply: Command-line sampling profiler for macOS and Linux](https://github.com/mstange/samply/)
- [Rayon is a data-parallelism library for Rust](https://github.com/rayon-rs/rayon)
- [jobserver protocol](https://www.gnu.org/software/make/manual/html_node/POSIX-Jobserver.html)
- [Amdahl's law](https://en.wikipedia.org/wiki/Amdahl%27s_law)


