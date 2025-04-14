### 固定Ubuntu server 22.04的ip地址

> 从Ubuntu 22.04开始，`gateway4`已被弃用，需要使用`routes`来定义默认网关。请按照以下步骤修改你的Netplan配置文件。

### **修改Netplan配置**

使用以下命令编辑你的Netplan文件，文件名可能不同：

`sudo nano /etc/netplan/01-netcfg.yaml`

将`gateway4`替换为`routes`，修改后的文件应如下所示：

```yaml
network:
  version: 2
  renderer: networkd
  ethernets:
    ens33:  # 请替换为你的实际网卡名称
      dhcp4: no
      addresses:
        - 192.168.1.100/24  # 你的固定IP
      routes:
        - to: default
          via: 192.168.1.1  # 你的网关
      nameservers:
        addresses:
          - 8.8.8.8
          - 8.8.4.4
```

### **应用新配置**

保存并退出（`Ctrl + X`，然后按`Y`，回车），然后应用更改：

`sudo netplan apply`

或者先测试：

`sudo netplan try`

如果没有错误，IP地址应该已经被正确设置。你可以用以下命令检查：

`ip a `

`ip route`

这样，你的Ubuntu服务器就能正确使用固定IP地址了。🚀
