# 阿里云磁盘扩容（centos7）

---

## 场景

在阿里云控制台对主机云盘成功完成磁盘扩容操作。

## centos系统内操作

查看磁盘和分区

```bash
fdisk -l
```

结果有可能会看到有一个名为/dev/vda的磁盘，同时有一个名为/dev/vda1的分区，而且/dev/vda的磁盘容量已经是扩容后的容量了。

查看分区大小

```bash
df -h
```

结果可以看到分区大小还是扩容前的大小。

为分区扩容

```bash
growpart /dev/vda 1
```

> 语法 growpart <DeviceName> <PartionNumber>

扩容文件系统

```bash
resize2fs /dev/vda1
```

> 语法 resize2fs <PartitionName>
