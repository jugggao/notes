# 企业邮件系统 - iRedMail 邮件服务器搭建

## 1. 先决条件

- 域名，本次试验环境为 `oook.cn`
- 服务器
- SSL 证书

环境：

- Debian 11.2
- iRedMail 1.5.2

## 2. 准备工作

### 2.1. 设置全限定域名（FQDN）主机名

输入以下命令查看当前主机名：

```bash
# 全限定主机名
$ hostname -f
mail.oook.cn

# 短主机名
$ hostname
mail
```

主机名配置主要在两个文件中：

- `/etc/hostname`：短主机名，不是 FQDN。

  ```
  mail
  ```

- `/etc/hosts`：主机名静态查找，**注意：**需要将 FQDN 主机名列为第一项。

  ```
  127.0.0.1	mail.oook.cn mail localhost localhost.localdomain
  ```

配置完成后输入 `hostname -f` 命令验证 FQDN 主机名。如果更新以上两个文件后没有主机名发生变化，需要重启服务器。

### 更新系统

```bash
sudo apt update

sudo apt upgrade
```

