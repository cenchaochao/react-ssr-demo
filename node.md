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

# 引入 redux，需要在 client 和 server 的 index 中都引入 redux
# 如果直接导出一个 store/index.js 直接导出一个 store，会生成一个单例 store，每个用户都是使用一个 store，所以使用导出一个函数，生成一个 store
# 大型项目的 redux 组织
# 在 constants 里写 type，引入到 actions 和 reducer
# actions 里写，生成 action 的函数，和异步 action
# reducer case type 
# 最后将 reducer 导入到 index 中，做导出
# 服务器端不会执行 componentDidMount
# 调整路由结构，成为数组结构，然后在 client 和 server 原先的 Routes 组件改为遍历数组 传入 {...route}
# 当接收到 http 请求时，遍历 routes 找到和当前路径匹配的 route 调用其中的 loadData 方法
# 服务器会自动发送一个 favicon 的请求，所以要设置 favicon.ico，放在 public 目录下即可
# react-router-dom 的 matchPath 只能匹配单层路由，要匹配嵌套路由使用 react-router-config 中的 matchRoutes
# 遍历 matchRoutes 数组，将带有 loadData 的路由调用 loadData，这时需要将 loadData 的 return 结果变为 promise(其实只要在 action 中把 axios return 出来即可)，然后将 loadData 的 return 结果 push 到 promises 数组中
# 调用 Promise.all(promises)，最后将结果返回
# 数据的注水与脱水，将服务器生成的 store，直接挂在到 script 标签下，客服端生成 store 时，取出 store.state 做默认值，这样就不用 componentDidMount 里写初始加载代码

# 用 express-http-proxy 做代理，如果使用了服务端渲染，最好所有请求都通过 node 中间层再请求后端数据，有利于排查错误
# axios 发起请求时，服务端发起请求需要使用绝对路径，仔细思考一下这边就能理解
# 为了让代码更加简洁，使用 axios 的 instance，axios.create 方法改写 baseURL
# 以上的做法，每次调用 dispatch 时都需要传入 server:true | false，可以使用 redux-thunk 的 withExtraArgument 属性，传入服务端和客服端各自的 axiosInstance，这样就可以在 dispatch 中接收到这个传入的 instance，直接调用发起请求即可，原因是，dispatch 是 store 的方法，在生成 store 时可以根据不同的环境传入不同的 instance
# 小结一下，上面主要处理的路径的问题
# 现在客户端的请求从中间层代理，中间层做转发
# 中间层的请求直接转向服务端，然后注入到 store 中

# 多级路由使用 react-router-config 中的 renderRoutes 方法，这个方法可以循环遍历数组项，并将 子路由 传递到 props 中，就可以实现嵌套路由，这里有点类似 vue-router 的概念，使用配置来完成路由，但其实 react-router 使用的还是组件式，不过是被包装在了 react-router-config 中。
# 处理 cookie，因为 express-http-proxy 不处理 cookie 转发，所以导致用户登录后并携带 cookie 访问接口时，经过 中间层 转发后，cookie 就没有携带了，导致携带 cookie 的情况下 自动登录 失败，所以在创建 axiosInstance 时，处理 header

# 如果单纯的做一个 NotFound 的组件，不会发送 404 code，所以在 renderHTML 时 传入一个 context，在 staticRouter 的 context:props 中传入，这个参数，会让每个组件都收到一个 staticContext 的 props，可以在 NotFound 组件中，给这个 staticContext 属性添加一个 NotFound = true 的键值，然后就可以在 render 后，判断并发送 404
# 301重定向，如果在服务端渲染页面时发现有 Redirect 组件调用时，会在 staticContext 中自动注入一些键值，可以利用这些属性完成 301 重定向
# 在 promises 中将 loadData:promise 再次包装 promise，这样不论 loadData 是否成功还是失败，最后都会调用 resolve，这样就可以在 Promise.all 中可以永远调用成功，保证不论 loadData 数据是否成功还是，都会渲染，即使一个请求失败，还是会等待其他请求的完成；如果不这么包装，一旦一个请求失败，就走到了 catch 流程。

# 引入 css-loader，client 配置 style-loader，server 端配置 iosmorphic-style-loader
# 使用 import styles from './index.css' 导入 css 样式时，iosmorphic-style-loader 提供了 _getCss() 方法，可以获得 css 字符串，可以将这个字符串注入到 staticContext 服务端独有的全局对象中，在 renderHTML 中注入到 HTML 文件中的 style 标签中
# 为了让多个组件的 css 样式都能启用，把 context 下的 css 改为数组，然后拼接起来
# 组件中的 loadData 是挂载在最原始的组件上的，之后可能要用 connect 注入 props，没有报错是因为 connect 组件把原始组件中的静态方法复制到了自己身上，这不过样看起来很不直观，应该把 loadData 挂载在导出的组件上
# 每个组件都会引入 css 文件，所以把 componentWillMount 抽离出来，做成 HOC，这里就一定要完成 loadData 的挂载，自己写的 withStyle 组件没有把 静态方法 复制到返回出来的组件上

- question
1. 为什么不把 renderToString 直接实现成可以运行浏览器的代码