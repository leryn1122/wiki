
# 文件系统性能测试工具
参考文档：

- [https://blog.csdn.net/wkl_venus/article/details/127486535](https://blog.csdn.net/wkl_venus/article/details/127486535)

测试工具安装：
```bash
apt install -y fio
```
```bash
# 创建测试路径并挂载
mkdir -p /mnt/{local,s3,nfs}

s3fs performance-test /mnt/s3 \
  -o passwd_file=/etc/passwd-s3fs \
  -o url=https://oss.domain.com \
  -o endpoint='',allow_other,retries=5 \
  -o use_path_request_style

mount -t nfs -o vers=3,tcp,nolock,async,mountproto=tcp,rsize=1048576,wsize=1048576 \
  xxx.xxx.xxx.xxx:/path/to/mount/ /mnt/nfs
```
测试命令：
```bash
fio -filename=/mnt/s3/perf.txt -rw=read \
    -direct=1 -iodepth 32 -thread \
    -ioengine=libaio -bs=4k -size=1G \
    -numjobs=8 -group_reporting -runtime=30 -time_base \
    -name=/root/fio_result_s3_read.txt >> /root/fio_result_s3_read.txt
```
参数如下：

- `filename=/mnt/storage/perf.txt` 测试文件名称，通常选择需要测试的盘的data目录；
- `direct=1` 是否使用 directIO，测试过程绕过OS自带的 buffer。Linux 读写的时候，内核维护了缓存，数据先写到缓存，后面再后台写到 SSD。读的时候也优先读缓存里的数据。这样速度可以加快，但是一旦掉电缓存里的数据就没了。所以有一种模式叫做 directIO，跳过缓存，直接读写 SSD，使测试结果更真实；
- `rw=read` 测试顺序读的 I/O；
- `rw=write` 测试顺序写的 I/O；
- `rw=randread` 测试随机读的 I/O；
- `rw=randwrite` 测试随机写的 I/O；
- `rw=randrw` 测试随机混合读和写的 I/O；
- `bs=4k` 单次io的块文件大小为 4k；
- `size=5G` 本次的测试文件大小为 5g，以每次 4k 的 IO 进行测试；
- `numjobs=8` 测试线程为 8；（需要根据测试的cpu线程数做对应修改）
- `name=job` 一个任务的名字，重复了也没关系； 
- `thread` 使用 pthread_create 创建线程，另一种是 fork 创建进程。进程的开销比线程要大，一般都采用thread测试；
- `group_reporting` 关于显示结果的，汇总每个进程的信息；
- `runtime=120` 测试时间为 120 秒，如果不写则一直将 5g 文件分 4k 每次写完为止；
- `time_based` 如果设置的话，即使 file 已被完全读写或写完，也要执行完 runtime 规定的时间，它是通过循环执行相同的负载来实现的；
- `ioengine=libaio` 指定io引擎使用 libaio 方式；<br />libaio：Linux本地异步 I/O。请注意，Linux 可能只支持具有非缓冲 I/O 的排队行为（设置为 `direct=1` 或 `buffered=0`）；
- `iodepth=32` 队列的深度为32。

某次结果（2C/4G/40G SSD/千兆网卡）：

|  | local（基准） | NFS | S3 |
| --- | --- | --- | --- |
| 读（MB/s） | 508 | 107 | 428 |
| 写（MB/s） | 256 | 17.8 | 248 |
| CPU占用率（%读） | 2.88 | 0.72 | 2.35 |
| CPU占用率（%写） | 1.68 | 0.23 | 1.65 |
| 随机读（MB/s） | 405 | 109 | 398 |
| 随机写（MB/s） | 153 | 18.8 | 160 |
| CPU占用率（%随机读） | 2.64 | 0.83 | 2.55 |
| CPU占用率（%随机写） | 1.26 | 0.21 | 1.14 |
| 随机读写混合（MB/s） | 88.2 | 12.1 | 123 |
| CPU占用率（%随机读写混合） | 1.43 | 0.26 | 1.90 |


