1 memcache
===

特点
memcache只能保存字符串的数据类型。

memcache的储存方式是：key => value

memcache的数据是不能持久化的。

memcache的数据是临时的，不需要备份。

memcache的key限度大小：250字节

memcache的value限度大小：1M

memcache 指令
add：添加数据，有key的时候，就返回失败。只能添加不存在的KEY值数据
set：添加数据。key存在不存在都添加。存在就是覆盖。在php里面常用set，不常用add。
incr：是给数字自动添加指定的数字
在memcache里面的数字最大值是2^64 – 1; 最小值是0。也就是猜测负数是用字符串存储?
decr：自动减少指定数量的值
delete：删除
flush_all:清空所有数据
stats：查看服务器的数据情况。关注 stats 和 hits
hits:命中率：
misses：未命中率：
关注它的原因就是要查看缓存内容是否有效，有否被获取到。如果命中率太低了，就证明缓存的数据有问题。需要把缓存数据整理好。重新存放。缓存命中率有70%就可以了

session 入 memcache 
以前的话是 
ini_set('session.sava.handler','memcache');
ini_set('session.save.path','tcp://xxxx:12111;tcp:///')


