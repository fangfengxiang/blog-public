四、异步IO
===

### swoole 定时器
常规定时器：crontab -> 分时日月周 
swoole 定时器 -> 毫秒级定时器

swoole定时器：
- swoole_timer_tick
每隔多少毫秒触发，持续触发，知道被swoole_timer_clear清除
- swoole_timer_after
只触发一次

### 异步文件IO

swoole支持两种异步文件IO读写
-   Linux原生异步IO (AIO模式：SWOOLE_AIO_LINUX)
-   线程池模式异步IO (AIO模式： SWOOLE_AIO_BASE)

### 异步MySQL
###  异步Redis