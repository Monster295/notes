```java
使用场景：
    现在需要读取 同一局域网下windows服务器共享文件夹中的文件，没挂载之前，由于本地测试时，
 后端在windows系统中运行，因此直接使用//ip+路径的形式可以进行文件的读写操作；
    然而，客户现场环境是Ubuntu系统，并不能直接找到对应的路径，因此需要挂载到linux中
```

###### 1. windows如何共享文件夹

选择一个文件夹，右击属性==>共享==>高级共享==>共享此文件夹

###### 2. Linux端挂载共享文件夹

```python
mount -t cifs -o username=Administrator,password=123456 //192.168.0.87/task4 /mnt/
```

命令解读：

```python
mount：    挂载命令
-t：    指定文件系统类型
cifs：    CIFS 是一个新提出的协议，它使程序可以访问远程Internet计算机上的文件并要求此计算机提供服务。
-o：    挂载选项参数，使用,分隔
username：    用户名
password：    用户密码
//192.168.0.87/task4 ：    源路径，共享文件夹主机的IP地址，以及共享的文件夹名称。（共享文件夹不需要填绝对路径）
/mnt/：    目标路径，linux中的挂载目录。
```

3. 彩晶示例：

```python
sudo mount -t cifs -o username=Administrator,password=LUOBO62197bc. /192.168.1.26/cjoe_interfacedata /mnt/
```

 这行命令会将26服务器中的共享文件夹挂载到 /mnt/中

###### 关于挂载共享文件夹的报错解决

1.关闭windows防火墙

2.确认windows的CIFS文件共享功能是否开启

  启动或关闭Windows功能-SMB 1.0/CIFS 文件共享支持

3.检查源路径与挂载路径是否正确
  注意IP地址是否正确，其次注意格式，源路径是IP/共享文件夹。

  目标路径的话，要注意路径是否存在，是否已经被挂载的问题。

4.检查用户名及密码是否正确
  用户名严格要求大小写，不管是用户名还是密码，错一个都会报错。

5.检查共享文件夹权限
  使用时权限拒绝，那么大概就跟这个有关，默认是只读权限，需要手动修改。

6.查看Linux中是否安装cifs-utils
  如果没有这个的话，挂载也可能会失败。apt show cifs-utils
