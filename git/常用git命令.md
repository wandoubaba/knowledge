# 常用git命令

> git官方中文文档 <https://www.git-scm.com/book/zh/v2> 中包含更多的信息
---

## git演示

## 从远程克隆一个仓库

```bash
# 通过SSH连接克隆一个仓库（可能需要事先配置SSH公钥）
# 执行成功后会在当前路径创建一个名为fsdoc的目录，进入目录后即可以用git命令操作仓库了
git clone git@gitee.com:wandoubaba517/fsdoc.git
# 通过HTTP连接克隆一个仓库（有时可能会需要用户名和密码）
# 执行成功后会在当前路径创建一个名为callapp_mrcp的目录
git clone https://gitee.com/polaris-arvin_admin/callapp_mrcp.git
```

> 除了`git clone`命令外，其他git命令都需要在仓库目录下进行操作

## 分支操作

> 别忘了在仓库目录下操作

```bash
# 查看所有分支（本地和远程）
git branch -a
# 切换分支
git checkout develop
# 创建一个名为iss053的分支同时切换到这个新分支
git checkout -b iss053
# 在iss053分支提交修改
git commit -a -m "添加一些git分支操作"
# 将iss053分支的变更合并到master分支
git checkout master
git merge iss053
# 合并后可以删iss053分支
git branch -d iss053
# 如果合并时发现冲突，需要手动处理一下
```

## 代码推拉

> 别忘了在仓库目录下操作

```bash
# 从远程分支拉取代码到本地
git pull    # 会提取默认关联的远程分支

# 添加本次要跟踪的代码(全部)
git add .
# 提交变更（在本地）
git commit -m "本次提交说明内容"

# 把变更推送到远程服务器
git push    # 推送到默认关联的远程分支
git push origin # 有些时候当关联有问题时，可能需要手写origin
git push origin bob # 可以指定目标远程分支
```

## 给本地的一个文件夹添加git管理并关联远程仓库

```bash
# 在这个文件夹内执行
git init
# 给本地的一个git文件夹关联一个远程服务器
git remote add origin git@gitee.com:wandoubaba517/ps.git
# 对目录内所有文件建立跟踪
git add .
# 提交本次跟踪
git commit -m "这次提交的说明"
# 推送当前分支并建立与远程上游的跟踪
git push --set-upstream origin master
```

## 在一个本地目录上添加多个远程仓库

```bash
# 关联另一个远程仓库
git remote set-url --add origin git@gitee.com:wandoubaba517/personal.git
# 推送当前分支并建立与远程上游的跟踪
git push --set-upstream origin master
# 以后再推送时直接push就行了
git push
```
