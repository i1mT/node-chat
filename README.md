## 效果图
动态图，有点大（4M）请耐心等加载。
![image](http://oqapmzmc9.bkt.clouddn.com/node-chat%20%281%29.gif)

## 随便说说
最近在做东西的时候有一个对战功能，需要用到Socket技术，于是了解了一番相关的实现方案，最后选择了Nodejs以及基于Node的socket.io  最主要的原因还是Node比较好写，相比于PHP好多个函数来说简洁太多了。

本文虽说是从0开始，但不是说无编程基础的也可以，至少要懂以下东西。
1. 基础的HTML
2. Javascript语法，很基础的jQuery语法

因为本文是说从0开始，所以必须要说一下Socket是什么东西，有较好基础的可以跳过这段直接到 **如何开始** 处开始。

## 为什么要使用Socket

不引官话，我直接按照我的理解通俗来说。通常的网站或是其他的联网的应用，因为要获取数据，都需要发送一个请求，这个请求你可以将它看作一个网址，比如你在浏览器键入网址"www.baidu.com"回车，就是请求了这个地址，然后它会返回给你一些东西，前面的就会给你返回百度首页的页面，浏览器自己会解析它（这又涉及到了其他的方面，我们暂且不谈）。

所以通常的用户和服务器的通信就只能是：用户发送请求 ->  服务器接受请求 -> 服务器返回结果 -> 用户收到结果。在这种情况下，想要做即时应用就无法实现。

想想一下，现在你和别人在对战，你打掉了他8点血，你可以告诉服务器这个信息，但是服务器没有办法在你告诉它之后，也迅速告诉你的对手，他掉了8点血。除非对手当时发送一个请求，来查看自己当前血量。但是游戏是实时的，不可能你每隔几秒钟就询问服务器更新一下自己的血量。

这里的需求就是让己方的数据发送之后即时发送给对方，那么只需要服务器也能够以主动发送数据给用户即可，所以需要一种客户端<=>服务器双向通信的技术，就是Socket。

## BTW
本文也相当于是[Socket.io](https://Socket.io)官方Demo的翻译，想看原教程的可以去看看原文。

***

## 如何开始
那我们知道了为什么要用Socket之后，我们来盘算一下该如何完成这个项目。

首先我们要用Nodejs，那就要去安装Nodejs，网上教程很多，我不想赘述，给你们一篇我觉得适合的教程：[Nodejs环境搭建](https://www.cnblogs.com/ewqv/p/6747269.html)

装好之后在cmd中键入 `node -v` 与 `npm -v`，显示版本号就说明安装没有问题了。

然后我们先新建一个工程文件夹，名字就叫`node-chat`，在里面我们新建两个文件，一个是`server.js`，一个是`index.html`，注意后缀名。

![文件展示](http://oqapmzmc9.bkt.clouddn.com/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20180507152916.png)

然后先安装两个东西。

打开CMD

因为npm是国外的库可能有点慢，可以安装cnpm，执行下面命令

`npm install -g cnpm --registry=https://registry.npm.taobao.org`

然后输入

`cnpm install --save express`

等待安装成功，再安装

`cnpm install --save socket.io`

都安装成功之后我们就可以开始着手Coding了。

## 功能分析

我们要做一个聊天室，简单起见，就不做私聊的功能了，那么我们想要的功能可以是这些：

1. 每个人有自己的昵称，在进入聊天室的时候自己输入。
2. 每个人都可以发言
3. 有一个区域用来展示所有的发言，并且要实时更新
4. 有人加入的时候提示xxx加入了群聊

# 开始Coding

## Nodejs之 Hello World
Nodejs是用来当服务器的，所以我们先来用Nodejs搭建一个服务器。其中用到了express这个框架，不知道的想知道可以搜搜相关知识，不想知道直接不要管也没关系。

让我们在server.js中写下以下代码：

```
var app = require('express')(); //引入express库
var http = require('http').Server(app); //将express注册到http中

//当访问根目录时，返回Hello World
app.get('/', function(req, res){
  res.send('<h1>Hello world</h1>');
});

//启动监听，监听3000端口
http.listen(3000, function(){
  console.log('listening on *:3000');
});
```
保存之后，我们在命令行cd到server.js所在的文件夹下，执行

`node server.js`

跟下面图片相同就说明成功了

![hello world控制台](http://oqapmzmc9.bkt.clouddn.com/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20180507154233.png)

然后打开浏览器，输入`localhost:3000`回车。

![hello world浏览器](http://oqapmzmc9.bkt.clouddn.com/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20180507154407.png)

OK，Node搭建的服务器就OK了。接下来，我们让浏览器进入的时候，跳转到我们的index.html页面，我们在那写聊天室的界面。

## 跳转到聊天室页面

### 聊天室页面的实现
在`index.html`中输入以下代码：


```
<!doctype html>
<html>
  <head>
    <title>Socket.IO chat</title>
    <style>
      * { margin: 0; padding: 0; box-sizing: border-box; }
      body { font: 13px Helvetica, Arial; }
      form { background: #000; padding: 3px; position: fixed; bottom: 0; width: 100%; }
      form input { border: 0; padding: 10px; width: 90%; margin-right: .5%; }
      form button { width: 9%; background: rgb(130, 224, 255); border: none; padding: 10px; }
      #messages { list-style-type: none; margin: 0; padding: 0; }
      #messages li { padding: 5px 10px; }
      #messages li:nth-child(odd) { background: #eee; }
    </style>
  </head>
  <body>
    <ul id="messages"></ul>
    <form action="">
      <input id="m" autocomplete="off" /><button>发送</button>
    </form>
  </body>
</html>
```

保存。

### 根目录跳转

前面我们在`server.js`中写了根目录返回hello world，现在我们让它跳转到聊天室页面。将`server.js`中的


```
app.get('/', function(req, res){
  res.send('<h1>Hello world</h1>');
});
```

替换为


```
app.get('/', function(req, res){
  res.sendFile(__dirname + '/index.html');
});
```

然后在CMD中ctrl+c结束，重新运行`node server.js`。
现在在浏览器中访问`localhost:3000`，已经跳转到聊天页面了。

![跳转之后的页面](http://oqapmzmc9.bkt.clouddn.com/%E6%97%A0%E6%A0%87%E9%A2%98.png)


## 引入Socket

继续修改server.js，增加下方标注内容：


```
var app = require('express')();
var http = require('http').Server(app);
//new addition
var io = require('socket.io')(http);
//end

app.get('/', function(req, res){
  res.sendFile(__dirname + '/index.html');
});

//new addition
io.on('connection', function(socket){
  console.log('a user connected');
});
//end

http.listen(3000, function(){
  console.log('listening on *:3000');
});
```

新加入的内容的意思就是，当有新的socket连接成功之后，就打印一下信息。

然后编辑`inde.js`，添加下方标注内容：


```
<!doctype html>
<html>
  <head>
    <title>Socket.IO chat</title>
    <style>
      * { margin: 0; padding: 0; box-sizing: border-box; }
      body { font: 13px Helvetica, Arial; }
      form { background: #000; padding: 3px; position: fixed; bottom: 0; width: 100%; }
      form input { border: 0; padding: 10px; width: 90%; margin-right: .5%; }
      form button { width: 9%; background: rgb(130, 224, 255); border: none; padding: 10px; }
      #messages { list-style-type: none; margin: 0; padding: 0; }
      #messages li { padding: 5px 10px; }
      #messages li:nth-child(odd) { background: #eee; }
    </style>
    //new addition
    <script src="http://apps.bdimg.com/libs/jquery/2.1.4/jquery.min.js"></script>
    //end
  </head>
  <body>
    <ul id="messages"></ul>
    <form action="">
      <input id="m" autocomplete="off" /><button>发送</button>
    </form>
  </body>

  //new addition
  <script src="/socket.io/socket.io.js"></script>
  <script>
    var socket = io()
  </script>
  //end
</html>
```

在上面的内容中，引入了jQuery与socket.io，并且在页面加载完成之后，新建了一个io对象。

然后我们重新启动node服务，访问`localhost:3000`。就会有以下结果。

![image](http://oqapmzmc9.bkt.clouddn.com/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20180507162039.png)


### 添加交互
当用户有动作的时候，我们要显示在聊天面板中，就插入一个`li`标签好了。在`index.html`的script中写一个方法，后面显示什么内容，就调用这个方法在面模板中插入一条`li`就好了。


将以下代码写到`index.html`中。
```
function addLine(msg) {
  $('#messages').append($('<li>').text(msg));
}
```

### 等等，我们好像忘了什么事情
在前面功能分析的时候，我们就说用户刚进来的时候，需要输入自己的昵称。
我们现在就先把这个功能实现一下。

在`index.html`的script最前面，我们先弹出一个`prompt()`来询问用户的昵称，然后将用户名发送给后端，让后端告诉大家这个用户进来了接。

### 发送昵称给后端
新用户发送自己的昵称给服务器后，要让所有处在聊天室的人知道，所以服务器要发一个广播。发送数据给后端，调用socket的emit方法，这里指定一个事件名字叫`join`，然后后端会处理这个事件。

然后现在我们的script标签中是这样的。

```
<script>
    var name   = prompt("请输入你的昵称：");
    var socket = io()
    
    //发送昵称给后端
    socket.emit("join", name)
    function addLine(msg) {
        $('#messages').append($('<li>').text(msg));
    }
</script>
```

### 后端发送广播
为了在后端区分用户，我们用一个`usocket`数组来保存每一个用户的`socket`实例。

首先监听`join`事件，在接收到昵称之后，将该用户的加入聊广播给所有用户。
`server.js`代码如下：

> 为了限制篇幅长度，从此处开始贴上来的代码有省略，请参考上下文确定位置。


```
var usocket = []; //全局变量
...
io.on('connection', function(socket){
  console.log('a user connected')

  //监听join事件
  socket.on("join", function (name) {
    usocket[name] = socket
    io.emit("join", name) //服务器通过广播将新用户发送给全体群聊成员
  })
});
...
```

然后在`index.html`的script中添加以下代码，接受到新用户加入事件的处理：


```
...
//发送昵称给后端
socket.emit("join", name)

//收到服务器发来的join事件时
socket.on("join", function (user) {
  addLine(user + " 加入了群聊")
})
...
```

现在我们再来测试一下，`node server.js`开启后端服务，然后访问`localhost:3000`，输入用户名iimT

![image](http://oqapmzmc9.bkt.clouddn.com/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20180513142157.png)

然后可以看到

![image](http://oqapmzmc9.bkt.clouddn.com/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20180513142209.png)

然后新开一个标签页，输入用户名iimY

![image](http://oqapmzmc9.bkt.clouddn.com/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20180513143144.png)

在iimY的面板中，可以看到

![image](http://oqapmzmc9.bkt.clouddn.com/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20180513143207.png)

再回到iimT的面板中，看到新提示的iimY加入了群聊

![image](http://oqapmzmc9.bkt.clouddn.com/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20180513143213.png)

### 临时加个小功能

为了更好的辨识每个面板是谁，我们在用户输入自己的昵称之后，将网页标题改成`XXX的聊天`。

只需要新加一行代码：


```
...
//发送昵称给后端，并更改网页title
socket.emit("join", name)
document.title = name + "的群聊" //new addition
...
```

## 处理发送和接受消息


### 【一】index.html 将用户消息发送给服务器
用户输入消息，然后点击发送我们将用户的消息发送给服务器，然后服务器将消息广播给所有的用户。

我们需要给`form`的提交绑定一个事件，让它来处理新消息的发送。


```
$('form').submit(function () {
    //solve code
})
```
在绑定的事件中，我们需要逐步做以下事情：

1. 获取用户输入的消息。

```
var msg = $("#m").val()
```
2. 发送事件给服务器，事件名就叫`message`吧，发送的数据就是`msg`。

```
socket.emit("message", msg) //将消息发送给服务器
```
3. 将消息输入框置空。

```
$("#m").val("") //置空消息框
```
4. 阻止`form`的提交事件。

```
return false //阻止form提交
```

最后，我们给`form`绑定的事件如下：

```
...
//当发送按钮被点击时
$('form').submit(function () {
  var msg = $("#m").val() //获取用户恮的信息
  socket.emit("message", msg) //将消息发送给服务器
  $("#m").val("") //置空消息框
  return false //阻止form提交
})
...
```

### 【二】server.js 服务器接受消息

只需要两步：
1. 监听前端的`message`事件
2. 在监听到事件之后，将新消息广播给全体用户，这里也将广播事件的名字叫`message`吧。

代码如下：
```
io.on('connection', function(socket){
  console.log('a user connected')

  socket.on("join", function (name) {
    usocket[name] = socket
    io.emit("join", name)
  })
  
  //new addition
  socket.on("message", function (msg) {
    io.emit("message", msg) //将新消息广播出去
  })
});
```

### 【三】index.html 客户接受新消息

客户接收到服务器发来的`message`事件，将新消息呈现在面板中。

```
...
socket.on("join", function (user) {
  addLine(user + " 加入了群聊")
})

//接收到服务器发来的message事件
socket.on("message", function(msg) {
  addLine(msg)
})

//当发送按钮被点击时
$('form').submit(function () {
  var msg = $("#m").val() //获取用户输入的信息
...
```

> 整个过程相当于，用户a将自己的消息用`message`事件发送给服务器，服务器监听`message`事件接收其中的消息，将消息用事件`message`广播给全体用户，全体用户监听`message`事件，将事件接受到的消息呈现在聊天框中。PS:这一段是整个socket编程的核心，可以多读几遍细细理解。


## 最终测试 
那么现在我们就完成了整个聊天室的功能，来测试一下吧。

#### 1. 开启服务器`node server.js`。

![image](http://oqapmzmc9.bkt.clouddn.com/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20180507154233.png)

#### 2. 开一个标签，输入地址`localhost:3000`，回车，填写昵称为`iimT`。

![image](http://oqapmzmc9.bkt.clouddn.com/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20180513151443.png)

#### 3. 再开一个标签，输入地址`localhost:3000`，回车，填写昵称为`iimY`。

![image](http://oqapmzmc9.bkt.clouddn.com/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20180513151907.png)

然后开始让两个人互相发送消息吧。我这里用一张动态图来显示最终效果，也就是本文最开头的那张效果图。

PS: 图大概有4M，所以需要点时间加载
![image](http://oqapmzmc9.bkt.clouddn.com/node-chat%20%281%29.gif)

至此，聊天室以就做完了。

## 最后

#### 源代码地址：[tfh93121/node-chat](https://github.com/tfh93121/node-chat)



> #### 我是iimT, 一个固执的技术直男。
>#### 我的微博 : @_iimT 
>#### 我的微信公众号 : iimT 　　个人博客： [www.iimt.me](https://www.iimt.me/)
