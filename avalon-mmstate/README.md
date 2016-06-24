# 前端开发文档
用到的框架：avalonjs、mmState、beyond admin ui

以及它们所依赖的库和插件

## 目录结构
```
- components                // 组件，这里认为页面由组件构成
    - demo                  // 一个业务模块，按需加载
        - demo.html         // 列表html结构
        - demo.js           // 列表js逻辑
        - edit.html         // 表单
        - edit.js           // 表单，同时处理添加和修改的逻辑
    + header                // 头部区域
    + index                 // 主页入口
    + sidebar               // 左侧导航区域
+ node_modules              // nodejs模块包括开发工具和第三方插件
- services                  // 公用服务
    - ajaxService.js        // 与后端服务通信的封装服务
    - menuService.js        // 主要包括菜单的定义和根据权限进行的菜单过滤
    - ...
+ static                    // 非模块化的静态资源，如jquery、bootstrap、公共css
+ vendor                    // 模块化的静态资源，如avalon、mmState
- index.html
```

## 通信
在services/ajaxService.js中封装了一个ajax方法，调用的时候可以不用传URL等参数，也可以不用考虑请求不正确的情况，只需要配置请求method和请求参数并在then里面处理请求返回的数据。
``` js
var ajax = require('/services/ajaxService');
ajax({
    type: 'post',
    data: {
        method: 'region.list',
        page: params.query.page || 1
    }
}).then(function (result) {
    beyond.hideLoading();
    demo.list = result.list;
    //rs();
});
```

## 增加一个增删改查的模块
1. 定义状态

   状态是描述页面显示的一个对象，例如，可以把主页作为root状态，列表页为root的一个子状态root.demo，增加/修改状态又是root.demo的一个子状态root.demo.edit

   在components/index.js中其它状态定义的末尾添加状态定义
   ``` js
    // 定义列表页状态
    avalon.state('root.demo', {
        url: 'demo', // host:port/#!/demo
        views: { // 此状态下的各个视图区域的显示设置
            "content@": { // 进入列表页只需要改变内容区，所以只配置了这一项
                templateProvider: require.async('/components/demo', 'view'), // 按需加载模板
                controllerProvider: require.async('/components/demo', 'controller') // 按需加载列表页逻辑
            }
        },
        ignoreChange: function (type) {
            console.log(arguments);
            return !!type;
        }
    });
    // 定义增加/修改状态
    avalon.state('root.demo.form', {
        // 添加和修改在这里都实现为弹出框形式，所以不使用mmState的路由方法，
        // 进入增加/修改url不发生改变所以也不配置url。但注意template一定要写，切字符串不能为空（长度能为0）
        template: ' ',
        // 按需加载处理逻辑 
        controllerProvider: require.async('/components/demo/form', 'controller')
    });
   ```

*  添加菜单

   在services/menuService.js中添加菜单项，对于简单的增删改查只需要增加一个进入列表页的菜单就可以

   ``` js
    var menu = [{ // 一级菜单
        name: 'dashboard', // 菜单名称，英文，可以和stateName一样
        stateName: 'root', // 对应的状态名称
        title: '主页', // 显示在菜单上的文字
        icon: 'glyphicon-home', // 图标
        href: '#!/' // 点击后调整的路径
    }, {
        name: 'demo1',
        title: '例子一级',
        icon: 'glyphicon-home',
        href: 'javascript:;', //有子菜单的菜单一般点击不跳转
        children: [{ // 二级菜单（目前只支持到二级）
            name: 'demo',
            stateName: 'root.demo',
            title: '例子',
            icon: 'glyphicon-home',
            href: '#!/demo'
        }]
    }];
   ```

*  实现模块

   在components目录下新建文件夹，如demo，再在新建的文件夹下新建文件

   简单的增删改查需要新建4个文件，demo.html demo.js form.html form.js

   html后缀的文件为模板，声明了avalon的ms-controller

   js后缀的文件，包括avalon viewmodel的定义，导出view和controller模块，controller中包含了当前页面的生命周期

   具体使用方法见demo

   注，目前还没实现表单验证
