# Exchange Server 2019 部署（二）- AD 域主从架构部署

## 1. 服务器规划

| 服务器名称 | 操作系统                           | 角色                      | IP 地址          |
| :--------- | :--------------------------------- | :------------------------ | :--------------- |
| ad-01      | Windows 2019 Server for Datacenter | Active Directory 主服务器 | 10.10.113.121/24 |
| ad-02      | Windows 2019 Server for Datacenter | Active Directory 从服务器 | 10.10.113.122/24 |

## 2. 主服务器 AD 域安装配置

### 2.1. 主服务器系统初始化配置

#### 2.1.1. 主服务器修改系统 SID（可选）

由于本次试验使用 VMware 虚拟机模板克隆得到虚拟机，克隆出来的系统的用户 SID 全部都相同。因此需要运行 Windows Sysprep 工具 ，可以防止出现 SID 不匹配的警告。

```powershell
Invoke-Expression 'C:\Windows\System32\Sysprep\Sysprep.exe /generalize /oobe /reboot /quiet'
```

![Windows Server 2019 修改系统 SID](https://cdn.jsdelivr.net/gh/jugggao/image-hosting/images_for_notes/20220426103511.png)

重启之后，根据提示填入相对应的信息即可。

需确保各系统的用户 SID 都不一致，输入以下命令进行查询：

```powershell
WMIC UserAccount Get Name,Sid
```

![Windows Server 2019 查询用户 SID](https://cdn.jsdelivr.net/gh/jugggao/image-hosting/images_for_notes/20220426103541.png)

#### 2.1.2. 主服务器修改计算机名

![主服务器更改计算机名](https://cdn.jsdelivr.net/gh/jugggao/image-hosting/images_for_notes/20220426103607.png)

修改完成后重启计算机。

#### 2.1.3. 主服务器修改网络配置

![主服务器网络配置](https://cdn.jsdelivr.net/gh/jugggao/image-hosting/images_for_notes/20220426103630.png)

### 2.2. 主服务器 AD 域安装

1. 添加角色和功能

   ![主服务器 AD 域安装 - 添加角色和功能](https://cdn.jsdelivr.net/gh/jugggao/image-hosting/images_for_notes/20220426103740.png)

2. 确认信息

   ![主服务器 AD 域安装 - 确认信息](https://raw.githubusercontent.com/jugggao/image-hosting/main/notes/exchange/exchange%20server%202019%20%E9%83%A8%E7%BD%B2%EF%BC%88%E4%BA%8C%EF%BC%89-%20ad%20%E5%9F%9F%E4%B8%BB%E4%BB%8E%E6%9E%B6%E6%9E%84%E9%83%A8%E7%BD%B2.md/567461618243704.png)

3. 选择安装类型

   ![主服务器 AD 域安装 - 安装类型](https://cdn.jsdelivr.net/gh/jugggao/image-hosting/images_for_notes/20220426103852.png)

4. 选择服务器

   ![主服务器 AD 域安装 - 选择服务器](https://cdn.jsdelivr.net/gh/jugggao/image-hosting/images_for_notes/20220426104014.png)

5. 添加服务器角色

   ![主服务器 AD 域安装 - 添加 AD 域服务](https://cdn.jsdelivr.net/gh/jugggao/image-hosting/images_for_notes/20220426104107.png)

   ![主服务器 AD 域安装 - 添加 DNS 服务](https://cdn.jsdelivr.net/gh/jugggao/image-hosting/images_for_notes/20220426104139.png)

   ![主服务器 AD 域安装 - 完成添加服务器角色](https://cdn.jsdelivr.net/gh/jugggao/image-hosting/images_for_notes/20220426104231.png)

6. 添加功能

   ![主服务器 AD 域安装 - 添加功能](https://cdn.jsdelivr.net/gh/jugggao/image-hosting/images_for_notes/20220426104253.png)

7. 确认 AD 域服务信息

   ![主服务器 AD 域安装 - 确认 AD 域服务器信息](https://cdn.jsdelivr.net/gh/jugggao/image-hosting/images_for_notes/20220426104333.png)

8. 确认 DNS 服务器信息

   ![主服务器 AD 域安装 - 确定 DNS 服务器信息](https://cdn.jsdelivr.net/gh/jugggao/image-hosting/images_for_notes/20220426104357.png)

9. 确认安装所选内容

   ![主服务器 AD 域安装 - 确认安装所选内容](https://cdn.jsdelivr.net/gh/jugggao/image-hosting/images_for_notes/20220426104500.png)

10. 完成安装

    ![主服务器 AD 域安装 - 完成安装](https://cdn.jsdelivr.net/gh/jugggao/image-hosting/images_for_notes/20220426110324.png)

### 2.3. 主服务器 AD 域配置

1. 提升域控制器

   ![主服务器 AD 域配置 - 提升域控制器](https://cdn.jsdelivr.net/gh/jugggao/image-hosting/images_for_notes/20220426111243.png)

2. 新建林

   ![主服务器 AD 域配置 - 新建林 ](https://cdn.jsdelivr.net/gh/jugggao/image-hosting/images_for_notes/20220505110047.png)

3. 设置 DSRM 密码

   ![主服务器 AD 域配置 - 配置 DSRM 密码](https://cdn.jsdelivr.net/gh/jugggao/image-hosting/images_for_notes/20220505110424.png)

4. 跳过 DNS 配置

   后续再创建 DNS 服务器，此步骤直接选择下一步即可。

   ![主服务器 AD 域配置 - 跳过 DNS 配置](https://cdn.jsdelivr.net/gh/jugggao/image-hosting/images_for_notes/20220505110558.png)

5. 设置 NetBIOS 名称

   ![主服务器 AD 域配置 - 设置 NetBIOS 名称](https://cdn.jsdelivr.net/gh/jugggao/image-hosting/images_for_notes/20220505141327.png)

6. 设置存储路径

   ![主服务器 AD 域配置 - 设置存储路径](https://cdn.jsdelivr.net/gh/jugggao/image-hosting/images_for_notes/20220505141626.png)

   > 生产环境中建议将存储位置存储至单独的磁盘中，便于扩容和备份。

7. 确认配置

   ![主服务器 AD 域配置 - 确认配置](https://cdn.jsdelivr.net/gh/jugggao/image-hosting/images_for_notes/20220505141820.png)

8. 检查先决条件

   ![主服务器 AD 域配置 - 检查先决条件](https://cdn.jsdelivr.net/gh/jugggao/image-hosting/images_for_notes/20220505142245.png)

配置过程会重启服务器，无需操作，等待安装完即可。

### 2.4. 主服务器 AD 域测试

1. 打开 AD 域用户和计算机

   打开服务器管理界面，选择「AD DS」，右键选择「AD-01」这个节点，在弹出的界面选择「Active Directory 用户和计算机」

   ![主服务器 AD 域测试 - 打开 AD 域用户和计算机](https://cdn.jsdelivr.net/gh/jugggao/image-hosting/images_for_notes/20220505150015.png)

2. 新建组织单位

   右键「oook.local」域，新建「组织单位」，设置组织单位名称为 `OOOK`。

   ![主服务器 AD 域测试 - 新建组织单位](https://cdn.jsdelivr.net/gh/jugggao/image-hosting/images_for_notes/20220505151346.png)
   ![主服务器 AD 域测试 - 设置组织单位名称](https://cdn.jsdelivr.net/gh/jugggao/image-hosting/images_for_notes/20220505151623.png)

3. 新建组

   右键「OOOK」组织，新建「组」，设置组名为 `System Manager`。

   ![主服务器 AD 域测试 - 新建组](https://cdn.jsdelivr.net/gh/jugggao/image-hosting/images_for_notes/20220505151833.png)
   ![主服务器 AD 域测试 - 设置组名称](https://cdn.jsdelivr.net/gh/jugggao/image-hosting/images_for_notes/20220505152100.png)

4. 新建用户

   右键「OOOK」组织，新建「用户」，设置用户信息与密码。

   ![主服务器 AD 域测试 - 新建用户](https://cdn.jsdelivr.net/gh/jugggao/image-hosting/images_for_notes/20220505152314.png)
   ![主服务器 AD 域测试 - 设置用户信息](https://cdn.jsdelivr.net/gh/jugggao/image-hosting/images_for_notes/20220505152638.png)
   ![主服务器 AD 域测试 - 设置用户密码](https://cdn.jsdelivr.net/gh/jugggao/image-hosting/images_for_notes/20220505152737.png)

5. 设置用户权限

   右键刚才创建的用户，点击「属性」，点击「隶属于」，点击「添加」给用户添加 `Administrator` 和 `Remote Desktop Users` 组。

   ![主服务器 AD 域测试 - 打开用户属性](https://cdn.jsdelivr.net/gh/jugggao/image-hosting/images_for_notes/20220505152919.png)
   ![主服务器 AD 域测试 - 添加用户权限](https://cdn.jsdelivr.net/gh/jugggao/image-hosting/images_for_notes/20220505153522.png)

6. 使用域用户远程登录

   ![主服务器 AD 域测试 - 使用域用户远程登录](https://cdn.jsdelivr.net/gh/jugggao/image-hosting/images_for_notes/20220505153854.png)

   ![主服务器 AD 域测试 - 确认用户和服务器信息](https://cdn.jsdelivr.net/gh/jugggao/image-hosting/images_for_notes/20220505154304.png)

## 3. 从服务器 AD 域安装配置

### 3.1. 从服务器系统初始化配置

#### 3.1.1. 从服务器修改系统 SID（可选）

如果从服务器也是从模板或者其他虚拟机克隆出来的，那么还需要更改系统的 SID。

参考 [2.1.1. 主服务器修改系统 SID（可选）](#211-主服务器修改系统-sid可选)。

#### 3.1.2. 从服务器修改计算机名

![从服务器更改计算机名](https://cdn.jsdelivr.net/gh/jugggao/image-hosting/images_for_notes/20220505174422.png)

修改完成后重启计算机。

#### 3.1.3. 从服务器修改网络配置

![从服务器修改网络配置](https://cdn.jsdelivr.net/gh/jugggao/image-hosting/images_for_notes/20220505174541.png)

### 3.2. 从服务器 AD 域安装

1. 添加角色和功能

   ![从服务器 AD 域安装 - 添加角色和功能](https://cdn.jsdelivr.net/gh/jugggao/image-hosting/images_for_notes/20220505174744.png)

2. 确认信息

   ![从服务器 AD 域安装 - 确认信息](https://cdn.jsdelivr.net/gh/jugggao/image-hosting/images_for_notes/20220505174917.png)

3. 选择安装类型

   ![从服务器 AD 域安装 - 选择安装类型](https://cdn.jsdelivr.net/gh/jugggao/image-hosting/images_for_notes/20220505175000.png)

4. 选择服务器

   ![从服务器 AD 域安装 - 选择服务器](https://cdn.jsdelivr.net/gh/jugggao/image-hosting/images_for_notes/20220505175037.png)

5. 添加服务器角色

   ![从服务器 AD 域安装 - 添加服务器角色](https://cdn.jsdelivr.net/gh/jugggao/image-hosting/images_for_notes/20220505175223.png)

6. 添加功能

   ![从服务器 AD 域安装 - 添加功能](https://cdn.jsdelivr.net/gh/jugggao/image-hosting/images_for_notes/20220505175416.png)

7. 确认 AD 域服务器信息

   ![从服务器 AD 域安装 - 确认 AD 域服务器信息](https://cdn.jsdelivr.net/gh/jugggao/image-hosting/images_for_notes/20220505175451.png)

8. 确认 DNS 服务器信息

   ![从服务器 AD 域安装 - 确认 DNS 服务器信息](https://cdn.jsdelivr.net/gh/jugggao/image-hosting/images_for_notes/20220505175521.png)

9. 确认安装所选内容

   ![从服务器 AD 域安装 - 确认安装所选内容](https://cdn.jsdelivr.net/gh/jugggao/image-hosting/images_for_notes/20220505175613.png)

10. 完成安装

    ![从服务器 AD 域安装 - 完成安装](https://cdn.jsdelivr.net/gh/jugggao/image-hosting/images_for_notes/20220505175751.png)

### 3.3. 从服务器 AD 域配置

1. 提升域控制器

   ![从服务器 AD 域配置 - 提升域控制器](https://cdn.jsdelivr.net/gh/jugggao/image-hosting/images_for_notes/20220505175904.png)

2. 将从域控制器添加到现有域

   ![从服务器 AD 域配置 - 添加到现有域](https://cdn.jsdelivr.net/gh/jugggao/image-hosting/images_for_notes/20220505180445.png)
   ![从服务器 AD 域配置 - 验证域管理员用户](https://cdn.jsdelivr.net/gh/jugggao/image-hosting/images_for_notes/20220505180548.png)
   ![从服务器 AD 域配置 - 确认域信息](https://cdn.jsdelivr.net/gh/jugggao/image-hosting/images_for_notes/20220505180619.png)

3. 设置 DSRM 密码

   ![从服务器 AD 域配置 - 设置 DSRM 密码](https://cdn.jsdelivr.net/gh/jugggao/image-hosting/images_for_notes/20220505180750.png)

4. 跳过 DNS 选项

   ![从服务器 AD 域配置 - 跳过 DNS 选项](https://cdn.jsdelivr.net/gh/jugggao/image-hosting/images_for_notes/20220505180905.png)

5. 选择复制项

   ![从服务器 AD 域配置 - 选择复制项](https://cdn.jsdelivr.net/gh/jugggao/image-hosting/images_for_notes/20220505180958.png)

6. 选择存储路径

   ![从服务器 AD 域配置 - 选择存储路径](https://cdn.jsdelivr.net/gh/jugggao/image-hosting/images_for_notes/20220505181035.png)

   > 生产环境中建议将存储位置存储至单独的磁盘中，便于扩容和备份。

7. 确认配置

   ![从服务器 AD 域配置 - 确认配置](https://cdn.jsdelivr.net/gh/jugggao/image-hosting/images_for_notes/20220505181119.png)

8. 检查先决条件

   ![从服务器 AD 域配置 - 检查先决条件](https://cdn.jsdelivr.net/gh/jugggao/image-hosting/images_for_notes/20220505181151.png)

配置过程会重启服务器，无需操作，等待安装完即可。

### 3.4. 确认从服务器同步配置

在从服务器上打开「Active Directory 用户和计算机」，确认在主服务器添加的林、组织、组和用户是否同步。

![从服务器 AD 域测试 - 确认从服务器同步配置](https://cdn.jsdelivr.net/gh/jugggao/image-hosting/images_for_notes/20220505182304.png)

## 4. 主从服务器 DNS 配置

DNS 配置需主从服务器配置相同。

### 4.1. 同步配置

1. 打开 DNS 配置界面

    ![主从服务器 DNS 配置 - 打开 DNS 配置界面](https://cdn.jsdelivr.net/gh/jugggao/image-hosting/images_for_notes/20220505183009.png)

2. 更改区域类型

    右键「oook.com」点击「属性」，「更改」Active Directory 集成区域类型。

    ![主从服务器 DNS 配置 - 更改区域类型](https://cdn.jsdelivr.net/gh/jugggao/image-hosting/images_for_notes/20220505183552.png)

3. 更改复制选项

    右键「oook.com」点击「属性」，「更改」复制选项。

    ![主从服务器 DNS 配置 - 更改复制选项](https://cdn.jsdelivr.net/gh/jugggao/image-hosting/images_for_notes/20220505184040.png)

4. 确认服务器名称

    ![主从服务器 DNS 配置 - 确认服务器名称](https://cdn.jsdelivr.net/gh/jugggao/image-hosting/images_for_notes/20220505184148.png)

    > 如果缺少，尝试手动添加。

5. 测试 DNS 同步功能

    手动添加一条解析，确认另一台服务器同步完成

    ![主从服务器 DNS 配置 - 测试 DNS 同步功能](https://cdn.jsdelivr.net/gh/jugggao/image-hosting/images_for_notes/20220505184815.png)

    然后在另一台服务器上刷新 DNS 记录查看是否同步完成。

### 4.2. 转发器配置（可选）

默认情况下，这两台服务器 DNS 无法访问公网域名。如果想能解析公网，需要添加转发器。

1. 编辑转发器

    邮件 「ad-01.oook.local」点击「属性」，在属性界面中点击「转发器」，点击「编辑」。

    ![主从服务器 DNS 配置 - 编辑转发器](https://cdn.jsdelivr.net/gh/jugggao/image-hosting/images_for_notes/20220505190333.png)

2. 添加转发器

    在地址栏中输入 `8.8.8.8` 回车后再输入 `114.114.114.114` 回车后点击「确定」。

    ![主从服务器 DNS 配置 - 添加转发器](https://cdn.jsdelivr.net/gh/jugggao/image-hosting/images_for_notes/20220505190510.png)

3. 验证 DNS 服务器

    打开命令行终端，输入 `nslookup ad-01.oook.local` 与 `nslookup www.baidu.com` 查看解析是否正常。

    ![主从服务器 DNS 配置 - 验证 DNS 服务器](https://cdn.jsdelivr.net/gh/jugggao/image-hosting/images_for_notes/20220505191252.png)

