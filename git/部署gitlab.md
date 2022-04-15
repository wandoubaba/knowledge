# 安装部署gitlab

---

## 在centos7部署

1. 安装配置必要的依赖

- ssh和防火墙

```shell
sudo yum install -y curl policycoreutils-python openssh-server perl
# Enable OpenSSH server daemon if not enabled: sudo systemctl status sshd
sudo systemctl enable sshd
sudo systemctl start sshd
# Check if opening the firewall is needed with: sudo systemctl status firewalld
sudo firewall-cmd --permanent --add-service=http
sudo firewall-cmd --permanent --add-service=https
sudo systemctl reload firewalld

```

- 安装postfix用来发通知邮件

```shell
sudo yum install postfix
sudo systemctl enable postfix
sudo systemctl start postfix
```

2. 安装gitlab

- 下载安装包

```shell
curl https://packages.gitlab.com/install/repositories/gitlab/gitlab-ee/script.rpm.sh | sudo bash
```

- 安装gitlab（先做好域名解析）

```shell
# EXTERNAL_URL是想要使用的域名
sudo EXTERNAL_URL="https://gitlab.example.com" yum install -y gitlab-ee
```

- Web登录进行配置
