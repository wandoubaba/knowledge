# vim快捷操作

---

## 定义tab键为4个空格

```bash
# ubuntu
vim /etc/vim/vimrc
# centos
vim /etc/vimrc
```

在最后添加如下配置

```text
set ts=4
set sw=4
set expandtab
```

## 查找替换

把每一行中的所有fromstring替换为tostring

```txt
:%s/fromstring/tostring/g
```

把每一行中的第一个fromstring替换为tostring

```txt
:%s/fromstring/tostring
```

## 粘贴时缩进错乱

```txt
:set paste
```
