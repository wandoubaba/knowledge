# 为PHP7.4安装pgsql和pdo_pgsql扩展

## 环境与版本

操作系统：CentOS7.9 / Debian11

> 在Debian上操作相关简单，所以本文以Centos7.9为例

软件版本：PHP7.4, postgresql15

## 操作过程

假设基本的LNMP环境都已经安装好了，但是没有安装pgsql扩展。

首先要先安装postgresql才可以安装pgsql扩展，其实不需要安装数据库的服务端，安装devel包就行了。

> postgresql官方下载安装网址：<https://www.postgresql.org/download/>

```sh
# Install the repository RPM:
sudo yum install -y https://download.postgresql.org/pub/repos/yum/reporpms/EL-7-x86_64/pgdg-redhat-repo-latest.noarch.rpm

# Install PostgreSQL:
# 这句和官网的不一样，官网让安装postgresql15-server，而我们只需要支持扩展，所以只安装postgresql15-devel
sudo yum install -y postgresql15-devel
```

以上任务执行完毕后，postgresql15的开发包被安装到了`/usr/pgsql-15/`中，后面我们会用到`/usr/pgsql-15/bin`这个路径。

下面需要找到我们PHP7.4的源码目录（没有的话就临时下载一份），进入源码目录`src/ext/pgsql`，执行以下操作：

```sh
phpize
# 注意替换下面的PATH/TO/php-config为真实的php-config路径，用find / -name "php-config"可以找到
./configure --with-pgsql=/usr/pgsql-15/bin/ --with-php-config=PATH/TO/php-config
make
make install
```

正常情况下，名为pgsql.so的文件将会出现在`PATH/TO/PHP/lib/php/extensions/no-debug-non-zts-20190902/`目录中，记住这个路径。

用php --ini命令找到php.ini和php-cli.ini文件，分别编辑这两个文件，在最后面添加以下内容：

```ini
[pgsql]
extension=PATH/TO/PHP/lib/php/extensions/no-debug-non-zts-20190902/pgsql.so
```

> 再次提醒，别忘了把`PATH/TO/PHP`替换成你的环境中的实际路径。

执行`php -m | grep pgsql`应该可以看到pgsql扩展已经安装好了。

PHP7.4源码目录中的`src/ext/pdo_pgsql`是pdo_pgsql扩展的源码，安装过程与pgsql扩展是相同的，这里就不说了。
