# 企业邮件系统 - iRedMail 邮件服务器搭建

| 更新时间 | 说明 |
| --- | --- |
| 2022-4-29 11:49:17 | 初版 |

待补充事项：

- `SPF`、`DKIM` 和 `DMARC` 解析待补充。

## 1. 先决条件

- 域名，本次试验环境为 `oook.cn`
- 服务器
- SSL 证书

环境：

- Debian 11.2
- iRedMail 1.5.2

> **警告**
> 阿里云、腾讯云均禁止使用邮件服务所必须的 25 端口提供邮件服务，因此你不能在它们的云服务器部署邮件服务器。

## 2. 准备工作

### 2.1. 更新系统

```bash
sudo apt update

sudo apt upgrade
```

### 2.2. 设置全限定域名（FQDN）主机名

输入以下命令查看当前主机名：

```bash
# 全限定主机名
$ hostname -f
mail.oook.cn

# 短主机名
$ hostname
mail
```

主机名设置主要在两个文件中：

- `/etc/hostname`：短主机名，不是 FQDN。

  ```
  mail
  ```

- `/etc/hosts`：主机名静态查找，**注意：**需要将 FQDN 主机名列为第一项。

  ```
  127.0.0.1	mail.oook.cn mail localhost localhost.localdomain
  ```

设置完成后输入 `hostname -f` 命令验证 FQDN 主机名。如果更新以上两个文件后没有主机名发生变化，需要重启服务器。

### 2.3. 安装 `tar` 和 `gzip` 用于解压缩

```bash
sudo apt-get install tar gzip wget
```

## 3. 安装 iRedMail

安装程序必须使用 `root` 用户执行。

### 3.1. 下载解压 iRedMail 安装包

访问[下载页面](https://www.iredmail.org/download.html)下载最新版本。

```bash
wget https://github.com/iredmail/iRedMail/archive/refs/tags/1.5.2.tar.gz
```

解压：

```
tar zxf iRedMail-1.5.2.tar.gz
```

### 3.2. 运行 iRedMail 安装程序

```bash
cd iRedMail-1.5.2

bash iRedMail.sh
```

安装过程：

1. 欢迎界面

   ![iRedMail 安装过程 - 欢迎界面](https://cdn.jsdelivr.net/gh/jugggao/image-hosting/images_for_notes/20220426111539.png)

2. 设置存储位置

   ![iRedMail 安装过程 - 设置存储位置](https://cdn.jsdelivr.net/gh/jugggao/image-hosting/images_for_notes/20220426141422.png)

3. 设置 Web 服务器

   ![iRedMail 安装过程 - 设置 Web 服务器](https://cdn.jsdelivr.net/gh/jugggao/image-hosting/images_for_notes/20220426141631.png)

4. 选择邮件账号存储数据库

   ![iRedMail 安装过程 - 选择邮件账号存储数据库](https://cdn.jsdelivr.net/gh/jugggao/image-hosting/images_for_notes/20220426141925.png)

   > 各个数据库之间没有太大区别，建议选择自己熟悉的数据库，便于维护。

5. 设置 MySQL 管理账号密码

   ![iRedMail 安装过程 - 设置 MySQL 管理账号密码](https://cdn.jsdelivr.net/gh/jugggao/image-hosting/images_for_notes/20220426142213.png)

6. 设置邮件域名

   ![iRedMail 安装过程 - 设置邮件域名](https://cdn.jsdelivr.net/gh/jugggao/image-hosting/images_for_notes/20220426142434.png)

7. 设置邮件管理员密码

   ![iRedMail 安装过程 - 设置邮件管理员密码](https://cdn.jsdelivr.net/gh/jugggao/image-hosting/images_for_notes/20220426142612.png)

8. 可选组件

   - Roundcubemail：邮件服务器 Web 客户端
   - SOgo：邮件客户端，与 Roundcubemail 二选一即可
   - netdata：系统监控
   - iRedAdmin：iRedMail Web 管理端
   - Fail2ban：禁止密码错误次数太多的 IP 登录，提供安全保障

   ![iRedMail 安装过程 - 可选组件](https://cdn.jsdelivr.net/gh/jugggao/image-hosting/images_for_notes/20220426142712.png)

9. 确定信息

   ![iRedMail 安装过程 - 确认信息](https://cdn.jsdelivr.net/gh/jugggao/image-hosting/images_for_notes/20220426143226.png)

10. 安装完成

    ![iRedMail 安装过程 - 安装完成](https://cdn.jsdelivr.net/gh/jugggao/image-hosting/images_for_notes/20220426144734.png)

11. 重启服务器

    ```bash
    reboot
    ```

## 4. 配置 iRedMail 邮件服务器

### 4.1. 阅读提示文件

阅读 `/root/iRedMail-1.5.2/iRedMail.tips` 文件，其中包含：

- 各个 Web 服务的访问地址、用户名和密码。
- 各个组件的配置文件路径。
- 其他的重要和敏感信息。

### 4.2. 设置 DNS 记录

> `A`、`MX` 记录是必须的，`Reverse PTR`、`SPF`、`DKIM` 和 `DMARC` 是可选的，但是强烈推荐。

#### 4.2.1. A 记录

```
NAME           TTL     TYPE    DATA
mail.oook.cn.  1800    A       10.10.113.52
```

#### 4.2.2. PTR 记录

如果域名没有 PTR 记录，那么垃圾邮件过滤软件可能会阻止来自你邮件服务器的电子邮件。

```
IP              TYPE    DATA
10.10.113.52    PTR     mail.oook.cn.
```

如果使用公网 IP，可能需要联系 ISP 运营商为你的邮件服务器 IP 地址创建反向 PTR 记录。

#### 4.2.3. MX 记录

```
NAME       PRIORITY    TYPE    DATA
oook.cn.   10          mx      mail.oook.cn.
```

该条记录的结果为，发送到 `<user>@oook.cn` 的电子邮件将被传递到 `mail.oook.cn` 邮件服务器。

#### 4.2.4. 自动配置与自动发现记录

自动配置和自动发现记录可以使邮件客户端自动获取邮箱的客户端配置。如果要配置的邮件是 `user@company.com`，那么它将自动检查 `autodiscover.company.com` 以获取正确的配置。

```
NAME                    PRIORITY    TYPE    DATA
autodiscover.oook.cn.   10          mx      mail.oook.cn.
autoconfig.oook.cn.     10          mx      mail.oook.cn.
```

### 4.3. 更换证书

```bash
mv /etc/ssl/certs/iRedMail.crt{,.bak}
mv /etc/ssl/private/iRedMail.key{,.bak}

cp oook.cn.pem /etc/ssl/certs/iRedMail.crt
cp oook.cn.key /etc/ssl/private/iRedMail.key
```

更换后重启服务：

```bash
systemctl restart postfix.service
systemctl restart dovecot.service
systemctl restart nginx.service
systemctl restart mariadb.service
```

## 5. 邮箱管理

### 5.1. 导入用户

iRedMail 自带生成用户的脚本。位置在 `/root/iRedMail-1.5.2/tools/create_mail_user_SQL.sh`。

由于我们之前在「设置存储位置」时修改了默认的邮件存储位置，因此需要在 `/root/iRedMail-1.5.2/tools/create_mail_user_SQL.sh` 脚本中更改以下配置：

```
# Storage base directory used to store users' mail.
STORAGE_BASE_DIRECTORY="/data/vmail/vmail1"
```

输入以下命令就可以生成用户 SQL：

```bash
# usage: bash create_mail_user_SQL.sh <new-email> <plain-password>
bash /root/iRedMail-1.5.2/tools/create_mail_user_SQL.sh peng.gao@oook.cn Ambow99999999 > /tmp/createiredmailuser.sql
```

`/tmp/createiredmailuser.sql` 文件内容如下：

```sql
INSERT INTO mailbox (username, password, name,
                     storagebasedirectory,storagenode, maildir,
                     quota, domain, active, passwordlastchange, created)
             VALUES ('peng.gao@oook.cn', '{SSHA512}x9y3MDjownsViHU/kwTjpWylbRxxp3HjW1/9uYNB6uUfzrJndkwiDW2HcbTqd3uZSvKVs3Far3YSSIJ2MHVidWGAEhs=', 'peng.gao',
                     '/data/vmail','vmail1', 'oook.cn/p/e/n/peng.gao-2022.04.29.12.10.55/',
                     '1024', 'oook.cn', '1', NOW(), NOW());
INSERT INTO forwardings (address, forwarding, domain, dest_domain, is_forwarding)
                 VALUES ('peng.gao@oook.cn', 'peng.gao@oook.cn','oook.cn', 'oook.cn', 1);
```

> **注意**：检查存储位置是否正确，否则登录邮件系统后会报错。

然后进入到 `vmail` 数据库中导入 SQL：

```bash
$ mysql -uroot -p

sql> USE vmail;
sql> SOURCE /tmp/createiredmailuser.sql;
```

然后用生成的新邮件用户和密码即可使用邮件系统。

### 5.2. 批量导入用户

公司发的用户信息为 Excel，因此需要先转换为 CSV。转换后的格式如下。

> 如果 CSV 文件中有中文，需要转化至 UTF8 编码。
> 如果输入格式不正确，还需要使用 `dos2unix` 工具将 CRLF 换行符转换为 LF 换行符。

查看数据的结构：

```bash
$ head -n 10 iRedMailUsersInfo.CSV 
孙婷,15166655123,6665123
田利娜,15266651782,6665782
徐寅杰,17366651315,6665315
周彦,18266650845,6665845
刘慧平,15766652545,6665545
朱传飞,13866657272,6665272
于显静,18766653019,6665019
张莉,13666651561,6665561
倪均明,13866656113,6665113
艾勇,19966657295,6665295
```

批量导入脚本 `geniReadMailUsersSQL` ：

```bash
# !/usr/bin/env bash

IFS=','

while read _ id password
do 
    echo "-- create ${id}@oook.cn user." >> /tmp/create_iredmail_multiuser.sql
    bash /root/iRedMail-1.5.2/tools/create_mail_user_SQL.sh ${id}@oook.cn $password >> /tmp/create_iredmail_multiuser.sql
    echo "" >> /tmp/create_iredmail_multiuser.sql
done < iRedMailUsersInfo.CSV
```

执行脚本：

```bash
./geniReadMailUsersSQL
```

然后进入到 `vmail` 数据库中导入 SQL：

```bash
$ mysql -uroot -p

sql> USE vmail;
sql> SOURCE /tmp/create_iredmail_multiuser.sql;
```

## 6. 问题收集

### 6.1. 国内安装 `web.py` 超时

```
[ INFO ] Installing required Python-3 modules with pip3: web.py>=0.62
WARNING: Retrying (Retry(total=4, connect=None, read=None, redirect=None, status=None)) after connection broken by 'ReadTimeoutError("HTTPSConnectionPool(host='pypi.org', port=443): Read timed out. (read timeout=15)")': /simple/web-py/
Collecting web.py>=0.62
  Downloading web.py-0.62.tar.gz (623 kB)
ERROR: Exception:
Traceback (most recent call last):
  File "/usr/share/python-wheels/resolvelib-0.5.4-py2.py3-none-any.whl/resolvelib/resolvers.py", line 171, in _merge_into_criterion
    crit = self.state.criteria[name]
KeyError: 'web-py'
...

  File "/usr/share/python-wheels/urllib3-1.26.5-py2.py3-none-any.whl/urllib3/response.py", line 443, in _error_catcher
    raise ReadTimeoutError(self._pool, None, "Read timed out.")
urllib3.exceptions.ReadTimeoutError: HTTPSConnectionPool(host='files.pythonhosted.org', port=443): Read timed out
```

使用国内源安装：

```bash
pip install -i https://pypi.tuna.tsinghua.edu.cn/simple web.py
```

## 7. 参考

- https://docs.iredmail.org/install.iredmail.on.debian.ubuntu-zh_CN.html
- https://docs.iredmail.org/file.locations.html
- https://docs.iredmail.org/setup.dns.html
- https://docs.iredmail.org/use.a.bought.ssl.certificate.html
- https://docs.iredmail.org/sql.bulk.create.mail.users.html
