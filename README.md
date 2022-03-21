# webpack_bundler
Create a simple webpack bundler
## 什么是webpack
分析模块打包器webpack是怎么分析ES6的模块依赖，怎么把ES6的代码转成ES5的。

## 实现
npm babel插件

`npm install @babel/core @babel/parser @babel/traverse @babel/preset-env --save-dev`
### 文件
使用webpack会涉及三个需要打包的js文件（`entry.js`、`message.js`、`name.js`）

## 流程图
![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a52b7f076cc242e39d33011a878fb9fd~tplv-k3u1fbpfcp-watermark.image?)

### 分析依赖

webpack分析依赖是从一个入口文件开始分析的，当我们把一个入口的文件路径传入，webpack就会通过这个文件的路径读取文件的信息（读取到的本质其实是字符串），然后把读取到的信息转成AST（抽象语法树），简单点来说呢，就是把一个js文件里面的内容存到某种数据结构里，里面包括了各种信息，**其中就有当前模块依赖了哪些模块**。我们暂时把通过传**文件路径**能返回文件信息的这个函数叫 `createAsset` 。

### `createAsset`返回
第一步我们肯定需要先从 `entry.js` 开始分析  
返回 id,filename, dependencies,code 值。
```
createAsset("./example/entry.js");
```
当执行这句代码，`createAsset` 会返回下面的数据结构，这里包括了**模块的id**，**文件路径**，**依赖数组**（`entry.js`依赖了`message.js`，所以会返回依赖的文件名），**code**（这个就是`entry.js` ES6转ES5的代码）
![](https://user-gold-cdn.xitu.io/2019/3/2/1693eee846b82ac0?w=1482&h=560&f=png&s=101080)
通过 `createAsset` 我们成功拿到了`entry.js`的依赖，就是 `dependencies` 数组。

### `createGraph`返回，如何找下一个依赖
我们通过上面可以拿到entry.js依赖的模块，于是我们就可以接着去遍历`dependencies` 数组，循环调用`createAsset`这样就可以得到全部模块相互依赖的信息。想得到全部依赖信息需要调用 `createGraph` 这个一个函数，它会进行广度遍历，这里可以debug看每个变量的值

### `bundle`返回
我们现在已经能拿到每个模块之前的依赖关系，我们再通过调用`bundle`函数，我们就能构造出最后的`bundle.js`，输出如下
```javascript

    (function(modules){
      /*
      * 创建require函数， 它接受一个模块ID（这个模块id是数字0，1，2） ，它会在我们上面定义 modules 中找到对应是模块.
      */
      function require(id){
        const [fn, mapping] = modules[id];
        function localRequire(relativePath){
          //根据模块的路径在mapping中找到对应的模块id
          return require(mapping[relativePath]);
        }
        const module = {exports:{}};
        //执行每个模块的代码。
        fn(localRequire,module,module.exports);
        return module.exports;
      }
      //执行入口文件，
      require(0);
    })({0:[
      function (require, module, exports){
        "use strict";

var _message = require("./message.js");

console.log(_message.name);
      },
      {"./message.js":1},
    ],1:[
      function (require, module, exports){
        "use strict";

Object.defineProperty(exports, "__esModule", {
  value: true
});
exports["default"] = void 0;

var _name = require("./name.js");

var _default = "hello ".concat(_name.name);

exports["default"] = _default;
      },
      {"./name.js":2},
    ],2:[
      function (require, module, exports){
        "use strict";

Object.defineProperty(exports, "__esModule", {
  value: true
});
exports["default"] = void 0;
var name = 'world';
var _default = name;
exports["default"] = _default;
      },
      {},
    ],})
  
```

