# Deployment Document

> 由于运行环境为Windows Server 2012，不支持 Docker 的虚拟化部署，因此没有使用 Docker
>
> 部署环境要求不严格



## 部署机器：

* 服务器：Windows Server 2012 R2 数据中心版
  
  * IP：139.192.79.198
  
  

## 环境：

* MySQL：5.5.62 Windows x64

  * MySQL Workbench：8.0.15
* Nodejs：10.1.0

  * npm：5.6.0
* Redis：3.2.160



## 部署步骤

### 安装软件

1. [安装node.js](<https://nodejs.org/zh-cn/>)

2. 安装npm（一般会和node.js一起安装）

   > **可选：** 修改npm源
   >
   > 由于众所周知的原因，直接访问国外的npm源会比较慢，推荐使用国内镜像，比如 taobao 镜像
   >
   > [可参照此博文](<https://blog.csdn.net/quuqu/article/details/64121812>)
   >
   > （推荐使用 npm 换源而不是 cnpm，并且不推荐npm和cnpm混用）

3. 安装redis：[官网地址](<https://redis.io/download>)，后台运行即可



### 后端

#### 1. 拉取后端源代码

* [下载并安装git](<https://git-scm.com/downloads>)，然后使用 git 命令

    ```bash
    $ git clone https://github.com/hhhghh/Happy-spare-money-back-end.git
    ```
    
* **或**在github上直接下载代码，仓库地址：
	https://github.com/hhhghh/Happy-spare-money-back-end

#### 2. 安装依赖

在根目录执行

```bash
$ npm install # 或 cnpm install
```

安装所有的依赖，如果遇到问题，可以把根目录下的node_modules文件夹删除，重新`npm install`

#### 3. 修改数据库的访问地址及密码

​	在根目录的 `config/db.js`文件中修改

#### 4. 运行程序

```bash
$ npm start # 或 node bin/www
```

此时程序默认运行在 3000 端口

> 注意此时只是开启了 API 服务，前端页面还未加载

#### 5. 运行可选服务

​	目前只有一个：**scheduler**

* 运行**scheduler**，会开启后台的定期检测，对过期的任务进行处理

  * 开启命令：`npm run scheduler`




### 前端

#### 1. 拉取前端源代码

- [下载并安装git](<https://git-scm.com/downloads>)，然后使用 git 命令

  ```
  $ git clone https://github.com/hhhghh/Happy-spare-money-front-end.git
  ```

- **或**在github上直接下载代码，仓库地址：
  https://github.com/hhhghh/Happy-spare-money-front-end

#### 2. 安装依赖

```
$ npm install
```

​	安装完成后，可以打开`npm run dev`测试，查看是否存在界面

#### 3. 生成静态页面

```
$ npm run build
```

​	生成的文件存放在 `dist` 文件夹中

#### 4. 整合前端界面

- 将生成的 `dist` 文件夹中的内容（`index.html`和`static`文件夹），复制到后端根目录的`static`文件夹下
- 重启后端服务（`ctrl+C` 停止，重新运行 `npm start`）
- 访问`http://localhost:3000/index.html/login`，查看界面是否正常