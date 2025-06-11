## 

### 在 Ubuntu 22.04 上安装 Docker

开始安装:

1、检查卸载老版本Docker

```bash
sudo apt-get remove docker docker-engine docker.io containerd runc 
```

2、更新软件包
```bash
sudo apt update
sudo apt upgrade
```

3、安装docker依赖
```bash
sudo apt-get install ca-certificates curl gnupg lsb-release
```

4、添加docker密钥
```bash
curl -fsSL http://mirrors.aliyun.com/docker-ce/linux/ubuntu/gpg | sudo apt-key add -
```
5、添加阿里云docker软件源
```bash
sudo add-apt-repository "deb [arch=amd64] http://mirrors.aliyun.com/docker-ce/linux/ubuntu $(lsb_release -cs) stable"
```
6、安装docker
```bash
apt install docker-ce docker-ce-cli containerd.io
```
---
下方需要科学上网

**步骤 1：更新系统**

```
sudo apt update
```

**步骤 2：安装 Docker 依赖项**

```
sudo apt install apt-transport-https ca-certificates curl software-properties-common
```

**步骤 3：添加 Docker 官方 GPG 密钥**

```
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
```

**步骤 4：添加 Docker 存储库**

```
echo "deb [arch=amd64] https://download.docker.com/linux/ubuntu focal stable" | sudo tee /etc/apt/sources.list.d/docker.list
```

**步骤 5：更新系统**

```
sudo apt update
```

**步骤 6：安装 Docker 引擎**

```
sudo apt install docker-ce -
```

**步骤 7：验证 Docker 是否已安装**

```
docker --version
```

**输出结果:**

```
Docker version 20.10.14, build e22a77e
```

**步骤 8：设置 Docker 用户组**

**将您的用户添加到 docker 用户组，以便无需 sudo 即可运行 Docker 命令:**

```
sudo usermod -aG docker $USER
```

**注销并重新登录后，您就可以无需 sudo 即可运行 Docker 命令。**
