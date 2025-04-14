# 拉取公司内网harbor中的tar包
1. 在本地hosts文件中增加域名解析，使用`nano /etc/hosts`命令编辑hosts文件

    ```text
    # harbor仓库域名解析
    172.16.0.190   harbor.gogetter.cn
    ```

2. 使用`systemd-resolve --flush-caches`命令刷新dns缓存
3. 编辑`/etc/docker/daemon.json`文件，增加harbor仓库的认证信息

    ```json
    {
    "registry-mirrors": ["https://docker.mirrirs.ustc.edu.cn"],
    "insecure-registries": ["172.16.0.190:8081"],
    "log-driver":"json-file",
    "log-opts": {"max-size":"500m", "max-file":"3"}
    }
    ```
4. 重启docker服务`systemctl restart docker`
5. 使用`docker login 172.16.0.190:8081`命令进行登录，其中，用户名为`deployment`，密码为`Ggzn_12345`