1. 将dump后的sql复制到宿主机中

2. 执行以下命令，其中 `<` 后的路径是sql文件在宿主机中的路径
   
   ```shell
   docker exec -i CONTAINER_NAME_OR_ID mysql -uUSERNAME -pPASSWORD DATABASE_NAME < /path/inside/container/dump.sql
   ```
   
   在这里，你需要替换以下占位符：
   
   - `CONTAINER_NAME_OR_ID`：你的MySQL容器的名称或ID。
   - `USERNAME`：你的MySQL用户名。
   - `PASSWORD`：你的MySQL密码。
   - `DATABASE_NAME`：你想要恢复的数据库的名称。
