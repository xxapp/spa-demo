# 前端开发文档
用到的框架：avalonjs、mmState、beyond admin ui

以及它们所依赖的库和插件

## 目录结构
```
- components                // 组件，这里认为页面全部由组件构成
    - demo                  // 一个业务模块，按需加载
        - demo.html         // 业务模块html结构
        - demo.js           // 业务模块js逻辑
    + header                // 头部区域
    + index                 // 主页入口
    + sidebar               // 左侧导航区域
    + ms-control-text       // UI组件，以后可能会移出这里，作为第三方模块使用
    + bbs-channel           // 业务模块命名示例，为了与公共模块区分，可以给模块名称加前缀表示业务
+ node_modules              // nodejs模块包括开发工具和第三方插件，**这个文件不要提交**
- services                  // 公用服务，供各个业务模块调用
    - ajaxService.js        // 与后端服务通信的封装服务
    - menuService.js        // 主要包括菜单的定义和根据权限进行的菜单过滤
    - storeService.js       // 定义系统的数据源，与后台接口的交互主要写在这里面
    - configService.js      // 系统通用配置
    - ...
+ static                    // 非模块化的静态资源，如jquery、bootstrap、公共css
+ vendor                    // 模块化的静态资源，如avalon、mmState
- index.html
```

## 与服务端通信
在services/ajaxService.js中封装了一个ajax方法，与jQuery使用方法一致

注意：**调用的时候可以不用考虑请求报错的情况**
``` js
var ajax = require('/services/ajaxService');
ajax({
    url: '/start/backend/region_list.do',
    type: 'get',
    data: {
        start: 0,
        limit: 10
    }
}).then(function (result) {
    beyond.hideLoading();
    demo.list = result.list;
});
```

## 增加一个业务模块
1. 定义路由

   状态是描述了一个页面如何展示，每个模块都基于root

   在components/index.js中其它状态定义的末尾添加状态定义，以下是demo模块的定义
   ``` js
    // 定义模块状态
    avalon.state('root.demo', {
        url: 'demo', // host:port/#!/demo
        views: { // 此状态下的各个视图区域的显示设置，root中已经定义了公共视图区域
            "content@": { // 进入demo页只需要改变内容区，所以只配置了这一项
                templateProvider: require.async('/components/demo', 'view'), // 按需加载模块模板
                controllerProvider: require.async('/components/demo', 'controller') // 按需加载模块逻辑
            }
        },
        ignoreChange: function (type) {
            return !!type;
        }
    });
   ```

2. 添加菜单

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

3. 实现模块

   在components目录下新建文件夹，如demo，再在新建的文件夹下新建文件

   简单的增删改查需要新建2个文件，demo.html demo.js

   html后缀的文件为模板，声明了avalon的ms-controller

   js后缀的文件，包括avalon viewmodel和组件配置的定义，导出view和controller模块，controller中包含了当前页面的生命周期

   具体使用方法见demo内容