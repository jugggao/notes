# Docker 安装

## Debian

### 卸载旧版本

```bash
sudo apt-get remove docker docker-engine docker.io containerd runc
```

### 配置仓库

```bash
sudo apt-get update

sudo apt-get install \
    ca-certificates \
    curl \
    gnupg \
    lsb-release
    
curl -fsSL https://download.docker.com/linux/debian/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg

echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/debian \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

### 安装 Docker

```bash
sudo apt-get update

sudo apt-get install docker-ce docker-ce-cli containerd.io
```

或者安装指定版本：

```bash
# 查看可以安装的版本
apt-cache madison docker-ce

# 安装指定版本将 <VERSION_STRING> 替换为你想要安装的版本
sudo apt-get install docker-ce=<VERSION_STRING> docker-ce-cli=<VERSION_STRING> containerd.io
```

### （可选）将普通用户加入 Docker 组

```bash
usermod -aG docker sysadmin
```