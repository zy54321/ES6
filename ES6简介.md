它的目标是使得 JavaScript 语言可以用来编写复杂的大型应用程序，成为企业级开发语言。

[toc]
### 1.ECMAScript 和 JavaScript 的关系
ECMAScript 和 JavaScript 的关系是，前者是后者的规格，后者是前者的一种实现（另外的 ECMAScript 方言还有 Jscript 和 ActionScript）。日常场合，这两个词是可以互换的。

### 2.ES6 与 ECMAScript 2015 的关系
ES6 既是一个历史名词，也是一个泛指，含义是 5.1 版以后的 JavaScript 的下一代标准，涵盖了 ES2015、ES2016、ES2017 等等，而 ES2015 则是正式名称，特指该年发布的正式版本的语言标准。本书中提到 ES6 的地方，一般是指 ES2015 标准，但有时也是泛指“下一代 JavaScript 语言”。

### 3.部署进度
各大浏览器的最新版本，对 ES6 的支持可以查看kangax.github.io/es5-compat-table/es6/。随着时间的推移，支持度已经越来越高了，超过 90%的 ES6 语法特性都实现了。

Node 是 JavaScript 的服务器运行环境（runtime）。它对 ES6 的支持度更高。除了那些默认打开的功能，还有一些语法功能已经实现了，但是默认没有打开。使用下面的命令，可以查看 Node 已经实现的 ES6 特性。
```
$ node --v8-options | grep harmony
```
上面命令的输出结果，会因为版本的不同而有所不同。

我写了一个工具 ES-Checker，用来检查各种运行环境对 ES6 的支持情况。访问ruanyf.github.io/es-checker，可以看到您的浏览器支持 ES6 的程度。运行下面的命令，可以查看你正在使用的 Node 环境对 ES6 的支持程度。
```
$ npm install -g es-checker
$ es-checker

=========================================
Passes 24 feature Dectations
Your runtime supports 57% of ECMAScript 6
=========================================
```

### 4.Babel 转码器
Babel 是一个广泛使用的 ES6 转码器，可以将 ES6 代码转为 ES5 代码，从而在现有环境执行。这意味着，你可以用 ES6 的方式编写程序，又不用担心现有环境是否支持。下面是一个例子。
```
// 转码前
input.map(item => item + 1);

// 转码后
input.map(function (item) {
  return item + 1;
});
```
上面的原始代码用了箭头函数，Babel 将其转为普通函数，就能在不支持箭头函数的 JavaScript 环境执行了。
#### 4.1配置文件.babelrc
Babel 的配置文件是.babelrc，存放在项目的根目录下。使用 Babel 的第一步，就是配置这个文件。

该文件用来设置转码规则和插件，基本格式如下。
```
{
  "presets": [],
  "plugins": []
}
```
presets字段设定转码规则，官方提供以下的规则集，你可以根据需要安装。
```
# 最新转码规则
$ npm install --save-dev babel-preset-latest

# react 转码规则
$ npm install --save-dev babel-preset-react

# 不同阶段语法提案的转码规则（共有4个阶段），选装一个
$ npm install --save-dev babel-preset-stage-0
$ npm install --save-dev babel-preset-stage-1
$ npm install --save-dev babel-preset-stage-2
$ npm install --save-dev babel-preset-stage-3
```
然后，将这些规则加入.babelrc。
```
  {
    "presets": [
      "latest",
      "react",
      "stage-2"
    ],
    "plugins": []
  }
```
注意，以下所有 Babel 工具和模块的使用，都必须先写好.babelrc。
#### 4.2命令行转码babel-cli
Babel 提供babel-cli工具，用于命令行转码。

它的安装命令如下。

```
$ npm install --global babel-cli
```

基本用法如下。

```
# 转码结果输出到标准输出
$ babel example.js

# 转码结果写入一个文件
# --out-file 或 -o 参数指定输出文件
$ babel example.js --out-file compiled.js
# 或者
$ babel example.js -o compiled.js

# 整个目录转码
# --out-dir 或 -d 参数指定输出目录
$ babel src --out-dir lib
# 或者
$ babel src -d lib

# -s 参数生成source map文件
$ babel src -d lib -s
```

上面代码是在全局环境下，进行 Babel 转码。这意味着，如果项目要运行，全局环境必须有 Babel，也就是说项目产生了对环境的依赖。另一方面，这样做也无法支持不同项目使用不同版本的 Babel。

一个解决办法是将babel-cli安装在项目之中。
```
# 安装
$ npm install --save-dev babel-cli
```
然后，改写package.json。
```
{
  // ...
  "devDependencies": {
    "babel-cli": "^6.0.0"
  },
  "scripts": {
    "build": "babel src -d lib"
  },
}
```
转码的时候，就执行下面的命令。
```
$ npm run build
```
#### 4.3babel-node
babel-cli工具自带一个babel-node命令，提供一个支持 ES6 的 REPL 环境。它支持 Node 的 REPL 环境的所有功能，而且可以直接运行 ES6 代码。

它不用单独安装，而是随babel-cli一起安装。然后，执行babel-node就进入 REPL 环境。
```
$ babel-node
> (x => x * 2)(1)
2
```
babel-node命令可以直接运行 ES6 脚本。将上面的代码放入脚本文件es6.js，然后直接运行。

```
$ babel-node es6.js
2
```
babel-node也可以安装在项目中。
```
$ npm install --save-dev babel-cli
```
然后，改写package.json。

```
{
  "scripts": {
    "script-name": "babel-node script.js"
  }
}
```
上面代码中，使用babel-node替代node，这样script.js本身就不用做任何转码处理

#### 4.4babel-register
babel-register模块改写require命令，为它加上一个钩子。此后，每当使用require加载.js、.jsx、.es和.es6后缀名的文件，就会先用 Babel 进行转码。

```$ npm install --save-dev babel-register```

使用时，必须首先加载babel-register。
```
require("babel-register");
require("./index.js");
```
然后，就不需要手动对index.js转码了。

需要注意的是，babel-register只会对require命令加载的文件转码，而不会对当前文件转码。另外，由于它是实时转码，所以只适合在开发环境使用。

#### 4.5babel-core
如果某些代码需要调用 Babel 的 API 进行转码，就要使用babel-core模块。

安装命令如下。
```
$ npm install babel-core --save
```
然后，在项目中就可以调用babel-core。
```
var babel = require('babel-core');

// 字符串转码
babel.transform('code();', options);
// => { code, map, ast }

// 文件转码（异步）
babel.transformFile('filename.js', options, function(err, result) {
  result; // => { code, map, ast }
});

// 文件转码（同步）
babel.transformFileSync('filename.js', options);
// => { code, map, ast }

// Babel AST转码
babel.transformFromAst(ast, code, options);
// => { code, map, ast }
```
配置对象options，可以参看官方文档http://babeljs.io/docs/usage/options/。

下面是一个例子。
```
var es6Code = 'let x = n => n + 1';
var es5Code = require('babel-core')
  .transform(es6Code, {
    presets: ['latest']
  })
  .code;
// '"use strict";\n\nvar x = function x(n) {\n  return n + 1;\n};'
```
上面代码中，transform方法的第一个参数是一个字符串，表示需要被转换的 ES6 代码，第二个参数是转换的配置对象。

#### 4.6babel-polyfill
Babel 默认只转换新的 JavaScript 句法（syntax），而不转换新的 API，比如Iterator、Generator、Set、Map、Proxy、Reflect、Symbol、Promise等全局对象，以及一些定义在全局对象上的方法（比如Object.assign）都不会转码。

举例来说，ES6 在Array对象上新增了Array.from方法。Babel 就不会转码这个方法。如果想让这个方法运行，必须使用babel-polyfill，为当前环境提供一个垫片。

安装命令如下。
```
$ npm install --save babel-polyfill
```
然后，在脚本头部，加入如下一行代码。
```
import 'babel-polyfill';
// 或者
require('babel-polyfill');
```
Babel 默认不转码的 API 非常多，详细清单可以查看babel-plugin-transform-runtime模块的definitions.js文件。

#### 4.7浏览器环境
Babel 也可以用于浏览器环境。但是，从 Babel 6.0 开始，不再直接提供浏览器版本，而是要用构建工具构建出来。如果你没有或不想使用构建工具，可以使用babel-standalone模块提供的浏览器版本，将其插入网页。
```
<script src="https://cdnjs.cloudflare.com/ajax/libs/babel-standalone/6.4.4/babel.min.js"></script>
<script type="text/babel">
// Your ES6 code
</script>
```
注意，网页实时将 ES6 代码转为 ES5，对性能会有影响。生产环境需要加载已经转码完成的脚本。

下面是如何将代码打包成浏览器可以使用的脚本，以Babel配合Browserify为例。首先，安装babelify模块。
```
$ npm install --save-dev babelify
```
babel-preset-latest
然后，再用命令行转换 ES6 脚本。
```
$  browserify script.js -o bundle.js \
  -t [ babelify --presets [ latest ] ]
```
上面代码将 ES6 脚本script.js，转为bundle.js，浏览器直接加载后者就可以了。

在package.json设置下面的代码，就不用每次命令行都输入参数了。
```
{
  "browserify": {
    "transform": [["babelify", { "presets": ["latest"] }]]
  }
}
```
#### 4.8在线转换
Babel 提供一个[REPL 在线编译器](https://babeljs.io/repl/)，可以在线将 ES6 代码转为 ES5 代码。转换后的代码，可以直接作为 ES5 代码插入网页运行。

#### 4.9与其他工具的配合
许多工具需要 Babel 进行前置转码，这里举两个例子：ESLint 和 Mocha。

ESLint 用于静态检查代码的语法和风格，安装命令如下。
```
$ npm install --save-dev eslint babel-eslint
```
然后，在项目根目录下，新建一个配置文件.eslintrc，在其中加入parser字段。
```
{
  "parser": "babel-eslint",
  "rules": {
    ...
  }
}
```
再在package.json之中，加入相应的scripts脚本。
```
  {
    "name": "my-module",
    "scripts": {
      "lint": "eslint my-files.js"
    },
    "devDependencies": {
      "babel-eslint": "...",
      "eslint": "..."
    }
  }
```
Mocha 则是一个测试框架，如果需要执行使用 ES6 语法的测试脚本，可以修改package.json的scripts.test。
```
"scripts": {
  "test": "mocha --ui qunit --compilers js:babel-core/register"
}
```
上面命令中，--compilers参数指定脚本的转码器，规定后缀名为js的文件，都需要使用babel-core/register先转码。

### 5Traceur 转码器
Google 公司的[Traceur转码器](https://github.com/google/traceur-compiler)，也可以将 ES6 代码转为 ES5 代码。

#### 5.1直接插入网页
Traceur 允许将 ES6 代码直接插入网页。首先，必须在网页头部加载 Traceur 库文件。
```
<script src="https://google.github.io/traceur-compiler/bin/traceur.js"></script>
<script src="https://google.github.io/traceur-compiler/bin/BrowserSystem.js"></script>
<script src="https://google.github.io/traceur-compiler/src/bootstrap.js"></script>
<script type="module">
  import './Greeter.js';
</script>
```
上面代码中，一共有 4 个script标签。第一个是加载 Traceur 的库文件，第二个和第三个是将这个库文件用于浏览器环境，第四个则是加载用户脚本，这个脚本里面可以使用 ES6 代码。

注意，第四个script标签的type属性的值是module，而不是text/javascript。这是 Traceur 编译器识别 ES6 代码的标志，编译器会自动将所有type=module的代码编译为 ES5，然后再交给浏览器执行。

除了引用外部 ES6 脚本，也可以直接在网页中放置 ES6 代码。
```
<script type="module">
  class Calc {
    constructor() {
      console.log('Calc constructor');
    }
    add(a, b) {
      return a + b;
    }
  }

  var c = new Calc();
  console.log(c.add(4,5));
</script>
```
正常情况下，上面代码会在控制台打印出9。

如果想对 Traceur 的行为有精确控制，可以采用下面参数配置的写法。
```
<script>
  // Create the System object
  window.System = new traceur.runtime.BrowserTraceurLoader();
  // Set some experimental options
  var metadata = {
    traceurOptions: {
      experimental: true,
      properTailCalls: true,
      symbols: true,
      arrayComprehension: true,
      asyncFunctions: true,
      asyncGenerators: exponentiation,
      forOn: true,
      generatorComprehension: true
    }
  };
  // Load your module
  System.import('./myModule.js', {metadata: metadata}).catch(function(ex) {
    console.error('Import failed', ex.stack || ex);
  });
</script>
```
上面代码中，首先生成 Traceur 的全局对象window.System，然后System.import方法可以用来加载 ES6。加载的时候，需要传入一个配置对象metadata，该对象的traceurOptions属性可以配置支持 ES6 功能。如果设为experimental: true，就表示除了 ES6 以外，还支持一些实验性的新功能。

#### 5.2命令行转换
作为命令行工具使用时，Traceur 是一个 Node 的模块，首先需要用 npm 安装。
```
$ npm install -g traceur
```
安装成功后，就可以在命令行下使用 Traceur 了。

Traceur 直接运行 ES6 脚本文件，会在标准输出显示运行结果，以前面的calc.js为例。
```
$ traceur calc.js
Calc constructor
9
```
如果要将 ES6 脚本转为 ES5 保存，要采用下面的写法。
```
$ traceur --script calc.es6.js --out calc.es5.js
```
上面代码的--script选项表示指定输入文件，--out选项表示指定输出文件。

为了防止有些特性编译不成功，最好加上--experimental选项。
```
$ traceur --script calc.es6.js --out calc.es5.js --experimental
```
命令行下转换生成的文件，就可以直接放到浏览器中运行。
#### 5.3Node 环境的用法
Traceur 的 Node 用法如下（假定已安装traceur模块）。
```
var traceur = require('traceur');
var fs = require('fs');

// 将 ES6 脚本转为字符串
var contents = fs.readFileSync('es6-file.js').toString();

var result = traceur.compile(contents, {
  filename: 'es6-file.js',
  sourceMap: true,
  // 其他设置
  modules: 'commonjs'
});

if (result.error)
  throw result.error;

// result 对象的 js 属性就是转换后的 ES5 代码
fs.writeFileSync('out.js', result.js);
// sourceMap 属性对应 map 文件
fs.writeFileSync('out.js.map', result.sourceMap);
```