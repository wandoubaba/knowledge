# 各操作系统生成sshkey的方法

---

---

## Windows 10

> 其他版本系统类似

### 1. 先得安装git

> 如果电脑里已经有git bash就跳过这一步

- 上git官网 `https://git-scm.com/` 去下载最新版，注意对应操作系统。
- 安装git，按照向导操作，明白选项意思的就根据自己实际情况配置，不明白的就留下默认值。

### 2. 检查系统内是否已经有sshkey文件

```cmd
cd ~/.ssh
dir
```

> 如果~/.ssh目录下已经存有id_rsa和id_rsa.pub文件的话，那就不需要执行 `3` 了

### 3. （在git bash中）创建sshkey

先打开git bash（是git带的命令行工具）

```bash
ssh-keygen -t rsa -C "yourname@yourpc"
```

> 一路回车（如果希望以后操作git每次push时都输入密码的话，那这里可以设置一个密码）

以上操作完成后，默认会在~/.ssh/目录下生成名为id_rsa和id_rsa.pub的文件。

### 4. 给自己的公钥文件起一个个性化的名字

> 还是在git bash中操作

```bash
cd ~/.ssh
cp id_rsa.pub yourname_yourpc.pub
```

### 5. 完成

在需要用到sshkey的地方（比如git服务器）把你的pub文件部署上去就可以了。

---

## MAC

### 1. 先检查系统内是否已经有sshkey文件

```bash
cd ~/.ssh
ls
```

> 如果~/.ssh目录下已经存有id_rsa和id_rsa.pub文件的话，那就不需要执行 `2` 了

### 2. 创建sshkey

```bash
ssh-keygen -t rsa -C "yourname@yourpc"
```

> 一路回车（如果希望以后操作git每次push时都输入密码的话，那这里可以设置一个密码）

以上操作完成后，默认会在~/.ssh/目录下生成名为id_rsa和id_rsa.pub的文件。

### 3. 给自己的公钥文件起一个个性化的名字

```bash
cd ~/.ssh
cp id_rsa.pub yourname_yourpc.pub
```

### 4. 完成

在需要用到sshkey的地方（比如git服务器）把你的pub文件部署上去就可以了。

---

## linux

> linux下的操作方式与mac非常非常类似。
