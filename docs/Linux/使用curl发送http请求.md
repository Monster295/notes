### curl发送HTTP请求

1. 参数说明
   
   1. 格式：`curl -H 请求头 -d 请求体 -X 请求协议 接口地址
   
   2. | 参数              | 内容     | 格式                                                                                            |
      | --------------- | ------ | --------------------------------------------------------------------------------------------- |
      | -H(或者 --header) | 请求头    | "Content-Type: application/json"                                                              |
      | -d              | POST内容 | '{"id": "001", "name":"张三", "phone":"13099999999"}' 或者<br/>'id=001&name=张三&phone=13099999999' |
      | -X              | 请求协议   | POST、GET、DELETE、PUSH、PUT、OPTIONS、HEAD                                                         |

2. 示例说明
   
   1. **application/x-www-form-urlencoded**  
      最常见的一种 POST 请求，用 [curl](https://so.csdn.net/so/search?q=curl&spm=1001.2101.3001.7020) 发起这种请求也很简单。
      
      ```shell
      $ curl  -X POST -d 'name=张三'  http://localhost:2000/api/basic
      ```
   
   2. **application/json**
      
      跟发起 `application/x-www-form-urlencoded` 类型的 POST 请求类似，-d 参数值是 `JSON 字符串`，并且多了一个 `Content-Type: application/json` 指定发送内容的格式。
      
      ```shell
      $ curl -H "Content-Type: application/json" -X POST -d '{"id": "001", "name":"张三", "phone":"13099999999"}'  http://localhost:2000/api/json
      ```
   
   3. **multipart/form-data**
      
      这种请求一般涉及到文件上传。后端对这种类型请求的处理也复杂一些。
      
      ```shell
      $ curl -F raw=@raw.data -F name=张三 http://localhost:2000/api/multipart
      ```
   
   4. **把文件内容作为要提交的数据**
      
      如果要提交的数据不像前面例子中只有一个 name: 张三 键值对，数据比较多，都写在命令行里很不方便，也容易出错，那么可以把数据内容先写到文件里，通过 -d @filename 的方式来提交数据。这是 -d 参数的一种使用方式，所以前面用到 -d 参数的地方都可以这样用。
      实际上就是把 -d 参数值写在命令行里，变成了写在文件里。跟 multipart/form-data 中上传文件的 POST 方式不是一回事。@ 符号表明后面跟的是文件名，要读取这个文件的内容作为 -d 的参数。
      
      例如，有一个 JSON 文件 data.json 内容如下:
      
      ```json
      {
       "id": "001",
       "name":"张三",
       "phone":"13099999999"
      }
      ```
      
      ```shell
      $ curl -H "Content-Type: application/json" -X POST -d @data.json http://localhost:2000/api/json
      ```
      
      来提交数据。
