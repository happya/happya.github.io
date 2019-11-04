---
title: nodejs+Express+mongodb搭建个人博客体验（1）
date: 2019-11-04 13:17:16
tags:
- node.js
- database
- backend
---



## 1. 目标

这个项目主要是开发一个简单的多人博客系统，其基本功能如下： 

- 用户管理：用户可创建账号，登入和登出博客系统；且可创建多个用户，每个用户拥有自己的信息
- 文章管理： 已经登录的用户，可以发表新文章，编辑和删除已有文章，以及查询文章

## 2. 工欲善其事，必先利其器——环境搭建

### 2.1 安装nodejs

>  我是windows系统，直接去官网下载，安装即可，安装完毕后打开cmd窗口，输入npm -v，如果可以查看版本信息，说明安装完成。  

### 2.2 安装MongoDB数据库

>  同样也是去官网安装，安装完毕后在系统变量中添加mongodb安装的bin文件夹，并通过如下命令  

```
mongod --dbpath C:\mongodb\data --logpath C:\mongodb\log
```

设置数据库的数据路径和日志路径，可自己定义。然后在终端敲入mongo，即可看到数据库在27017端口启动。

<!-- more -->

![img](https://pic1.zhimg.com/v2-de502b50405eedfdb79c77e940b91368_b.png)



### 2.3 安装各种依赖的模块

>  打开cmd或者git bash，用npm install命令安装。其中：  npm install -g packagename: 全局安装  npm install packgename --save:安装包的信息保存到package.json文件  

因为这里要用到Express，所以要做如下安装:

```
npm install -g express
npm install -g express-generator
```

安装完毕后，直接生成项目框架

```
express -e nodeBlog
```

其中，nodeBlog是项目的名称，也是生成的文件夹的名称，其中包含了项目所需的各类文件。其他所需要的包也可以通过上述方法一一安装，比如上面说到的mongoDB数据库，这里也要安装对应的模块

```
npm install mongoose --save
```

### 2.4 加载所安装的模块

>  nodeJs中通过require来加载所需的模块，有点类似Python里的import。比如引入上述的express模块，即可通过下面语句实现。  

```
var express = require('express');
```

## 3. 文件结构

按照上述方法生成的框架的文件结构如下图所示：

![img](https://pic2.zhimg.com/v2-5357e75d4d0b293af5f06d46732afe95_b.png)



如图所示，项目由express框架快速搭建，整个blog项目的文件包含在nodeBlog文件夹中，其中

- bin:  存放可执行文件，express直接生成的是一个www.js文件。
- models:  这里存放的是自己模块文件，比如用到的数据模型。
- node_modules: 用到的模块，由`npm install <module name> `安装
- public：存放项目的静态资源，如css文件，图片等等
- routes: 存放路由文件，控制了整个项目各个页面的实现
- views: 存放视图文件，也就是每个页面的长相
- app.js: 项目的启动文件
- package.json: 存放项目的基本信息，如项目名，模块依赖信息等等。

在app.js中设置监听端口为8080,app.listen(8080)，在文件夹内打开终端(这里我用的git bash)，输入node app.js，在浏览器内打开localhost:8080，即可看到如下界面:

![img](https://pic1.zhimg.com/v2-e591fe2d60945ed5fcf08990e62ebaf8_b.png)



下面我们来看这个页面主要是由`routes/index.js`和`views/index.ejs`控制实现的。

## 4. 页面设计

### 4.1 路由控制

　　Express框架直接生成项目时，有一个routes文件夹，其中包含有index.js和users.js文件，其中index.js是这个项目的重中之重。routes文件夹负责对页面的路由进行控制。打开index.js文件，其中有如下形式的代码：

```
router.get('/',function(req,res) {
    res.render('index',{
        title: 'Homepage'
    });
});
```



这即是一个路由控制文件，之后我们写的负责各个页面的路由的文件均有类似的形式。其详细含义如下：

- get: http的请求方式，这个项目中只用到了get和post两种方法  
-  参数 '/': 表示路由规则，这里指向主页，比如监听端口为8080,即指向localhost:8080。如果我们将参数设为'/login' ，则表示指向localhost:8080/login页面。  
-  第二个参数function(req,res)为callback函数，其中的req和res代表请求信息(request)和相应信息(respond)。稍后会介绍这两个参数包含的具体信息和用法。  
- res.render()中传递的是一个json格式的数据，这个数据会传递给第一个参数指向的ejs文件，这里指向的是index.ejs。也就是说这个函数向index.ejs传递了一个名为title的参数，其值为Homepage。而对应地在index.ejs文件里我们通过<%= title %>可以引用这个参数。

```
 <!DOCTYPE html> 
 <html>   
         <head>      
                    <title><%= title %></title>      
                    <link rel='stylesheet' href='/stylesheets/style.css' />     
        </head>    
       <body>       
                   <h1><%= title %></h1>       
                  <p>Welcome to <%= title %></p>  
      </body>  
 </html>
```

>  这样页面就会显示为下图。  



![img](https://pic2.zhimg.com/v2-8a319b26eb20aae59d5035158b12159d_b.png)

### 4.2 ejs文件

从上图可以看出ejs文件和html很类似，除了用<% %>包裹的部分，这个项目中我们用到的ejs的语法规则也很简单，只有如下三种：

- `<% %>`: 包含的是js代码，如`if-else`
- `<%= %>` : 用于显示替换过的html特殊字符的内容，如上面的<%= title %>即显示的是由js文件中res.render()函数传递过来的title的值
- `<%- %>`：用来包含一段html的原始内容，比如所有页面通常会有一个共用的信息，比如一些包含的css文件，比如页面顶端的一些显示，为了避免在每个ejs文件里重复写，我们可以单独写一个header.ejs文件，然后在其他ejs文件里通过<%- include header %>来直接调用这个文件，其效果相当于将整个header.ejs文件复制过去。

ejs文件存放在views文件夹中，顾名思义，它们负责的是每个页面的样式的。我们在使用express建立项目时，命令里的-e事实上就是制定了使用ejs作为模板引擎。

### 4.3 req和res参数介绍

　　在这个项目中，我们只用到了get和post两种请求方式，我的理解是get，就是向我直接请求一个页面，比如说"请给我显示主页吧"，router.get()函数就会把主页的信息返回在浏览器显示出来。Post就是提交一些信息，比如注册时填写的表单，发表博客的内容等，会由route.post()函数处理。

　　需要注意的是，get方法同样可以进行表单的提交，这个在html的表单属性中会指明用哪种方法发送请求<form method=' '，可能是get也可能是post，区别在与get方法能发送的数据量有限，一般只有几百个字节，所以在这个项目中，注册信息，博客内容通过post方法提交，而查询时的字段通过get方法提交。 

　　在这些过程中，参数的获取方式如下：

- req.query: 处理get请求，获取get请求参数。
- req.body: 处理post请求
- req.params: 处理形如/:xxx形式的get或者post请求

下面举几个具体的例子生动形象地说明下怎么传递这些参数。

### 4.3.1. req.body

以登录页面(/login)为例。因为登录需要提交登录信息，这里要用到post方法，我们首先在login.ejs文件中写入如下代码：

```
<%- include header %>
<form method="post">
    <!-- 用户名输入框 -->
    <div class="form-group">
        <input type="text" class="form-control" name="username" placeholder="用户名"
    </div>
    <!-- 密码输入框 -->    
    <div class="form-group">
        <input type="password" class="form-control" name="password" placeholder="密码">
    </div>
    <!-- 提交按钮 -->      
   <button type="submit" class="btn btn-success center-block">Login</button>
</form>
<%- include footer %>
```

　　　可以看出，这里建立了用户名和密码输入框，以及一个提交按钮，其中用到的class:form-group, form-control, btn btn-success等都是用到了bootstrap里的样式。那么在对应的路由控制文件里怎么接受这些参数呢？这里请求了用户名和密码两个参数，名称分别为username和password(由name属性命名)，那么在router.post('/login',callback)函数里就可这样获得请求体：

```
var username = req.body.username;
var password = req.body.password;
```

### 4.3.2. req.query

再比如，我们在页面顶端定义了一个搜索框：

```
<form class="navbar-form navbar-right" role="search" action="/search" method="get">
    <!-- 搜索框 -->
    <div class="form-group">
        <input type="text" class="form-control" name="search" placeholder="Search">
    </div>
    <!-- 搜索提交按钮 -->
    <button type="submit" class="btn btn-default">search</button>                   
</form>
```

　　这里表单提交的请求方法为get，输入的搜索信息的name为search，那么我们在对应的路由控制函数，如router.get('/search',callback)中，就可以这样获得请求体：

```
var query=req.query.search;
```

### 4.3.3. req.params

　　比如我们想通过博客文章的id来查看文章详情，并通过/detail/:_id来访问，这里的_id是建立文章的时候生成的，比如我新建一篇文章，数据库里会新添一条记录



![img](https://pic4.zhimg.com/v2-b6c6232412582e8da3c6f15608d1a9d3_b.png)

而我们想通过这个_id访问文章，即可通过req.params._id获得对应的请求参数。

### 4.2 页面逻辑

根据第一部分的目标分析，我们目前需要设计如下的一些页面：

![img](https://pic1.zhimg.com/v2-751daf946fa91eadd07656e8ab644b34_b.png)

- /:    主页，由views/index.ejs负责
- /login: 登录页面，由views/login.ejs负责，需要构建一个form表单，填写用户名，密码信息
- /reg： 注册页面，由views/register.ejs负责，需要构建一个form，填写用户名、密码、重复密码、邮箱信息
- /post:  文章发表页面，由views/post.ejs负责，需要一个form，填写标题，作者，标签，创建时间，博客正文信息。
- /edit: 文章编辑页面，由views/edit.ejs负责，其表单和post页面一致，需要自动填写文章原有信息，方便修改。
- /search搜索结果页面，由views/search.ejs负责，布局和主页一致
- header.ejs和footer.ejs为其他页面共享。

上述结果的路由规则均在routes/index.js文件中通过router.get()和router.post()函数指定。

### 4.4 连接数据库

　　之前我们已经安装了Mongodb和专为NodeJs设计的模型工具mongoose，这里我们先在app.js中引入mongoose模块：

```
var mongoose=require('mongoose');
```

指定连接的数据库

```
var DB_URL='mongodb//localhost:27017/blogs';
```

　　其中blogs是我们使用的数据库的名字，在打开的终端中输入mongo后，通过use blogs命令可以切换到该数据库，然后通过show collections可以看数据库中的所有集合，目前因为我们还没创建数据，所以是空的。

然后，继续在app.js中连接数据库

```
mongoose.connect(DB_URL,{useMongoClient:true});
//连接成功
mongoose.connection.on('connected',function(){
    console.log('connect success '+DB_URL);
});
//连接失败
mongoose.connection.on('error',function(err) {
    console.log('connect error '+err);
});
```

### 4.5 设置数据模型

　　我们的博客系统主要有两类数据：用户和文章，需要建立这两类数据模型。在这里，我们使用mongoose的schema模型对象来创建数据模型，模型文件存放在models/model.js中。首先我们引入mongoose模块并定义schema模型对象：

```
var mongoose=require('mongoose');
var Schema=mongoose.Schema;
```

用户类数据具有用户名，密码，邮箱，创建时间几个属性：

```
var userSchema=new Schema({
    username: String,
    password: String,
    email: String,
    createTime: {
        type: Date,
        default: Date.now()
    }
});
```

文章类数据有标题，标签，内容，作者，创建时间，浏览次数几个属性：

```
var articleSchema= new Schema({
    title: String,
    author: String,
    tag: String,
    content: String,
    createTime: {
        type: Date,
        default: Date.now()
    },
    pv: {
        type: Number,
        default: 0
    }
});
```

导出模块供其他文件调用

```
exports.User=mongoose.model('User',userSchema);
exports.Article=mongoose.model('Article',articleSchema);
```

在routes/index.js中调用创建好的数据模型：

```
var model=require('../models/model')
var User=model.User;
var Article=model.Article;
```


