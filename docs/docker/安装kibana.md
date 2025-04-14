1. 上传kibana的tar包至服务器

2. docker load -i

3. 使用以下命令启动kibana
   
   ```shell
   docker run -d --name kibana --restart=always -p 5601:5601 docker.elastic.co/kibana/kibana:7.15.2
   ```

4. 使用`docker exec -it kibana /bin/bash`进入kibana容器

5. `cd config`然后编辑kibana.yml文件

6. 修改es的host地址(docker宿主机的ip)，增加` i18n.locale: "zh-CN" `配置

7. 重启kibana
