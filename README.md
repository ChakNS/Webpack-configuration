# webpack配置


## 注意：
webpack目前已经更新到4.x版本，配置跟之前有所不同。


## 1.安装webpack，建议局部安装，避免多个项目使用不同版本的webpack

+ 进入项目根目录，运行 npm init -y 创建package.json文件。

+ 局部安装webpack，如果是webpack 4.x版本，还需要安装webpack-cli。

```js
npm i webpack --save -dev

npm i webpack-cli --save -dev
```
+ 安装完成后在package.json中可以看到:

```
  "dependencies": {
    "webpack": "^4.31.0",
    "webpack-cli": "^3.3.2"
```

## 2.创建项目结构

+ 在根目录下创建src文件夹，用于存放HTML、JS、CSS项目文件。

+ 在src目录中创建index.html文件，用于存放HTML模板，并编写html代码。

+ 在src目录中创建main.js文件，用于存放js代码，并编写js。

+ 在根目录下创建webpack.config.js文件，用于配置webpack。

## 3.配置webpack

+ 修改webpack.config.js文件如下：

```
const path = require('path');   //引入path

module.exports = {
  mode: 'development',    //声明开发环境
  entry: './src/main.js',  //指定入口文件，为src目录下的js文件
  output: {
    filename: 'bundle.js',    //指定编译后的文件名称
    path: path.resolve(__dirname, './dist')    //指定编译后文件的保存目录
  }
};
```

+ 配置完了之后，在根目录下运行 npx webpack 即可对main.js进行编译，编译之后会在根目录下生成dist文件夹，并在dist中生成bundle.js文件，最后在HTML中通过script标签引入bundle.js文件即可。

+ 注意：运行npx webpack命令会在node_modules中找webpack并执行，也可以用绝对路径运行。

## 4.配置loader -- babel 8.x安装及配置


babel-loader用于将ES6语法转化为ES5语法，以解决浏览器兼容性问题。

+ 安装babel 8.x

请注意，babel 8.x中有一些包被废弃，而有一些新的包被要求引入进来，同时对包名进行了修改，配置的写法也相应做了改变，当时在配置的时候花了好长时间才搞明白。

```
cnpm i babel-loader '@babel/core' '@babel/preset-env' '@babel/plugin-transform-runtime' '@babel/runtime' '@babel/plugin-proposal-class-properties' -D
```
以下这些包最好都装上：
```
"@babel/core": "^7.4.4",
"@babel/plugin-proposal-class-properties": "^7.4.4",
"@babel/plugin-transform-runtime": "^7.4.4",
"@babel/preset-env": "^7.4.4",
"@babel/runtime": "^7.4.4",
"babel-loader": "^8.0.6"
```
大版本好对上就行，小版本号没什么大影响，其中babel-loader、@babel/core、@babel/preset-env、@babel/plugin-proposal-class-properties"是必须的，否则会报错。

+ 配置webpack.config.js

```
module.exports = {
  mode: 'development',    //声明开发环境
  entry: './src/main.js',  //指定入口文件，为src目录下的js文件
  output: {
    filename: 'bundle.js',    //指定编译后的文件名称
    path: path.resolve(__dirname, './dist')    //指定编译后文件的保存目录
  },
  //配置第三方模块
  module: {
  		//配置第三方模块规则
      rules : [
      	//test接收一个正则，表示以.js结尾的文件
        //use表示使用babel-loader来处理js文件
        //exclude是必须的，表示排除node_modules中的js文件，否则会耗费性能和时间，而且编译后的文件不可用
          {test: /\.js$/, use: 'babel-loader', exclude: /node_modules/}
      ]
  }
};
```

+ 配置.babelrc文件

在根目录下创建.babelrc文件，用于配置babel-loader：
```
{
    "presets": [
        "@babel/preset-env"
    ],
    "plugins": [
        "@babel/plugin-transform-runtime",
        "@babel/plugin-proposal-class-properties"
    ]
}
```
presets在这里表示语法，plugins声明上述安装的插件。
注意：@babel/plugin-proposal-class-properties 是必须的，否则在编译ES6中的Class语法时有时候会报错，这个是折腾我最长时间的一个问题。

## 5.配置loader -- css加载模块安装及配置

为了避免多次进行二次请求，一般建议将css的引入请求声明在main.js文件中，这时候需要安装相应的loader来进行解析。

+ 安装

```
cnpm i style-loader css-loader -D
```

+ 配置webpack.config.js

```
module: {
      rules : [
      		//配置js加载模块
          { test: /\.js$/, use: 'babel-loader', exclude: /node_modules/ }, 
          //配置css加载模块
          { test: /\.css$/, use: ['style-loader','css-loader'] }
      ]
  }
```
模块的处理流程是先用css-loader对css文件进行处理，再用style-loader进行二次处理并输出。因此同理可安装并配置scss、less等样式文件，分别安装sass-loader、less-loader，并在webpack.config.js中配置，添加在css-loader后面即可，webpack会从右到左调用loader模块来加载相应的样式文件。

注意：安装sass-loader、less-loader的时候，可能会要求安装node-sass、less模块，用cnpm安装即可。

+ 这时候，你可以在src目录下创建css文件，添加css样式，并在main.js中引入：

```
import './style.css';
```
这样只需要在HTML中引入script文件即可，减少浏览器的二次请求。

## 6.配置loader -- url加载模块安装及配置

当你在css文件中添加与url相关的样式时，webpack会报错，需要安装相应的loader

+ 安装url-loader

```
cnpm i url-loader file-loader -D  
```

+ 在webpack.config.js中配置

```
module: {
      rules : [
          //配置js加载模块
          { test: /\.js$/, use: 'babel-loader', exclude: /node_modules/ },
          //配置css加载模块
          { test: /\.css$/, use: ['style-loader','css-loader'] },
          //配置url加载模块
          { test: /\.(jpg|gif|png|jpeg)$/, use: 'url-loader' }
      ]
  }
```
注意：处理过后的图片名称会自动进行base64编码，防止重名，可以传入limit参数，只有小于指定字节大小的图片才会被编码：
```
{ test: /\.(png|jpg|gif)$/, use: 'url-loader?limit=43960' }
```

## 7.配置自动编译webpack-dev-server

+ 安装：

```
cnpm webpack-dev-server -D
```
+ 在package.json的scripts中配置：

```
"scripts": {
    "test": "echo \"Error: no test specified\" && exit 1",
    "dev": "webpack-dev-server" //添加dev属性
  },
```
+ 运行npm run dev

	- 此时你会发现webpack能够自动编译，但页面显示错误，而且dist中没有生成js文件，这是因为webpack-dev-server将编译生成的bundle.js放在内存中，这么做的好处是能大大加快编译速度。
	- 通过访问 http://localhost:8080/ 来进入网站，此时看到一个目录，这是我们创建的根目录，单击src即可访问到index.html。
	- 此时根目录下有一个看不到的bundle.js文件，可以通过修改html中的script标签来进行关联。

```
<script src="../bundle.js"></script>
```
+ 此时网站可以正常访问了，同时修改main.js中的代码，webpack会自动编译
+ 修改package.json来达到访问 http://localhost:8080/ 时默认进入src目录的目的：

```
"dev": "webpack-dev-server --contentBase src"
```

## 8.安装配置html-webpack-plugin插件
当你安装并配置好webpack-dev-server之后，会发现只有修改main.js时会自动编译，而修改index.html时并不会，而且浏览器不会自动刷新页面；另外，你还需修改script标签，添加 --contentBase 属性，这显然不符合我们的需求，接下来通过安装并配置html-webpack-plugin插件来解决这些问题。

+ 安装

```
cnpm i html-webpack-plugin -D 
```

+ 配置webpack.config.json,导入html-webpack-plugin插件并在module.exports中添加plugins属性

```
const path = require('path');   //导入path路径处理模块
const hwp = require('html-webpack-plugin');  //导入HTML自动处理插件

module.exports = {
  mode: 'development',    //声明开发环境
  entry: './src/main.js',  //指定入口文件，为src目录下的js文件
  output: {
    filename: 'bundle.js',    //指定编译后的文件名称
    path: path.resolve(__dirname, './dist')    //指定编译后文件的保存目录
  },
  module: {
      rules : [
          //配置js加载模块
          { test: /\.js$/, use: 'babel-loader', exclude: /node_modules/ },
          //配置css加载模块
          { test: /\.css$/, use: ['style-loader','css-loader'] },
          //配置url加载模块
          { test: /\.(jpg|gif|png|jpeg)$/, use: 'url-loader' }
      ]
  },
  plugins: [
    //添加plugins节点配置插件
    new hwp({
      template:path.resolve(__dirname,'src/index.html'), //模板文件路径
      filename:'index.html' //自动生成的HTML文件名称，该文件会自动添加script引用标签
    })
  ]
};
```
+ --contentBase src将不再需要，因为html-webpack-plugin插件会将生成的index.html文件保存在内存中，此时在根目录下跟bundle.js一样有一个看不到的index.html。
+ 此时修改index.html也会自动编译并刷新浏览器页面。
+ 另外index.html中的script标签也可以注释掉，生成的index.html文件中会自动注入bundle.js。

## 9.配置dev属性实现自动打开浏览器
+ 修改package.json：

```
"scripts": {
    "test": "echo \"Error: no test specified\" && exit 1",
    "dev": "webpack-dev-server --open --port 3000 --hot"
  },
```
+ --open 表示自动打开浏览器
+ --port 3000 表示端口为3000
+ --hot 表示启用热更新，会在下面提到

这样，当运行npm run dev时，会自动打开浏览器并访问 http://localhost:3000 

## 10. --hot热更新

+ 启动热更新的时候，当修改代码时，webpack会把修改的内容以一种类似补丁的形式加载到页面中（这是我的理解，错误的地方请大佬指正），这样不会对整个页面进行刷新重加载，加快速度。
+ 这个功能在样式改变时表现得比较明显，由于main.js中引入css，当css文件更改时，也会触发热更新，此时页面样式改变，但不刷新页面。
+ 由于webpack把main.js当作入口文件，所以只有当main.js中内容或其引入的文件内容发生改变时，才会触发热更新，因此开启热更新后，修改HTML并不会使页面更新内容，可以通过在main.js中引入index.html解决，这是一种方法。
+ 引入index.html

```
import './index.html';
```

+ 安装raw-loader加载模块

```
cnpm i raw-loader -D
```

+ 在webpack.config.js中配置

```
module: {
      rules : [
          //配置js加载模块
          { test: /\.js$/, use: 'babel-loader', exclude: /node_modules/ },
          //配置css加载模块
          { test: /\.css$/, use: ['style-loader','css-loader'] },
          //配置url加载模块
          { test: /\.(jpg|gif|png|jpeg)$/, use: 'url-loader' },
          //配置html加载模块
          { test: /\.(htm|html)$/, use: 'raw-loader' }
      ]
  },
```

+ 这样，当修改index.html的内容时，页面会自动更新




