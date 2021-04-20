
服务器时延统计工具，在原有功能基础上，增加时延阈值统计，记录超过阈值的包个数，并把数据包时间戳记录到日志文件，这样可以根据时间戳快速定位到抓包文件中对应的包，从而可以快速定位到大时延包，避免了人肉搜索抓包文件，提高问题排查效率，所有参数可配置。
 
可以以在线方式和离线方式分析redis、memcached、mysql、nginx等请求应答系统的时延，并定位慢响应数据报文。

（forked from [y123456yz/tcprstat](https://github.com/y123456yz/tcprstat)）

-------------------------

## 安装

```
git clone https://github.com/Damanchen/tcprstat.git
cd tcprstat
yum install perf patch bison flex glibc-static libcap-devel yacc
(No package yacc available. CentOS7 不影响，yacc和bison都是做词法和语法的分析)
autoreconf -fvi
./configure
make & make install
```
#### 编译说明
依赖pthread线程静态库

#### 异常处理
如果出现如下异常，一般是系统netlink库引起的，卸载掉libnl即可

```
../libpcap/libpcap-1.1.1//libpcap.a(pcap-linux.o): In function nl80211_init': pcap-linux.c:(.text+0xd1d): undefined reference to nl_handle_alloc'
pcap-linux.c:(.text+0xd31): undefined reference to genl_connect' pcap-linux.c:(.text+0xd3d): undefined reference to genl_ctrl_alloc_cache'
pcap-linux.c:(.text+0xd59): undefined reference to genl_ctrl_search_by_name' pcap-linux.c:(.text+0xda1): undefined reference to nl_handle_destroy'
pcap-linux.c:(.text+0xe0d): undefined reference to `nl_cache_free'
*/
```

## 命令行参数

- -l: 指定以逗号分隔的用作本地ip地址的列表，用以取代默认的从操作系统获取的ip地址列表
- -p: 指定用于捕获网络通信的 TCP 端口，如果未指定则捕获所有端口
- -T: 时延统计阈值，超过该时间的会通过tc打印超过该时延阈值的包个数，单位为 ms
- -o: 如果有包的时延超过阈值，则把该包的时间戳记录到该文件中，有助于分析报文 
- -t: 在连续两行输出之间等待的时间间隔，以秒为单位
- -n: 在程序退出前，需要执行迭代的次数；0 表示无限
- -f: 控制输出的格式，详见下列说明
- --[no-]header: 是否显示首行提示信息
- -r: 从 pcap 文件中读取信息, 而不是从网络上实时获取
- -help: 显示程序信息和用法
- --version: --version

### 输出格式定制
| Format Code	|Header	|Default?	|Meaning|
|--|--|--|--|
%n	|count|	y	|在迭代期间完成的 request 个数
%a	|avg	|y	|平均 |response 时间
%s	|sum	|y	|response |时间的总和
%x	|sqs	|	  |response 时间的平方和
%m	|min	|y	|最小 response time
%M	|max	|y	|最大 response time
%h	|med	|y	|中间 response time
%S	|stddev	|y	|response 时间值标准差
%v	|var	|	|response 时间的总体方差
%I	|iter#	|	|迭代次数
%t	|elapsed	|	|从第一次迭代开始流逝的秒数
%T	|timestamp	|y	|Unix 时间戳
%%	|	|	|% 字符
\t	|	|	|tab 字符
\n	|	|	|换行字符
95,99|	Adds a prefix	|y	|百分比指示符

## 输出统计项说明
(*单位为 us*)

- timestamp: 时间戳
- count: 包总数
- max: 最大时延
- min: 最小时延
- avg: 平均时延
- med: 按照时延从小到达排序，最中间的时延值，例如100个包，则第50个包的时延为多少
- stddev: 每个包的时延平方之和的平均值，和avg
的平方的差值，该值可以反映时延波动情况
- tc: 时延超过 x ms的包个数，可以通过-T参数指定这个时间
- 95_max: 按照时延从小到达排序，第95%的时延值，例如1000个请求，把他们的时延值排序，第95%的时延也就是第950个数组成员的时延值
- 95_avg: 按照时延从小到达排序，时延值小的95%时延的平均值 
- 95_std: 按照时延从小到达排序，时延值小的95%的stddev值 
- 99_max: 按照时延从小到达排序，第95%的时延值，例如1000个请求，把他们的时延值排序，第%95的时延也就是第990个数组成员的时延值
- 99_avg: 按照时延从小到达排序，时延值小的99%时延的平均值 
- 99_std:按照时延从小到达排序，时延值小的99%的stddev值


## 使用方法

### 在线分析:
```
tcprstat -p 1111 -l 10.2.x.x -t 1 -n 333333 -T 20
```

### 抓包文件离线分析:
```
tcprstat -p 1111 -l 10.2.x.x -t 1 -n 333333 -r xxx.pcap -T 10 -o timetamp.txt
```

## 示例
### 示例一：在线时延实时统计
下面是统计10.12.1.1服务器上面，端口为12322的数据包的响应时间：
- -l 表示统计 10.12.1.1 服务器上面
- -p 表示统计端口为 12322 
- -t 1 表示每隔1s统计一次，也就是每行的打印
- -T 1 表示时延超过1ms的请求的个数。
- -o tcprstat.log 把时延超过-T设置的1ms的数据包时间戳记录到文件，并记录该包的响应时间，见后面的时延tcprstat.log内容

```
[root@s10-12-1-1 test]# ./tcprstat -l 10.12.1.1 -p 12322 -n 3333 -t 1 -T 1 -o tcprstat.log
timestamp count max min avg med stddev tc 95_max 95_avg 95_std 99_max 99_avg 99_std
1483946797 4002 1466 42 199 194 106 6 359 186 82 465 194 92
1483946798 3498 1456 44 208 211 103 1 364 196 87 463 204 94
1483946799 3485 1262 40 203 200 103 1 371 190 87 479 199 96
1483946800 3867 2499 38 206 202 116 5 382 191 86 503 201 96
1483946801 3992 1973 41 201 197 114 6 362 186 84 499 195 94
1483946802 4271 1828 39 196 193 102 1 362 183 83 476 192 91
1483946803 3591 1678 42 206 202 110 5 374 192 86 511 201 94
1483946804 4510 1889 32 195 190 101 2 351 183 78 457 191 87


tcprstat.log记录慢响应数据包的时间戳，delay_time单位为us：

[root@s10-12-1-1 test]# cat tcprstat.log
timestamp: 1483946796.658649 delay_time: 1136(该数据报文产生时间戳是1483946796.658649，响应时间戳是1483946796.658649 + 1136us)
timestamp: 1483946796.884682 delay_time: 1292
timestamp: 1483946796.954704 delay_time: 1326
timestamp: 1483946797.327451 delay_time: 1399
...
timestamp: 1483946799.643757 delay_time: 2499
timestamp: 1483946800.97223 delay_time: 1151
timestamp: 1483946800.103771 delay_time: 1164
timestamp: 1483946800.120869 delay_time: 1315
timestamp: 1483946800.388201 delay_time: 1557
timestamp: 1483946800.655855 delay_time: 1191
...
timestamp: 1483946801.152817 delay_time: 1951
timestamp: 1483946801.181594 delay_time: 1376
...
```

### 示例二：在线抓包 + wireshark离线分析

通过wireshark工具查看时间戳就可以确定出抓包文件中慢响应报文的时间戳，就可以快速定位慢响应报文，分析响应慢的原因 

具体的用法是：
1. 使用```tcpdump```对网卡和端口进行抓包，并把数据保存到 test.pcap 文件中；
2. 使用```tcprstat```对抓到的包进行过滤，过滤出时延超过 x ms的数据，并保存到 tcprstat.log 文件中，此文件中有记录时间戳；
3. 使用```wireshark```打开 test.pcap 文件，按下 ctrl+f，匹配规则为“分组详情”+“宽窄”+“字符串”，在输入框输入 ```tcprstat.log``` 文件里记录的时间戳，就可以找到对应的数据包了。

```
[root@s10-12-1-1 test]# tcpdump -i bond0 port 15211 -w test.pcap
tcpdump: listening on bond0, link-type EN10MB (Ethernet), capture size 65535 bytes
^C90800 packets captured
91141 packets received by filter
291 packets dropped by kernel
[root@s10-12-1-11 test]#
[root@s10-12-1-11 test]# ./tcprstat -l 10.12.1.1 -p 15211 -n 3333 -t 1 -T 1 -o tcprstat.log -r test.pcap
timestamp count max min avg med stddev tc 95_max 95_avg 95_std 99_max 99_avg 99_std
1483947284 5041 1943 38 181 180 97 2 334 169 79 441 177 87
1483947285 4957 1677 39 184 185 98 3 330 172 79 440 180 86
1483947286 4799 1147 38 189 190 97 1 345 177 81 449 185 89
1483947287 4982 1317 39 185 182 96 2 338 173 82 417 181 88
[root@s10-12-1-1 test]#
[root@s10-12-1-1 test]# cat tcprstat.log
timestamp: 1483947283.777246 delay_time: 1943
timestamp: 1483947284.94361 delay_time: 1015
timestamp: 1483947284.848973 delay_time: 1216
timestamp: 1483947285.397089 delay_time: 1187
timestamp: 1483947285.438686 delay_time: 1677
timestamp: 1483947286.702593 delay_time: 1147
timestamp: 1483947286.869181 delay_time: 1317
timestamp: 1483947287.400980 delay_time: 1161
timestamp: 1483947287.794503 delay_time: 1456
timestamp: 1483947287.828082 delay_time: 1333
```

## 目前存在的问题

### 意外退出
tcprstst会不定期意外退出，不管是该升级版本，还是之前的老版本，都存在这个问题。