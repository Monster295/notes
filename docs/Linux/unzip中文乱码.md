在 Linux 中解压中文文件名压缩包时出现乱码，是因为 unzip 命令默认使用系统的本地编码来解压，而 Windows 下生成的 zip 文件中的编码是 GBK/GB2312 等，导致这些 zip 文件在 Linux 下解压时出现乱码问题。

**解决方法:**

1. **使用 -O 选项指定编码:**

```
unzip -O CP936 xxx.zip
```

该命令将使用 GBK/GB2312 编码来解压 xxx.zip 文件。

2. **使用 unar 命令:**

```
unar xxx.zip
```

unar 命令支持自动识别编码，可以正确解压中文文件名压缩包。

3. **安装 unzip-iconv:**

```
sudo apt install unzip-iconv
```

安装 unzip-iconv 后，就可以使用 -O 选项来指定编码了。

4. **使用其他解压工具:**
- 7-Zip
- PeaZip
- File Roller

这些解压工具都支持自动识别编码，可以正确解压中文文件名压缩包。

在 Ubuntu 系统中，可以使用 `unzip` 命令解压 zip 文件到指定路径。

**基本语法:**

```
unzip [-c][-d directory][-n][-t][-v] zipfile
```

**参数说明:**

- `-c`：将解压缩的结果显示到屏幕上，并对字符做适当的转换。
- `-d directory`：指定解压缩后的文件存放目录。
- `-n`：更新现有的文件。
- `-t`：检查压缩文件是否正确，但不解压。
- `-v`：显示详细的信息。
- `zipfile`：要解压缩的 zip 文件。

**解压到指定路径的示例:**

```
unzip -d /path/to/target/directory zipfile.zip
```

**例如，要将 `myfile.zip` 解压缩到 `/home/user/data` 目录下，可以使用以下命令:**

```
unzip -d /home/user/data myfile.zip
```

**注意:**

- 如果要解压缩的 zip 文件名包含空格，则需要使用引号将文件名括起来。
- 如果目标目录不存在，则 `unzip` 命令会自动创建它。
- 如果目标目录中存在同名文件，则 `unzip` 命令会询问是否覆盖现有的文件。

**其他用法:**

- **解压并显示内容:**

```
unzip -c zipfile.zip
```

- **检查压缩文件是否正确:**

```
unzip -t zipfile.zip
```

- **显示详细解压信息:**

```
unzip -v zipfile.zip
```
