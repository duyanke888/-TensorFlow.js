参考[官方文档](https://mp.weixin.qq.com/wxopen/plugindevdoc?appid=wx6afed118d9e81df9&token=468942979&lang=zh_CN)
# TensorFlow.js 微信小程序插件
## 介绍及部署
TensorFlow.js是谷歌开发的机器学习开源项目，致力于为javascript提供具有硬件加速的机器学习模型训练和部署。 TensorFlow.js 微信小程序插件封装了TensorFlow.js库，用于提供给第三方小程序调用。
### 添加插件
在使用插件前，首先要在小程序管理后台的 “设置-第三方服务-插件管理” 中添加插件。开发者可登录小程序管理后台，通过 appid [wx6afed118d9e81df9] 查找插件并添加。本插件无需申请，添加后可直接使用。
### 引入插件代码包
使用插件前，使用者要在 app.json 中声明需要使用的插件，例如：
代码示例：

    {
      ...
      "plugins": {
        "tfjsPlugin": {
          "version": "0.0.6",
          "provider": "wx6afed118d9e81df9"
        }
      }
      ...
    }

### 引入TensorFlow.js npm
TensorFlow.js 最新版本是以npm包的形式发布，小程序需要使用npm或者yarn来载入TensorFlow.js npm包。也可以手动修改 package.json 文件来加入。
TensorFlow.js有一个联合包 - @tensorflow/tfjs，包含了四个分npm包：

 - tfjs-core: 基础包
 - tfjs-converter: GraphModel 导入和执行包
 - tfjs-layers: LayersModel 创建，导入和执行包
 - tfjs-data：数据流工具包
 
对于小程序而言，由于有2M的app大小限制，不建议直接使用联合包，而是按照需求加载分包。
如果小程序只需要导入和运行GraphModel模型的的话，建议只加入tfjs-core和tfjs-converter包。这样可以尽量减少导入包的大小。
如果需要创建,导入或训练LayersModel模型，需要再加入 tfjs-layers包。
下面的例子是只用到tfjs-core和tfjs-converter包。代码示例：

    {
      "name": "yourProject",
      "version": "0.0.1",
      "main": "dist/index.js",
      "license": "Apache-2.0",
      "dependencies": {
        "@tensorflow/tfjs-core": "1.2.7"，
        "@tensorflow/tfjs-converter": "1.2.7"
      }
    }

参考小程序npm工具[文档](https://developers.weixin.qq.com/miniprogram/dev/devtools/npm.html)如何编译npm包到小程序中。
注意 请从微信小程序[开发版Nightly Build更新日志](https://developers.weixin.qq.com/miniprogram/dev/devtools/nightly.html)下载最新的微信开发者工具，保证版本号>=v1.02.1907022.
## Polyfill fetch 函数
如果需要使用tf.loadGraphModel或tf.loadLayersModel API来载入模型，小程序需要按以下流程填充fetch函数：
1 如果你使用npm, 你可以载入fetch-wechat npm 包 


    {
      "name": "yourProject",
      "version": "0.0.1",
      "main": "dist/index.js",
      "license": "Apache-2.0",
      "dependencies": {
        "@tensorflow/tfjs-core": "1.2.7"，
        "@tensorflow/tfjs-converter": "1.2.7"，
        "fetch-wechat": "0.0.3"
      }
    }


2 也可以直接拷贝以下文件到你的javascript源目录：

 [https://cdn.jsdelivr.net/npm/fetch-wechat@0.0.3/dist/fetch_wechat.min.js](https://cdn.jsdelivr.net/npm/fetch-wechat@0.0.3/dist/fetch_wechat.min.js)

### 在app.js的onLaunch里调用插件configPlugin函数

    var fetchWechat = require('fetch-wechat');
    var tf = require('@tensorflow/tfjs-core');
    var plugin = requirePlugin('tfjsPlugin');
    //app.js
    App({
      onLaunch: function () {
        plugin.configPlugin({
          // polyfill fetch function
          fetchFunc: fetchWechat.fetchFunc(),
          // inject tfjs runtime
          tf,
          // provide webgl canvas
          canvas: wx.createOffscreenCanvas()
        });
      }
    });

组件设置完毕就可以开始使用 TensorFlow.js库的API了。

### 以上均为引用，特此声明

--------------------------------------------------------------------------------
以上为TensorFlow.js微信小程序的部署工作，下面介绍一个例子。

例子：TensorFlow.js官网在配置tfjs时提供的例子: [链接](https://tensorflow.google.cn/js/tutorials/setup)
  
    import * as tf from '@tensorflow/tfjs';
    
    //定义一个线性回归模型。
    const model = tf.sequential();
    model.add(tf.layers.dense({units: 1, inputShape: [1]}));
    
    model.compile({loss: 'meanSquaredError', optimizer: 'sgd'});
    
    // 为训练生成一些合成数据
    const xs = tf.tensor2d([1, 2, 3, 4], [4, 1]);
    const ys = tf.tensor2d([1, 3, 5, 7], [4, 1]);
    
    // 使用数据训练模型
    model.fit(xs, ys, {epochs: 10}).then(() => {
      // 在该模型从未看到过的数据点上使用模型进行推理
      model.predict(tf.tensor2d([5], [1, 1])).print();
      //  打开浏览器开发工具查看输出
    });

  

在项目下新建myModule文件夹，创立module.js文件，文件内容即为以上代码。此程序是在浏览器的控制台进行输出，不直接显示在前端。
下面引用module.js。在app.js中写入以下代码：

    //导入模型
    const module = require('./myModule/module.js');

此时出现了一个问题
![输出一个错误](https://img-blog.csdnimg.cn/20190816231432109.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM0Mzc4MDMy,size_16,color_FFFFFF,t_70)

原因：忘了安装@tensorflow/tfjs

	npm install @tensorflow/tfjs

控制台能够正确输出如下图：
![正确的输出](https://img-blog.csdnimg.cn/20190816231535208.jpg)

微信小程序的TensorFlow.js完成。

---

注：控制台在输出时报如下警告
![WebGL is not supported on this device](https://img-blog.csdnimg.cn/20190816231622724.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM0Mzc4MDMy,size_16,color_FFFFFF,t_70)

  WebGL 是一种 3D 绘图标准，这种绘图技术标准允许把 JavaScript 和 OpenGL ES 2.0 结合在一起，通过增加 OpenGL ES 2.0 的一个 JavaScript 绑定，WebGL 可以为 HTML5 Canvas 提供硬件 3D 加速渲染，这样 Web 开发人员就可以借助系统显卡来在浏览器里更流畅地展示3D场景和模型了，还能创建复杂的导航和数据视觉化。
[WebGL学习](https://www.w3cschool.cn/webgl/vjxu1jt0.html)


