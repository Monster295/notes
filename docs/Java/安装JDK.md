### 设置JAVA_HOME环境变量

安装jdk8: `apt install openjdk-8-jdk -y`

执行以下命令，找到 OpenJDK 8 的安装路径：

`readlink -f $(which java)`

输出示例：

`/usr/lib/jvm/java-8-openjdk-amd64/jre/bin/java`

Java 的主目录是 `/usr/lib/jvm/java-8-openjdk-amd64`（去掉 `/jre/bin/java`）。

打开终端，使用`nano`或`vim`编辑器来编辑`~/.bashrc`文件（如果你使用的是其他shell，可能是`~/.zshrc`或`~/.profile`）：

```
nano ~/.bashrc
```

在文件末尾添加以下行，将`<path_to_java>`替换为你的Java安装路径：

```
export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64
export PATH=$JAVA_HOME/bin:$PATH
```

保存并关闭编辑器（在`nano`中，按`Ctrl + X`，然后按`Y`确认保存，最后按`Enter`）。

### 使环境变量生效

为了让这些更改立即生效，你可以通过以下命令重新加载`~/.bashrc`文件：

```
source ~/.bashrc
```

或者，你可以关闭并重新打开终端。

### 验证JAVA_HOME设置

验证`JAVA_HOME`是否已正确设置：

```
echo $JAVA_HOME
```

这个命令应该返回你之前设置的Java安装路径。
