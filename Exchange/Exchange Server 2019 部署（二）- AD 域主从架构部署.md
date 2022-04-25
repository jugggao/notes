# Exchange Server 2019 部署（二）- AD 域主从架构部署


## 1.  服务器规划

| 服务器名称 | 操作系统                            | 角色                      | IP 地址          |
| :-------- | :--------------------------------- | :------------------------ | :--------------- |
| ad-01     | Windows 2019 Server for Datacenter | Active Directory 主服务器 | 10.10.113.121/24 |
| ad-02     | Windows 2019 Server for Datacenter | Active Directory 从服务器 | 10.10.113.122/24 |

## 2. 主服务器配置

### 2.1. 系统初始化配置

#### 2.1.1. 修改系统 SID（可选）

由于本次试验使用 VMware 虚拟机模板克隆得到虚拟机，克隆出来的系统的用户 SID 全部都相同。因此需要运行 Windows Sysprep 工具 ，可以防止出现 SID 不匹配的警告。

```powershell
Invoke-Expression 'C:\Windows\System32\Sysprep\Sysprep.exe /generalize /oobe /reboot /quiet'
```

![Windows Server 2019 修改系统 SID](https://raw.githubusercontent.com/jugggao/image-hosting/main/notes/exchange/exchange%20server%202019%20%E9%83%A8%E7%BD%B2%EF%BC%88%E4%BA%8C%EF%BC%89-%20ad%20%E5%9F%9F%E9%83%A8%E7%BD%B2.md/58120617249781.png "Windows Server 2019 修改系统 SID")

重启之后，根据提示填入相对应的信息即可。

需确保各系统的用户 SID 都不一致，输入以下命令进行查询：

```powershell
WMIC UserAccount Get Name,Sid
```

![Windows Server 2019 查询用户 SID](https://raw.githubusercontent.com/jugggao/image-hosting/main/notes/exchange/exchange%20server%202019%20%E9%83%A8%E7%BD%B2%EF%BC%88%E4%BA%8C%EF%BC%89-%20ad%20%E5%9F%9F%E4%B8%BB%E4%BB%8E%E6%9E%B6%E6%9E%84%E9%83%A8%E7%BD%B2.md/256325017247385.png "Windows Server 2019 查询用户 SID")

#### 2.1.2. 修改计算机名

![AD 域主服务器更改计算机名](https://raw.githubusercontent.com/jugggao/image-hosting/main/notes/exchange/exchange%20server%202019%20%E9%83%A8%E7%BD%B2%EF%BC%88%E4%BA%8C%EF%BC%89-%20ad%20%E5%9F%9F%E4%B8%BB%E4%BB%8E%E6%9E%B6%E6%9E%84%E9%83%A8%E7%BD%B2.md/485530818226128.png "AD 域主服务器更改计算机名")

修改完成后重启计算机。

#### 2.1.3. 修改网络配置

![AD 域主服务器网络配置](https://raw.githubusercontent.com/jugggao/image-hosting/main/notes/exchange/exchange%20server%202019%20%E9%83%A8%E7%BD%B2%EF%BC%88%E4%BA%8C%EF%BC%89-%20ad%20%E5%9F%9F%E4%B8%BB%E4%BB%8E%E6%9E%B6%E6%9E%84%E9%83%A8%E7%BD%B2.md/464050618244887.png "AD 域主服务器网络配置")

### 2.2. AD 域控安装配置

#### 2.2.1. 安装域控

1. 添加角色和功能

    ![AD 域主服务器安装 - 添加角色和功能](https://raw.githubusercontent.com/jugggao/image-hosting/main/notes/exchange/exchange%20server%202019%20%E9%83%A8%E7%BD%B2%EF%BC%88%E4%BA%8C%EF%BC%89-%20ad%20%E5%9F%9F%E4%B8%BB%E4%BB%8E%E6%9E%B6%E6%9E%84%E9%83%A8%E7%BD%B2.md/34351518248568.png)

2. 确认信息

    ![AD 域控主服务器安装 - 确认信息](https://raw.githubusercontent.com/jugggao/image-hosting/main/notes/exchange/exchange%20server%202019%20%E9%83%A8%E7%BD%B2%EF%BC%88%E4%BA%8C%EF%BC%89-%20ad%20%E5%9F%9F%E4%B8%BB%E4%BB%8E%E6%9E%B6%E6%9E%84%E9%83%A8%E7%BD%B2.md/567461618243704.png)

3. 选择安装类型

    ![AD 域控主服务器安装 - 安装类型](https://raw.githubusercontent.com/jugggao/image-hosting/main/notes/exchange/exchange%20server%202019%20%E9%83%A8%E7%BD%B2%EF%BC%88%E4%BA%8C%EF%BC%89-%20ad%20%E5%9F%9F%E4%B8%BB%E4%BB%8E%E6%9E%B6%E6%9E%84%E9%83%A8%E7%BD%B2.md/596241818230384.png)

4. 服务器选择

    ![AD 域控主服务器安装 - 服务器选择](https://raw.githubusercontent.com/jugggao/image-hosting/main/notes/exchange/exchange%20server%202019%20%E9%83%A8%E7%BD%B2%EF%BC%88%E4%BA%8C%EF%BC%89-%20ad%20%E5%9F%9F%E4%B8%BB%E4%BB%8E%E6%9E%B6%E6%9E%84%E9%83%A8%E7%BD%B2.md/307352018220914.png)

5. 添加服务器角色

    ![AD 域主服务器安装 - 添加 AD 域服务](https://raw.githubusercontent.com/jugggao/image-hosting/main/notes/exchange/exchange%20server%202019%20%E9%83%A8%E7%BD%B2%EF%BC%88%E4%BA%8C%EF%BC%89-%20ad%20%E5%9F%9F%E4%B8%BB%E4%BB%8E%E6%9E%B6%E6%9E%84%E9%83%A8%E7%BD%B2.md/513552218223418.png)

    ![AD 域主服务器安装 - 添加 DNS 服务](https://raw.githubusercontent.com/jugggao/image-hosting/main/notes/exchange/exchange%20server%202019%20%E9%83%A8%E7%BD%B2%EF%BC%88%E4%BA%8C%EF%BC%89-%20ad%20%E5%9F%9F%E4%B8%BB%E4%BB%8E%E6%9E%B6%E6%9E%84%E9%83%A8%E7%BD%B2.md/349482918232365.png)

    ![AD 域主服务器安装 - 完成添加服务器角色](https://raw.githubusercontent.com/jugggao/image-hosting/main/notes/exchange/exchange%20server%202019%20%E9%83%A8%E7%BD%B2%EF%BC%88%E4%BA%8C%EF%BC%89-%20ad%20%E5%9F%9F%E4%B8%BB%E4%BB%8E%E6%9E%B6%E6%9E%84%E9%83%A8%E7%BD%B2.md/380563118225250.png)
    
6. 添加功能

    ![AD 域主服务器安装 - 添加功能](https://raw.githubusercontent.com/jugggao/image-hosting/main/notes/exchange/exchange%20server%202019%20%E9%83%A8%E7%BD%B2%EF%BC%88%E4%BA%8C%EF%BC%89-%20ad%20%E5%9F%9F%E4%B8%BB%E4%BB%8E%E6%9E%B6%E6%9E%84%E9%83%A8%E7%BD%B2.md/538653418225859.png)
    
7. 确认 AD 域服务信息

    ![AD 域主服务器安装 - 确认 AD 域服务器信息](https://raw.githubusercontent.com/jugggao/image-hosting/main/notes/exchange/exchange%20server%202019%20%E9%83%A8%E7%BD%B2%EF%BC%88%E4%BA%8C%EF%BC%89-%20ad%20%E5%9F%9F%E4%B8%BB%E4%BB%8E%E6%9E%B6%E6%9E%84%E9%83%A8%E7%BD%B2.md/570923618252814.png)

8. 确认 DNS 服务器信息

    ![AD 域主服务器安装 - 确定 DNS 服务器信息](https://raw.githubusercontent.com/jugggao/image-hosting/main/notes/exchange/exchange%20server%202019%20%E9%83%A8%E7%BD%B2%EF%BC%88%E4%BA%8C%EF%BC%89-%20ad%20%E5%9F%9F%E4%B8%BB%E4%BB%8E%E6%9E%B6%E6%9E%84%E9%83%A8%E7%BD%B2.md/578283718235027.png)
    
9. 确认安装所选内容

    ![AD 域主服务器安装 - 确认安装所选内容](https://raw.githubusercontent.com/jugggao/image-hosting/main/notes/exchange/exchange%20server%202019%20%E9%83%A8%E7%BD%B2%EF%BC%88%E4%BA%8C%EF%BC%89-%20ad%20%E5%9F%9F%E4%B8%BB%E4%BB%8E%E6%9E%B6%E6%9E%84%E9%83%A8%E7%BD%B2.md/2963918224325.png)

10. 完成安装
