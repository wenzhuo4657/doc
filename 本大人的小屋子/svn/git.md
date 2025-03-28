## 基本概念

[Git - Git 是什么？](https://git-scm.com/book/zh/v2/%e8%b5%b7%e6%ad%a5-Git-%e6%98%af%e4%bb%80%e4%b9%88%ef%bc%9f)

传统的版本控制会存储**文件差异**，而git舍弃了这一点，根据文件状态或者说**文件本身的链接**去尝试控制版本。
这么做的好处显而易见，如果存储文件差异，则不可避免的需要从文件内部开始操作，所带来的是复杂且绝不能出错的高安全性要求，而git管理文件链接，换言之，他只需要保存每个版本的文件链接列表即可，对于真实的文件数据，他们互相隔离。
而坏处也同样清晰，即文件系统容易变得臃肿庞大。


因此我们可以称git为快照存储数据库，他将每次提交时的数据都会记录在仓库中。那么存储在何处？


Git 是一个内容寻址文件系统，听起来很酷。但这是什么意思呢？ 这意味着，Git 的核心部分是一个简单的键值对数据库（key-value data store）。 你可以向 Git 仓库中插入任意类型的内容，它会返回一个唯一的键，通过该键可以在任意时刻再次取回该内容。

具体来说则是在.git/objects目录下。



# 命令行


[git rev-parse (Plumbing Commands) - Git 中文开发手册 - 开发者手册 - 腾讯云开发者社区-腾讯云](https://cloud.tencent.com/developer/section/1138781)


git rev-parse --verify HEAD： 验证只提供了一个参数，并且可以将其转换为可用于访问对象数据库的原始20字节SHA-1。如果是这样，将其发送到标准输出; 否则，出错


