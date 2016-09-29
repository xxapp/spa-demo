# 前端调试文档

## 首次调试需要做的准备
* 安装环境

    [Nodejs V4.2.1](https://nodejs.org/download/release/v4.2.1/)

* 安装构建工具[FIS](http://fis.baidu.com)：

    Nodejs安装成功后，在控制台执行以下命令
    ``` bash
    npm install fis3@3.4.13 -g
    ```
* 安装项目的第三方依赖

    进入到前端项目目录，在控制台执行安装命令
    ``` bash
    # 目录路径根据实际情况调整
    cd /code/pcadmin
    npm install
    ```

* 寻找eclipse的web部署路径

    在Eclipse中打开Run Configurations对话框，并找到tomcat的wtp.deploy参数的值，如下图所示：

    ![](https://github.com/xxapp/spa-demo/raw/master/avalon-mmstate/QQ%E6%88%AA%E5%9B%BE20160922002631.png)

    找到的值有可能是这样的：```/Users/admin/Documents/workspace/.metadata/.plugins/org.eclipse.wst.server.core/tmp0/wtpwebapps```

    这个值之后调试的时候要用到，所以要记下来。如果你觉得这个值太长，可以把它添加到系统全局变量里面，这里设置为ECLIPSE_DEPLOY

## 每次调试前的准备
1. 在Eclipse中启动server
2. 启动FIS构建工具

    进入到前端项目目录，在控制台执行FIS的发布命令：
    ``` bash
    # for windows
    fis3 release local-server -w -d %ECLIPSE_DEPLOY%\privilege\pcadmin
    # for unix like
    fis3 release local-server -w -d $ECLIPSE_DEPLOY/privilege/pcadmin

    # 注意：部署路径下面的项目路径可能需要根据实际情况调整
    ```
3. 浏览器访问```http://localhost:8080/start/pcadmin```
4. 每次修改源代码后直接刷新浏览器，不用重新执行以上步骤，FIS会自动执行部署过程
