---
title: Charles
date: 2018-12-11 16:38:57
tags:
- 移动端抓包
- https乱码
categories: 前端
---
> 需求： 完成移动端H5抓包，分析请求，解决https请求出现的乱码问题。
<p hidden><!--more--></p>

## 使用背景
在移动端开发中，我们无法像PC端那样可以直接查看请求，判断请求是否发对，返回的结果是否没有被篡改，请求对于我们来讲，就是透明的，不可见的，尤其是每个人操作习惯的不同，导致程序的隐藏bug出现时，我们不能直接凭经验判断出bug的由来，请求的发送顺序等，所以抓包工具的使用在移动端开发中，显得十分必要。

## 工具安装
1. [官网下载](https://www.charlesproxy.com/latest-release/download.do);
2. [破解版jar包](https://pan.baidu.com/s/15DIgEm_5AcrCbYQ7kLxlRA) 提取码：qxa2
破解jar包使用方式：右击Charles > 显示包内容 > Contents > Java > 替换charles.jar(使用破解版jar包来替换)

## 使用方式
### HTTP请求
对于普通的请求，如果查看请求的方式和返回结果，直接对使用的设备配置代理即可；代理服务器为本机mac的ip，端口号为8888，这样你在charles上就可以看到客户端发出的请求了，如果你在本地进行开发，想查看本地代码在客户端的预览效果，也可以通过代理的方式
***方案1***
1. 将localhost改为ip，如localhost:3005可以改成192.168.1.128: 3005 OR 配置hosts文件，将127.0.0.1指向一个自定义域名如lc.dy.com
2. 安装浏览器插件FE Helper，将链接生成二维码，手机扫描二维码即可；
3. 手机端配置代理，charles即可捕捉到请求
***方案2***
1. 安装浏览器插件FE Helper，将链接(localhost:3005)生成二维码，手机扫描二维码即可；
2. 手机端配置代理，charles即可捕捉到请求
***方案3***
1. 此方法适用于，app内嵌h5，调试app内的h5页面，因为app的qa环境里面内嵌的是我们的qa环境，我们想要app的h5指向本地，需要改映射，将app内的h5的qa地址映射为我们本地的地址。
2. Tools > Map Remote Settings > Enable Map Remote > Add
3. 配置map from 和map to，即将原本要发到from的请求映射到to上的请求来。
### HTTPS请求
HTTPS请求时加密过的请求，返回的结果会出现乱码，解决这种问题，需要三个步骤：

1. mac安装证书
2. 客户端安装证书
3. mac配置代理
## 证书安装
### mac安装证书步骤
1. 打开Charles，Help > SSL Proxying > Install Charles Root Certificate
2. 自动打开的钥匙串访问 > 搜索 Charles Proxy > 双击 > 点击信任，使用此证书时始终信任，默认会勾选所有项。
### 安卓手机安装证书的步骤：
***方案1***

1. 打开Charles，Help > SSL Proxying > Install Charles Root Certificate on a Mobile Device or Remote Browser
2. 手机配置代理，代理服务器为本机mac的ip，端口号为8888
3. 手机浏览器打开下载证书
4. 证书会自动下载，针对不同机型，有的可以成功，有的失败了，提示“无法安装证书”。
***方案2***
1. 打开Charles，眼睛瞄到顶上菜单栏的 “ Help ” ，点开。
2. SSL Proxying > Save Charles Root Certificate > 选择cer格式的文件
3. 传到微信里
4. 找到设置 > 更多设置 > 系统安全 > 从SD卡读取证书 > tentcent > micromsg > weixin > download > charles-ssl-proxying-certificate.cer（选中，即可安装）
### 苹果手机安装证书步骤
1. 打开Charles，Help > SSL Proxying > Install Charles Root Certificate on a Mobile Device or Remote Browser
2. 手机配置代理，代理服务器为本机mac的ip，端口号为8888
3. 手机浏览器打开下载证书，证书会自动下载，
4. 设置 > 通用设置 > 关于手机 > 证书信任设置 > 勾选已下载的证书文件
### 配置代理
1. 打开Charles, Proxy > SSL Proxying Settings > 勾选Enable SSL Proxying > 点击Add（添加你要访问的https请求的域名和端口号）
2. 如果不知道对应域名和端口号，可以添加Host: Port:
3. OK
## 乱码缘由
### 为什么使用https?
http的不足：

通信使用明文（不加密），内容可能被窃听
不验证通信双方的身份，因此有可能遭遇伪装
无法证明报文的完整性，所以有可能已遭篡改
tips 即使已经加密处理过的通信，在传输过程中，通信内容也是会被窥视到的，这点和未加密的通信相同，只是说加密后的报文的含义，难以破解，但是加密之后的报文本身还是可见的。

### 通信加密
http协议中本身没有加密机制，但是可以通过和SSL(安全套接层)的组合使用，加密HTTP的通信内容，用SSL建立安全的通信线路之后，在这条线路上进行的HTTP通信协议被称为HTTPS。SSL不仅提供加密处理，还使用了证书，可用于确定通信方。证书由值得信任的第三方机构颁发，用以证明服务器和客户端是真实存在的。通过使用证书，以证明通信方式意料中的服务器。

这也就解释了为什么在没有做任何配置的情况下，我们抓到的内容是乱码的。那为什么配置证书之后，就可以正常查看了呢？

***未完待续

## 参考文章
1. [细说 Charles 配置 HTTPS 代理的乱码问题](https://malcolmyu.github.io/2017/02/26/Dive-into-Charles-HTTPS-Proxying/)
2. [charles本地调试之map和rewrite功能](https://www.cnblogs.com/wonyun/p/5586746.html)


