# 线上部署 - 使用宝塔面板

* [宝塔面板官网](https://www.bt.cn/)

推荐使用`Linux`系服务器
## 1. 安装宝塔面板
宝塔面板介绍安装教程：https://www.bt.cn/

### 通过面板安装必要的软件
* Nginx
* PM2
* 数据库
  * redis
  * mysql
  * MongoDB
* phpMyAdmin
* PHP

![图片](https://img.cdn.sugarat.top/mdImg/MTY0NzQ4MzMwMzg2OQ==647483303869)

安装完后，可以把常用的加至首页

![图片](https://img.cdn.sugarat.top/mdImg/MTY0NzQ4MzM3ODE4Mw==647483378183)

## 2. 安装必要依赖
### 安装Node

使用PM2面板安装，

![图片](https://img.cdn.sugarat.top/mdImg/MTY0NzQ4MzQ0MjUzMg==647483442532)

此部分的`shell`指令无特殊说明，都是在宝塔面板的终端工具中运行

![图片](https://img.cdn.sugarat.top/mdImg/MTY0NzYxMjU3OTk2NA==647612579964)

测试是否正常安装，终端工具运行如下指令
```shell
node -v
```

建议安装大于等于`14.8`版本的

### 切换镜像源
其中`npm`是随Node一起安装的包管理工具，通过切换到国内的镜像源，有助于加快安装速度

安装`nrm`（切换镜像源工具）
```shell
npm i -g nrm --registry=https://registry.npmmirror.com
```

查看可用源列表
```shell
nrm ls
```

切换`淘宝源`
```shell
nrm use taobao
```

验证是否切换成功
```shell
npm config get registry
```

结果是上述淘宝源即可

### 安装PNPM
#### 方式1
使用 `npm` 安装
```shell
npm install -g pnpm
```

#### 其它方式
参看 [pnpm官方中文文档](https://pnpm.io/zh/installation)

#### 验证是否安装成功
```shell
pnpm -v
```

## 3. 部署网站
### 构建产物
参考[本地部署-启动客户端](./local.md#_3-启动客户端)

在完成依赖安装后,执行`build`,构建产物
```shell
pnpm build
```

![图片](https://img.cdn.sugarat.top/mdImg/MTY0NzYxMjkxMzU1MA==647612913550)

此时构建产物，都在项目的`dist`目录中

![图片](https://img.cdn.sugarat.top/mdImg/MTY0NzYxMjk3OTkzOQ==647612979939)

### 创建网站
点击`添加站点`

![图片](https://img.cdn.sugarat.top/mdImg/MTY0NzYxMzAzMTExMg==647613031112)

输入要绑定的域名，选择纯静态，没有域名可以联系我，给你绑定个`sugarat.top`下的3||4级域名

或者直接使用IP访问，这里就输入`你的机器IP加一个端口号`,格式`ip:port`，例如`39.156.66.18:3333`

![图片](https://img.cdn.sugarat.top/mdImg/MTY0NzYxMzI5NTU2NQ==647613295565)

#### 使用IP+端口注意事项
需在自己服务器的`防火墙`中开放使用的端口

例如`笔者`的腾讯云

![图片](https://img.cdn.sugarat.top/mdImg/MTY0NzY5Mjk1NDI4OA==647692954288)

### 上传产物
点击前往创建的目录

![图片](https://img.cdn.sugarat.top/mdImg/MTY0NzYxMzQ3OTEwNQ==647613479105)

点击上传，将我们刚才构建生成的`dist`目录拖到上传面板

![图片](https://img.cdn.sugarat.top/mdImg/MTY0NzYxMzU1NDU4Mw==647613554583)

上传完成

![图片](https://img.cdn.sugarat.top/mdImg/MTY0NzYxMzY2MzU3Mw==647613663573)

### 修改网站访问目录

![图片](https://img.cdn.sugarat.top/mdImg/MTY0NzYxMzc3ODEwNA==647613778104)

访问 `http://ep.test.sugarat.top`测试,就看到咱们的前端应用了

### 添加配置防止路由404
在对应网站设置面板，点击`配置文件`,加入以下配置

```conf
# vue-router
location / {
   try_files $uri $uri/ /index.html;
}
```

![图片](https://img.cdn.sugarat.top/mdImg/MTY0NzY5MzExMDgyMw==647693110823)


### 开启HTTPS(可选)
为网站添加`SSL`证书

![图片](https://img.cdn.sugarat.top/mdImg/MTY0NzY1Mzc4OTkzNQ==647653789935)

宝塔提供免费SSL证书点击申请即可

申请完然后点击对应证书的部署按钮即可

部署完成后，可以点击右上角开启`强制HTTPS`

## 4. 创建MySQL数据库
### 新增数据库
在数据库面板，点击添加数据库

![图片](https://img.cdn.sugarat.top/mdImg/MTY0NzY1MjU0MDYwNg==647652540606)

**密码使用随机的，输入数据库名字即可（账号默认和数据库名一致）**

**数据库名记得小写**
### 导入表结构
管理创建的数据库

![图片](https://img.cdn.sugarat.top/mdImg/MTY0NzY1MjY5MjQzNA==647652692434)

跳转到`phpMyAdmin`面板,选择导入

![图片](https://img.cdn.sugarat.top/mdImg/MTY0NzY1Mjg2NDcxNw==647652864717)

选择服务端项目中 [docs/sql/auto_create.sql](https://github.com/ATQQ/easypicker2-server/blob/master/docs/sql/auto_create.sql) 进行上传

![图片](https://img.cdn.sugarat.top/mdImg/MTY0NzY1MzA2NjE5MQ==647653066191)

选择文件后点击执行

![图片](https://img.cdn.sugarat.top/mdImg/MTY0NzY1MzE2OTExNw==647653169117)

再次来到结构面板，即可看到完成了 6 张表的创建

![图片](https://img.cdn.sugarat.top/mdImg/MTY0NzY1MzI1MDUzNQ==647653250535)

## 5. 部署后端服务
### 本地构建源码
参照[本地启动-后端服务](./local.md#_5-启动后端服务)，进行依赖安装和构建

此部分是在本地项目中进行
```shell
pnpm install
```

```shell
pnpm build
```

![图片](https://img.cdn.sugarat.top/mdImg/MTY0NzYxNDEwMTIyMw==647614101223)

### 上传构建产物
咱们再上面创建网站的目录下，创建一个`server`目录

![图片](https://img.cdn.sugarat.top/mdImg/MTY0NzYxNDI4MDc2OA==647614280768)

进入这个目录，上传我们的产物以及一些配置文件（共4个文件）

![图片](https://img.cdn.sugarat.top/mdImg/MTY0NzYxNDU5MTY2Mw==647614591663)

### 安装依赖
在当前目录下打开宝塔终端工具执行

![图片](https://img.cdn.sugarat.top/mdImg/MTY0NzYxNDg5MDkyMg==647614890922)
```shell
pnpm install
```

### 修改环境变量
双击 `.env` 文件进行修改

![图片](https://img.cdn.sugarat.top/mdImg/MTY0NzYxNTIzNzAzMg==647615237032)

每个变量的释义参看源码中的 [src/types/env.d.ts](https://github.com/ATQQ/easypicker2-server/blob/master/src/types/env.d.ts)

通常情况下只需要关心一下
* 服务相关
  * SERVER_PORT: 服务启动的端口，默认3000，无特殊需求可以不修改
* MySql相关
  * MYSQL_DB_NAME: 数据库名
  * MYSQL_DB_USER: 账号
  * MYSQL_DB_PWD:  密码
* 七牛云相关：OSS - 文件存储，上传/下载文件依赖其提供服务
* 腾讯云相关：短信服务，不接入短信不用配

#### MySQL 相关
MySQL 的账号密码在数据库面板获取，即前面创建的数据库账号密码

#### 监听端口
默认启动监听端口为3000，如与其它服务有冲突可以修改`SERVER_PORT`为其它值（推荐 3000 => 65535）

#### 七牛云
参考[七牛云OSS服务创建](./qiniu.md)部分的文章，获取七牛云相关的几个环境变量

到此准备工作算完成了，接下来启动我们的后端服务即可

### 启动服务

在PM2面板点击添加项目

![图片](https://img.cdn.sugarat.top/mdImg/MTY0NzY1NTQxMTAzOQ==647655411039)

* 启动文件输入 `npm`
* 运行目录选择 前面创建的 `server`所在目录
* 项目名称随意，自己能辨别即可

然后点击启动即可


### 查看运行日志
点击对应服务的 运行/错误 查看相关日志，从面板能看到最终服务启动所在的端口

![图片](https://img.cdn.sugarat.top/mdImg/MTY0NzY1NjM4MzI3Mw==647656383273)

到此服务端启动算完成了

只差最后一步了

## 5. 配置反向代理

打开网站的设置面板，点击添加反向代理，勾选`高级功能`

![图片](https://img.cdn.sugarat.top/mdImg/MTY0NzY2Njc0Nzg4Mw==647666747883)

代理名称随便填
* 代理目录`/api/`
* 目标URL填`自己的后端服务地址`
* 内容替换`/api`,第二个留空

## 6. 其余功能
### 开启内容压缩
在网站设置面板，点击反向代理，配置文件

添加`#`注释或者删掉 `proxy_set_header Accept-Encoding "";`

![图片](https://img.cdn.sugarat.top/mdImg/MTY0NzY5MzM2MDI3OQ==647693360279)

这样返回的内容会进行压缩，响应速度会有所提升

![图片](https://img.cdn.sugarat.top/mdImg/MTY0NzY5Mzg3MDc2NA==647693870764)

### 配置管理员权限

账号加了管理员权限后，就能看到后台管理的入口
* 查看访问日志
* 管理用户账号


| 添加前                                                                      | 添加后                                                                      |
| --------------------------------------------------------------------------- | --------------------------------------------------------------------------- |
| ![图片](https://img.cdn.sugarat.top/mdImg/MTY0NzY5NDAzOTMxNg==647694039316) | ![图片](https://img.cdn.sugarat.top/mdImg/MTY0NzY5NDMxMDE1OA==647694310158) |


打开对应的数据库

![图片](https://img.cdn.sugarat.top/mdImg/MTY0NzY5NDE5NTQyMA==647694195420)

选择`user`表

![图片](https://img.cdn.sugarat.top/mdImg/MTY0NzY5NDI0NjM5Ng==647694246396)

修改账号的`power`字段值为`0`

![图片](https://img.cdn.sugarat.top/mdImg/MTY0NzY5NDE1NTczMg==647694155732)

然后重新登录账号，就能看到入口了

![图片](https://img.cdn.sugarat.top/mdImg/MTY0NzY5NDMxMDE1OA==647694310158)

大功告成

有其它问题可以小群交流，方便可以加入及时交流沟通问题: 685446473

![图片](https://img.cdn.sugarat.top/mdImg/MTY0Nzc1MjI3MzUwMw==647752273503)