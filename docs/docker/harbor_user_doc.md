# Harbor用户手册&仓库规范

> author: liumingkun,
> last_update: 2023/12/11

# -------------------------------------------

### 前言： harbor服务地址

```
地址： http://172.16.0.190:8081
测试用户名： test
测试密码： Ggzn_1234

test用户拥有common项目的访客(R)权限和test项目的开发(RW)权限
如果需要获取更高权限，请联系管理员为你进行定制化用户创建和权限分配。
```

# 上篇：harbor仓库规范

### TAG规范

1. 【重要】推送镜像必须要有REPOSITORY和TAG，否则harbor会在一段时间后把你的容器GC掉。
2. 【重要】如果镜像需要GPU环境，必须在容器名上特别标注。否则一律认为是CPU版本。
3. 【重要】latest标签应在最新版本完整测试无bug后，由功能验收者登录harbor进行手动添加或推送。
4. 【推荐】每个项目可以根据自身情况自定义tag规则，要求见名知义即可。

tag命令格式如下

```
$ docker tag 镜像ID  harbor地址/harbor项目名/容器名:标签文本
```

列出几个示例，供大家参考。

```
某天的逆向合成CPU版本
$ docker tag 1f3f4ef7dc4d 172.16.0.190:8081/retro/retro_cpu:v20230714
$ docker tag 1f3f4ef7dc4d 172.16.0.190:8081/retro/retro:v20230714

分子图像识别GPU最新稳定版本
$ docker tag 1f3f4ef7dc4d 172.16.0.190:8081/image_recognition/decimer-image-transformer-gpu:latest

配置了ubuntu国内镜像源、conda国内源、pip国内源，基于 anaconda2020.2（py3.7.6）封装的墙内镜像
$ docker tag ef74c1f34ddf 172.16.0.190:8081/common/anaconda:v2020.2.cn

默认ubuntu官方源、conda官方源、pip官方源，基于 anaconda2020.2（py3.7.6）封装的墙外镜像
$ docker tag 1f3f7ff4e7d  172.16.0.190:8081/common/anaconda:v2020.2.en
```

# -------------------------------------------------------------------------------

# 下篇：用户手册

### 客户端的docker配置文件增加镜像仓库

如果你配置了https，应该就可以像下边“registry-mirrors”中科大镜像那样配置。
由于docker默认不推荐使用非https方式推送镜像，所以在需要pull镜像时要加上harbor地址，
然后还要在“insecure-registries”配置无SSL证书的仓库（我们内网没法用域名也就不用去配置SSL）。

```
$ sudo vim /etc/docker/daemon.json

{
    "registry-mirrors": ["https://docker.mirrirs.ustc.edu.cn"],
    "insecure-registries": ["172.16.0.190:8081"]
}
```

重启docker

```
$ systemctl restart docker
```

本地docker登录到harbor仓库

```
登录，交互式输密码
$ docker login 172.16.0.190:8081

或者 类似于mysql一样在命令行上输入密码
$ docker login -uUSER -pPASSWORD 172.16.0.190:8081

(base) jintaoyang@jellyfish:~$ docker login 172.16.0.190:8081
Username: 你的用户名
Password: 看不见的神秘字符串

WARNING! Your password will be stored unencrypted in /home/jintaoyang/.docker/config.json.
Configure a credential helper to remove this warning. See
https://docs.docker.com/engine/reference/commandline/login/#credentials-store

Login Succeeded
```

> 警告：docker将你的认证信息存放在 ```/home/USERNAME/.docker/config.json```
> 如果你的用户名或密码改了，把这个文件删掉重新进行 docker login 即可。

### 推送本地镜像到harbor

> 注意：必须本地镜像必须要有tag，否则harbor会在一段时间后给你GC掉。

【可选】你的镜像上需要有REPOSITORY和TAG，可以先用 ```docker images``` 查看。
如果```Dockerfile```没有指定-t参数，或者使用 ```docker commit```
构建镜像，就会在REPOSITORY和TAG处展示```<NONE>```。

```
$ docker tag 镜像ID  harbor地址/harbor项目名/容器名:标签(通常用来标记版本号)
$ docker tag 1f3f4ef7dc4d  172.16.0.190:8081/retro/retro_cpu:v20230714
```

推送到harbor，完事以后可以在浏览器登录harbor看一下。

```
$ docker images

$ docker push REPOSITORY:TAG
$ docker push 172.16.0.190:8081/retro/retro_cpu:v20230714
```

### 从harbor上下载(拉取)私有镜像到本地docker

拉取命令格式，有两种格式如下。推荐第一种。第二种命令拉取到本地的镜像是没有tag的，需要你自己设定。

```
【推荐】命令格式1：
$ docker pull harbor地址/harbor项目名/容器名:标签(一般都是版本号)

命令格式2：
$ docker pull harbor地址/harbor项目名/容器名@sha256签名:89492b84d3a51c7e67
```

那么我们如何知道 docker pull 后面的一大串字符串是什么呢？步骤如下：

```
1、 登录harbor网页
2、 左侧菜单点击'项目'
3、 在项目列表中选择你想拉取的项目(比如retro)，进入项目镜像仓库
4、 在镜像列表中可以看到不用的镜像名称，点击你想要的（比如：retro/retro_cpu）
5.1、 在'retro/retro_cpu'镜像列表中，可以看到许许多多的历史版本，复制拉取命令到本地docker执行即可（此命令是上一小节中所说的第2种，如果你喜欢这种命令，那么久到此为止了。）
5.2、 在'retro/retro_cpu'镜像列表中，可以看到许许多多的历史版本。根据tag选择你想要的镜像点进去，在tag列表中复制拉取命令到本地执行即可（此命令是上一小节中所说的第1种，拉取到本地的镜像包含tag）
```

### 如何从harbor上删除不想要的镜像呢？

```
首先在群里@你的项目管理员，然后让他登录harbor手动删除。（向上管理🥶）
```

### 登录harbor后如何从互联网拉取公共的镜像呢？

前言

```
当本地docker登录harbor后，当拉取DockerHub中的公开镜像(docker pull redis:7)时，
实际上还是从harbor上拉取（代理dockerHub），而不走你之前配置的国内镜像源了。
```

解决方案1（不推荐）

```
在harbor上专门建立一个proxy项目用来代理163的镜像源，
当我们需要DockerHub中的公开镜像(比如redis:7)时，从harbor的proxy项目上拉取即可。
这样会消耗服务器流量。（不推荐，所以没有开启这个选项。）
```

解决方案2（推荐）

```
拉取镜像时在镜像前面加上代理地址，示例如下：

常规镜像代理
官方命令：docker pull stilleshan/frpc:latest
代理命令：docker pull dockerproxy.com/stilleshan/frpc:latest

根镜像代理
官方命令：docker pull nginx:latest  
官方命令：docker pull docker.io/library/ubuntu:22.04
代理命令：docker pull dockerproxy.com/library/nginx:latest
代理命令：docker pull dockerproxy.com/library/ubuntu:22.04
```

# -------------------------------------------

# 番外篇：异常解决

Docker login 错误：

```
现象：Error response from daemon: Get "http://172.16.0.190:8081/v2/": Get "http://172.16.0.97:8081/service/token?account=liumingkun&client_id=docker&offline_token=true&service=harbor-registry": context deadline exceeded (Client.Timeout exceeded while awaiting headers)
原因：
1、可能忘记配置“insecure-registries”。
2、harbor.yml文件的hostname配置项阻止了不正确的访问。
```
