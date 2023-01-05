# JsRPC

-- js逆向之远程调用(rpc)免去抠代码补环境

> tip:懒得自己编译的 ，[releases](https://gitee.com/ripperTs/jsrpc/releases)中有已经编译好的包 （win、Linux、MacOS的都有~）

## 目录结构

```dart
-- main.go (服务器的主代码)
-- resouces/JsEnv.js (客户端注入js环境)
```

## 基本介绍

运行服务器程序和js脚本 即可让它们通信，实现调用接口执行js获取想要的值(加解密)

## 实现

原理：在网站的控制台新建一个WebScoket客户端链接到服务器通信，调用服务器的接口 服务器会发送信息给客户端 客户端接收到要执行的方法执行完js代码后把获得想要的内容发回给服务器 服务器接收到后再显示出来

> 说明：本方法可以https证书且支持wss

在https的网站想要新建WebSocket连接如果是连接到普通的ws可能会报安全错误，好像连接本地(127.0.0.1)不会报错~ 可以用本地和wss 你自己看着玩

1. 无https证书者。直接编译main.go 我试了一下，发现使用本地ip(127.0.0.1)可以在https的网站直接连接ws使用 默认端口12080
2. 有https证书者。修改main.go文件 把r.Run()注释掉，把r.RunTls注释取消掉 并且参数设置证书的路径 直接输入名字就是当前路径 默认端口：12443

> 另外的题外话，有域名没证书不会搞的 或者有域名有公网(非固定IP的)都可以搞成的，自己研究研究

## 食用方法

### 打开编译好的文件，开启服务

如下图所示

![image.png](./assets/image.png)

**api 简介**

- `/list` :查看当前连接的ws服务
- `/ws`  :浏览器注入ws连接的接口
- `/result` :获取数据的接口  (数据格式`json: {"group":"hhh","hello":"好困啊yes","name":"baidu","status":"200"}` )

说明：接口用`group`和`name`来区分 如 `ws://127.0.0.1:12080/ws?group={}&name={}` //注入`ws`的例子 `group`和`name`都可以随便



#### 示例接口

`http://127.0.0.1:12080/go?group=hhh&name=baidu&action=hello&param=yes`

参数解析说明

`http://127.0.0.1:12080/go?group={}&name={}&action={}&param={}`    

这是调用的接口 `group`和`name`填写上面注入时候的，`action`是注册的方法名,`param`是可选的参数（这个参数完全取决于`console`注入的方法）  


### 注入JS，构建通信环境

打开JsEnv 复制粘贴到网站控制台(注意有时要放开断点)

![image](https://user-images.githubusercontent.com/41224971/134774597-5c8c845b-072e-40d1-bdf7-24e89f78b22e.png)

### 注入ws与方法

```js
// 连接通信
var demo = new Hlclient("ws://127.0.0.1:12080/ws?group=hhh&name=baidu");
// 注册一个方法 第一个参数hello为方法名，
// 第二个参数为函数，resolve里面的值是想要的值(发送到服务器的)
// param是可传参参数，可以忽略
demo.regAction("hello", function (resolve, param) {
    var c = "好困啊" + param;
    resolve(c);
})
```

![image](https://user-images.githubusercontent.com/41224971/134774859-a4594f23-b828-4538-8b89-9d96813f7d1e.png)

### 访问接口，获得数据

```dart
http://127.0.0.1:12080/go?group=hhh&name=baidu&action=hello&param=yes
// 其中 hello是会变的 是action名字。 用代码访问的时候要注意这个名字
{
  "group":"hhh",
  "hello":"好困啊yes",
  "name":"baidu",
  "status":"200"
}
```

![image](https://user-images.githubusercontent.com/41224971/134775037-167724d4-ae94-4fcf-88c4-d881621b712c.png)

## 食用方法

### 控制台直接注入方法

1. 将`resouces/JsEnv.js`代码完全复制，在浏览器`f12`控制台直接粘贴回车即可
2. 将上方的**注入ws与方法** 同样的复制然后在控制台粘贴回车

### 油猴脚本

推荐使用油猴脚本在浏览器外部挂载js,示例代码：`https://greasyfork.org/zh-CN/scripts/440234-%E6%8A%96%E9%9F%B3json-rpc`
