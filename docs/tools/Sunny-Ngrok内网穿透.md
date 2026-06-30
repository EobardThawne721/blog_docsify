# Sunny-Ngrok内网穿透

## 一.Sunny-Ngrok

### 1.1 注册、登录。

​								http://www.ngrok.cc/



### 1.2 Windows使用

* 点击隧道管理-开通隧道-立即购买 `美国Ngrok免费服务器原价¥0.00 折扣价¥0`

* > 1. 隧道协议选择 **http**
  >
  > 2. 隧道名字**自定义**
  >
  > 3. 前置域名**自定义**, 将会自动加入到分配的域名：http://自定义.free.idcfengye.com
  >
  > 4. 本地端口填写**自己发布项目的IP地址+端口号**，如本机：127.0.0.1:8080
  > 5. http验证用户名和http验证密码**可以不填**

* 进入隧道管理，根据服务器类型进行Ngrok客户端下载，如本机Windows

* 解压下载的文件，找到windows_amd64文件夹，运行里面的**Sunny-Ngrok启动工具.bat**

* 回到我们的隧道管理，找到我们的id，复制id到我们刚刚打开的启动工具，回车确定

* 根据控制台上面的Forwarding地址进行访问即可

==注意：如果关闭了**Sunny-Ngrok启动工具.bat**，我们通过域名也就访问不到了==



### 1.3 Linux使用

* 首先在Linux系统中安装解压软件

> yum -y install zip

* 在官网下载 [Linux 64Bit版本](https://www.ngrok.cc/sunny/linux_amd64.zip?v=2.1) 
* 通过Xftp将压缩包上传给Linux
* 解压上传上来的文件

> unzip  文件名

* 执行

> ./sunny clientid  隧道id

==注意：若想要Linux后台运行，则可以执行以下命令==

> ./sunny clientid  隧道id &



### 1.4 帮助文档

[Sunny-Ngrok使用教程 · Sunny-Ngrok说明文档](https://www.ngrok.cc/_book/)



