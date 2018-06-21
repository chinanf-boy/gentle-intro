## 温柔的介绍

对 `rust` 的 温柔 介绍 「翻译 」

## 使用 

用 mdbook  - https://github.com/rust-lang-nursery/mdBook 提供 `md`网页

```
mdbook serve
```

> 服务器查看 http://localhost:3000/

### SUMMARY.md 

[SUMMARY.md](./src/SUMMARY.md)是 书的目录, 注意 md文件路径命名不能错

<details>

<summary> 目录 内容</summary>

``` m
# 概要

[介绍](./readme.zh.md)

-   [基本](./1-basics.zh.md)
-   [结构,枚举和匹配](./2-structs-enums-lifetimes.zh.md)
-   [文件系统和进程](./3-filesystem.zh.md)
-   [模块和货物](./4-modules.zh.md)
-   [标准库容器](./5-stdlib-containers.zh.md)
-   [错误处理](./6-error-handling.zh.md)
-   [线程,网络和共享](./7-shared-and-networking.zh.md)
-   [面向对象编程](./object-orientation.zh.md)
-   [用nom解析](./nom-intro.zh.md)
-   [痛点](./pain-points.zh.md)

```

</details>


## 校对

> `src/` 下 `所有 *.zh.md` 文件是翻译, 还没校对


- [x] [1.基本](./src/1-basics.zh.md)
- [x] [2.结构,枚举和匹配](./src/2-structs-enums-lifetimes.zh.md)
- [x] [3.文件系统和进程](./src/3-filesystem.zh.md)
- [x] [4.模块和货物](./src/4-modules.zh.md)
- [x] [5.标准库容器](./src/5-stdlib-containers.zh.md)
- [x] [6.错误处理](./src/6-error-handling.zh.md)
- [ ] [7.线程,网络和共享](./src/7-shared-and-networking.zh.md)
- [ ] [8.面向对象编程](./src/object-orientation.zh.md)
- [ ] [9.用nom解析](./src/nom-intro.zh.md)
- [ ] [10.痛点](./src/pain-points.zh.md)

> // ...


## 其他

- translate-mds >> https://github.com/chinanf-boy/translate-mds 提供初稿
