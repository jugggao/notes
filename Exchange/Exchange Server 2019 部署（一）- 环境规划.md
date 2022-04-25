# Exchange Server 2019 部署（一）- 环境规划

## 1. 服务器规划

- 域控制器 2 台，主从架构。
- 邮件服务器 2 台，做负载均衡。

| 服务器名称 | 操作系统                            | 角色                                             | IP 地址          |
| :-------- | :--------------------------------- | :---------------------------------------------- | :--------------- |
| ad-01     | Windows 2019 Server for Datacenter | Active Directory 服务器、DNS 服务器、CA 证书服务器 | 10.10.113.121/24 |
| ad-02     | Windows 2019 Server for Datacenter | Active Directory 服务器、DNS 服务器、CA 证书服务器 | 10.10.113.122/24 |
| ex-01     | Windows 2019 Server for Datacenter | 客户端访问服务、邮箱服务器                         | 10.10.113.123/24 |
| ex-02     | Windows 2019 Server for Datacenter | 客户端访问服务、邮箱服务器                         | 10.10.113.124/24 |


## 2. 域名规划

内部网络域名为 `oook.local`。

公网域名为 `oook.cn`。其中：
- 邮件服务器域名为 `mail.oook.cn`。
- 自动发现服务器域名为 `autodiscover.oook.cn`。