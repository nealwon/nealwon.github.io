---
layout: default
title: memcache在telnet下的命令操作
category: linux
excerpt: 记录关于telnet下操作memcached
---

## memcache在telnet下的命令操作

***

* **telnet请求命令格式**

{% highlight shell %}
<command name> <key> <flags> <exptime> <bytes>\r\n <data block>\r\n
{% endhighlight %}

1. <command name> 可以是”set”, “add”, “replace”。
    “set”表示按照相应的<key>存储该数据，没有的时候增加，有的覆盖。
    “add”表示按照相应的<key>添加该数据,但是如果该<key>已经存在则会操作失败。
    “replace”表示按照相应的<key>替换数据,但是如果该<key>不存在则操作失败

2. <key> 客户端需要保存数据的key。

3. <flags> 是一个16位的无符号的整数(以十进制的方式表示)。
    该标志将和需要存储的数据一起存储,并在客户端get数据时返回。
    客户可以将此标志用做特殊用途，此标志对服务器来说是不透明的。

4. <exptime> 过期的时间。
    若为0表示存储的数据永远不过时(但可被服务器算法：LRU 等替换)。
    如果非0(unix时间或者距离此时的秒数),当过期后,服务器可以保证用户得不到该数据(以服务器时间为标准)。

5. <bytes> 需要存储的字节数(不包含最后的”\r\n”),当用户希望存储空数据时,<bytes>可以为0

6. 最后客户端需要加上”\r\n”作为”命令头”的结束标志。
    <data block>\r\n

> 紧接着”命令头”结束之后就要发送数据块(即希望存储的数据内容),最后加上”\r\n”作为此次通讯的结束。

***

* **telnet响应命令**

  结果响应：reply
  当以上数据发送结束之后,服务器将返回一个应答。可能有如下的情况:

1. “STORED\r\n”：表示存储成功
2. “NOT_STORED\r\n” ： 表示存储失败,但是该失败不是由于错误。
   通常这是由于”add”或者”replace”命令本身的要求所引起的,或者该项在删除队列之中。
如： 
{% highlight shell %}
set key 33 0 4\r\n
ffff\r\n 
{% endhighlight %}

***

* **获取/检查KeyValue**

  `get <key>*\r\n`

1. <key>* 表示一个或者多个key(以空格分开)
2. “\r\n” 命令头的结束
结果响应：reply
服务器端将返回0个或者多个的数据项。每个数据项都是由一个文本行和一个数据块组成。当所有的数据项都接收完毕将收到”END\r\n”
每一项的数据结构：

{% highlight shell %}
VALUE <key> <flags> <bytes>\r\n
<data block>\r\n
{% endhighlight %}

> 1. `<key>` 希望得到存储数据的key
> 2. `<flags>` 发送set命令时设置的标志项
> 3. `<bytes>` 发送数据块的长度(不包含”\r\n”)
> 4. “\r\n” 文本行的结束标志
> 5. `<data block>` 希望接收的数据项。
> 6. “\r\n” 接收一个数据项的结束标志。

如果有些key出现在get命令行中但是没有返回相应的数据，这意味着服务器中不存在这些项，这些项过时了，或者被删除了
如：
{% highlight shell %}
get aa
VALUE aa 33 4
ffff
END
{% endhighlight %}

***

* **删除KeyValue**

{% highlight shell %}
delete <key> <time>\r\n
{% endhighlight %}

> 1. `<key>` 需要被删除数据的key
> 2. `<time>` 客户端希望服务器将该数据删除的时间(unix时间或者从现在开始的秒数)
> 3. “\r\n” 命令头的结束

***

* **检查Memcache服务器状态**

{% highlight shell %}
stats\r\n
{% endhighlight %}

在这里可以看到memcache的获取次数，当前连接数，写入次数，已经命中率等；

{% highlight shell %}
STAT pid 53929
STAT uptime 495494
STAT time 1467538425
STAT version 1.4.15
STAT libevent 2.0.21-stable
STAT pointer_size 64
STAT rusage_user 12.697448
STAT rusage_system 8.632060
STAT curr_connections 10
STAT total_connections 1302
STAT connection_structures 12
STAT reserved_fds 20
STAT cmd_get 1299
STAT cmd_set 1498
STAT cmd_flush 3
STAT cmd_touch 0
STAT get_hits 1256
STAT get_misses 43
STAT delete_misses 0
STAT delete_hits 0
STAT incr_misses 0
STAT incr_hits 0
STAT decr_misses 0
STAT decr_hits 0
STAT cas_misses 0
STAT cas_hits 0
STAT cas_badval 0
STAT touch_hits 0
STAT touch_misses 0
STAT auth_cmds 0
STAT auth_errors 0
STAT bytes_read 1269791
STAT bytes_written 886169
STAT limit_maxbytes 67108864
STAT accepting_conns 1
STAT listen_disabled_num 0
STAT threads 4
STAT conn_yields 0
STAT hash_power_level 16
STAT hash_bytes 524288
STAT hash_is_expanding 0
STAT bytes 2586
STAT curr_items 3
STAT total_items 1498
STAT expired_unfetched 20
STAT evicted_unfetched 0
STAT evictions 0
STAT reclaimed 20
END
{% endhighlight %}

***

* **高级缓存细节查看方法**

1. 清空统计数据: `stats reset`
2. 显示内存分配数据: `stats malloc`
3. 显示某个slab中的前limit_num个key列表: `stats cachedump slab_id limit_num`
4. 显示各个slab的信息，包括chunk的大小、数目、使用情况等: `stats slabs`
5. 显示各个slab中item的数目和最老item的年龄(最后一次访问距离现在的秒数): `stats items`
6. 设置或者显示详细操作记录: `stats detail [on|off|dump]`

***

* **清空所有键值**
{% highlight shell %}
flush_all
{% endhighlight %}
>注：flush并不会将items删除，只是将所有的items标记为expired，因此这时memcache依旧占用所有内存。

*** 

* **退出**
{% highlight shell %}
quit\r\n
{% endhighlight %}

***

> 附录Memcached操作命令
<table class="table table-striped table-bordered"><tbody>
<tr><th>命令</th>
<th>描述</th>
<th>示例</th>
</tr>
<tr><td>get</td>
<td>读取键值</td>
<td>get mykey</td>
</tr>
<tr><td>set</td>
<td>设置新键值</td>
<td>set mykey 0 60 5</td>
</tr>
<tr><td>add</td>
<td>新增键值</td>
<td>add newkey 0 60 5</td>
</tr>
<tr><td>replace</td>
<td>替换现有值</td>
<td>replace key 0 60 5</td>
</tr>
<tr><td>append</td>
<td>末尾添加值</td>
<td>append key 0 60 15</td>
</tr>
<tr><td>prepend</td>
<td>头部添加值</td>
<td>prepend key 0 60 15</td>
</tr>
<tr><td>incr</td>
<td>递增数值</td>
<td>incr mykey 2</td>
</tr>
<tr><td>decr</td>
<td>递减数值</td>
<td>decr mykey 5</td>
</tr>
<tr><td>delete</td>
<td>删除键</td>
<td>delete mykey</td>
</tr>
<tr><td rowspan="2">flush_all</td>
<td>清除所有数据</td>
<td>flush_all</td>
</tr>
<tr><td>清除n秒内的数据</td>
<td>flush_all 900</td>
</tr>
<tr><td rowspan="7">stats</td>
<td>打印所有状态信息</td>
<td>stats</td>
</tr>
<tr><td>打印内存信息</td>
<td>stats slabs</td>
</tr>
<tr><td>打印内存信息</td>
<td>stats malloc</td>
</tr>
<tr><td>高级信息</td>
<td>stats items</td>
</tr>
<tr><td></td>
<td>stats detail</td>
</tr>
<tr><td></td>
<td>stats sizes</td>
</tr>
<tr><td>重置状态</td>
<td>stats reset</td>
</tr>
<tr><td>version</td>
<td>打印服务器(memcached)版本</td>
<td>version</td>
</tr>
<tr><td>verbosity</td>
<td>日志级别</td>
<td>verbosity</td>
</tr>
<tr><td>quit</td>
<td>退出telnet控制台</td>
<td>quit</td>
</tr>
</tbody>
</table>

