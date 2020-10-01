面试的生涯中，曾经有过这个下面这段问答：
```
Q : MySQL char(10) 是什么意思?
A : 字符串定长类型，最大存储10个字符。
Q : 10个字符? 不是字节吗? 那么最大存储多少个字节?
A : 看字符编码，utf8编码就是30个字节，gbk就是20个字节
Q : 那么在utf8编码下，一个字母占几个字节?
A : 一个字节
Q  : 那char(10) 最大存储30个字节，就应该能存储30个字母，不就是30个字符吗？
A :  额 ............ 内心OS，懵逼树下你和我，难道是我记错了，不会呀，不过如果是字节的话，确实就能说通呀。
```
那么 真相是 怎么样的呢 ?
万事不决，手册来help  
```
help char
```
![image.png](https://upload-images.jianshu.io/upload_images/5261067-22ad021bb568a25a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
手册已经明确说明 char 10 表示的是字符。 对于一个char(10)的字符，最大允许存储的字节是30个字节(utf8) 编码 ，那么对于字母类型，在utf8格式下是存储是占一个字节还是三个字节，如果是一个字节 最大能存储到30个字符吗？
我们通过建一张模拟表来测一下：
```
 CREATE TABLE `test_char_1` (
  `id` int(11) NOT NULL AUTO_INCREMENT COMMENT '主键id',
  `char_str` char(10) DEFAULT '',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB  DEFAULT CHARSET=utf8; 
```

插入
```
insert test_char_1 (char_str) value ("abcdefghij");
insert test_char_1 (char_str) value ("零一二三四五六七八九");
insert test_char_1 (char_str) value ("一a2b3c4d五e");
```
查询
```
select char_str,CHAR_LENGTH(char_str),LENGTH(char_str) from test_char_1;
```
![image.png](https://upload-images.jianshu.io/upload_images/5261067-bd85ec7223561c5e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

使用mysql --column-type-info  进入终端查看字段类型占用信息
- select char_str from test_char_1 where id = 1;
![image.png](https://upload-images.jianshu.io/upload_images/5261067-77d60411eeb7b71e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- select char_str from test_char_1 where id = 2;
![image.png](https://upload-images.jianshu.io/upload_images/5261067-2d6bff7d54ab11a7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- select char_str from test_char_1 where id = 3;
![image.png](https://upload-images.jianshu.io/upload_images/5261067-4abb5c0c091ab0e8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

我们可以看到虽然字母字符10个只占了10个字节，长度最大是30字节，但是最大限制仍旧是以字符来计算的。


![image.png](https://upload-images.jianshu.io/upload_images/5261067-027407aae93d7c29.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

对此 《innodb存储引擎》一书有下面这段说明
> 从MySQL4.1 版本开始 ，char(n) 中的n 指字符长度，不再表示之前版本的字节长度。也就是说在不同字符集下，char类型列的内部存储可能不是定长数据。

> 也就是说多于对字节字符集编码，char类型不再代表固定长度的字符串。对于多字节字符编码的char数据类型的存储，innodb存储引擎在内部将其视为变长字符类型
