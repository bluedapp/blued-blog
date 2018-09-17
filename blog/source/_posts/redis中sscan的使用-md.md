---
uuid: 5bdb24b0-ba18-11e8-8439-a1e03d77a2c7
author: jiandandkl
email: emolingzhu@126.com
github: https://github.com/jiandandkl
avatar: https://avatars1.githubusercontent.com/u/16009933?v=3
title: redis中sscan的使用.md
date: 2018-09-17 09:24:01
tags: redis
---

> 这周使用smembers获取了一个成员量接近2万的集合,实力坑了自己公司的服务,所以写这篇总结下

redis非常快,官方提供的数据是可以达到100000+的QPS（每秒内查询次数）,是单进程单线程,适合高并发单次数据量小的情况,操作一些大数据量的时候会有性能问题,如keys/smembers/hgetall。


那既然smembers不能用了,要获取某个集合的所有成员怎么办?redis2.8版本增加了scan命令,及相关的sscan/hscan/zscan。

```bash
sadd person jack rose lilei lee
sscan person 0
1) 0
2) 1) jack
   2) rose
   3) lilei
   4) lee
```

* cursor选项(必填)
  游标,从0开始,每次操作后会返回一个新的游标,下次操作已新游标作为游标参数

```bash
sscan mykey 0
0) 423
1) 1) aaa
   2) bbb
   3) ddd
   ...
sscan mykey 423
0) 334
1) 1) fff
   2) ee
   3) gg
   ...
```

* match选项(选填)
  匹配相应的元素

```bash
sscan person 0 match l*
1) 0
2) 1) lilei
   2) lee
```

* count选项(选填, 默认值为10)
  这个就比较奇怪了,有两种情况:
  1. 集合内元素较少,redis会忽略count值,返回所有的元素
  2. 集合内元素较多,设定count后返回等于或大于count值的元素