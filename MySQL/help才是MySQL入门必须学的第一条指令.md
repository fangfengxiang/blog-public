

### help 才是MySQL入门必须学的第一条指令

---



help 才是MySQL入门必须学的第一条指令！！！ 

是的，我一直是怎么认为的。我真正喜欢MySQL也是从这条指令开始的。

基础教程很多，教学文档很多。记住一个又一个的知识点容易，而去寻找未知的答案不容易。



#### help 是什么？

在mysql命令行中，使用help 或者 ? 能够拿到官方文档提示。后面 我就直接写 ?

> [`HELP`](https://dev.mysql.com/doc/refman/8.0/en/help.html)语句从《 MySQL参考手册》中返回在线信息。它的正确操作要求`mysql` 使用帮助主题信息来初始化数据库中的帮助表

也就是这句指令，可以快速帮你找到官方手册中，相关的文档内容。并给出一个官方文档相关模块的链接。

说实在的，很多人遇到MySQL问题，第一时间没有翻手册的习惯。其实不外乎，不知道要去对应那个章节，还有官网手册是英文的。

所以从现在起 养成 遇到问题 执行 ` help 关键字 ` 让MysQL告诉你应该去哪里找答案

例如 你想寻找 联表相关的 知识

- ` ? join `

![image-20200902011740381](https://tva1.sinaimg.cn/large/007S8ZIlgy1gibnn0ejlyj31a00lm77k.jpg)



### 如何利用help来学习

当然如何你说 你不知道该使用那个关键字合适。那么也不用怕，help还支持按模块 一层层搜索。

平时我们也可以利用这个指令，系统地扫两眼，看看MySQL知识该如何分类，有哪些是我们没有看过的，大概看一下简介，了解一下涉及的内容是哪方面。

- ` ? contents ` 列出全部分类

![image-20200902004915484](https://tva1.sinaimg.cn/large/007S8ZIlgy1gibmtiybj6j31c00smwiu.jpg)

- ` ? functions ` 查看函数模块

![image-20200902005927831](https://tva1.sinaimg.cn/large/007S8ZIlgy1gibn40y6aqj31by0jkwho.jpg)

- ` ? Comparison operators` 查看函数下的 操作符 相关内容

![image-20200902010117918](https://tva1.sinaimg.cn/large/007S8ZIlgy1gibn5xtm4oj31b40u077b.jpg)

- ` ? coalesce` 查看操作符 下的 COALESE 具体内容

![image-20200902010455540](https://tva1.sinaimg.cn/large/007S8ZIlgy1gibn9ph1sqj31cq0pq424.jpg)



### 模糊搜索

![image-20200910233052047](https://tva1.sinaimg.cn/large/007S8ZIlgy1gilz4nbkovj312w0hgzmi.jpg)



---



### 推荐阅读

- [MySQL 5.6 Reference Manual/HELP Statement](https://dev.mysql.com/doc/refman/5.6/en/help.html)

- [Getting Help in MySQL Shell](https://mysqlserverteam.com/getting-help-in-mysql-shell/)

- [MySQL 的 help 命令你真的会用吗](https://blog.csdn.net/woqutechteam/article/details/81115892)

  

