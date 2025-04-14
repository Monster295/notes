### 限制docker容器的日志大小

1. 问题描述：
   
   使用docker compose文件启动milvis后，服务器可用硬盘空间急剧下降，排查原因后，发现某个容器的日志文件一直在记录且没有大小限制

2. 解决方案：
   
   修改compose文件中每个service的配置，追加以下配置
   
   ```docker
   logging:
     driver: "json-file"
     options:
       max-size: "10m"
       max-file: "3"
   ```
   
   ### 配置说明：
   
   - **driver**: 指定日志驱动程序。在本例中，设置为 `"json-file"`，这是默认的日志驱动程序。
   - **max-size**: 限制日志文件的最大 大小（例如，"10m" 表示 10 兆字节）。
   - **max-file**: 限制 Docker 保留的日志文件数量（例如，"3" 表示 Docker 会保留 3 个日志文件）。
   
   ### docker daemon配置
   
   1. 新建 daemon.json  
      vi /etc/docker/daemon.json
      
      max-size=500m，意味着一个容器日志大小上限是500M，  
      max-file=3，意味着一个容器有三个日志，分别是id+.json、id+1.json、id+2.json。
      
      ```json
      {
        "log-driver":"json-file",
        "log-opts": {"max-size":"500m", "max-file":"3"}
      }
      ```
   
   2. 重启docker守护进程  
      systemctl daemon-reload && systemctl restart docker
   
   3. 如果同时配置了compose文件和daemon，优先以compose文件为准
   
   4. 如果修改了compose文件，但使用docker inspect containerID查看LogConfig没有生效的话，先使用docker compose stop，然后再使用docker up -d
