# 首先配置 webpack
# 使用 webpack --watch 配合 nodemon 实现自动打包和node自动重启
# npm scripts 使用命名空间 npm-run-all 实现命名空间下的命令全部运行
# 同构代码，因为 renderToString 只能渲染出 html，所以让 react 代码再浏览器再运行一遍
# 使用 express.static 中间件
# client 也需要一份打包，需要配置一份 webpack.client.js 文件，打包客户端代码
# 客户端和服务端的 webpack 配置有冗余，所以将重复的部分抽离出来，并使用 webpack-merge 来进行合并

# 小结一下
1. 服务端运行 React 代码渲染出 HTML -- 所以用 webpack 打包服务端代码
2. 发送 HTML 给浏览器 -- 所以要配置 express.static 中间件
3. 浏览器接收到内容展示
4. 浏览器加载 JS 文件 -- 所以要 webpack 打包客户端代码
5. JS 中的 React 代码在浏览器端重新执行
6. JS 中的 React 代码接管页面操作

# 使用 react-router 的 staticRouter 模式，并传入 location:props
# 配置多路由，需要将 get('/') 改为 get('*')
# 为了保持 server/index.js 的单一性，将 get('*') 下的代码抽离到 utils 中
# 当一个页面返回后，后面的 Link 跳转使用的是客户端的 react 路由跳转机制，即 服务端渲染 只针对 首屏 渲染，之后的路由跳转，还是使用 客户端 机制。

- question
1. 为什么不把 renderToString 直接实现成可以运行浏览器的代码