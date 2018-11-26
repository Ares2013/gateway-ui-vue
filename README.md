# Gateway UI By Element UI
> Gateway 可视化管理界面,thanks to [Element UI](http://element-cn.eleme.io/#/zh-CN)


## install

```bash
# install
npm install

# serve with hot reload at localhost:3000
npm run dev

# build for production with minification
npm run build

```
见package.json 文件的 script操作
```
{
"dev": "webpack-dev-server --inline --hot --env.dev",
"build": "rimraf dist && webpack -p --progress --hide-modules"
}
```

## 开箱即用

Gateway 后端服务 export 9093 端口之后，指定一下ui的`npm run build`后的dist目录就可以了

浏览器直接访问 `localhost:9093/ui/index.html`


## 基本使用方法

Gateway 后端核心服务请查阅 [fagongzi/gateway](https://github.com/fagongzi/gateway)


### dev 环境修改配置

见 webpack.config.js 文件的 devServer 配置
```
{
        host: 'localhost',
        port: 3000, // 启动端口
        proxy: {
            '/Web': {
                target: 'http://localhost:9093', // 转发地址
                changeOrigin: true,
                pathRewrite: {
                    '^/Web': ''
                }
            }
        },
        historyApiFallback: {
            index: url.parse(options.dev ? '/assets/' : publicPath).pathname
        }
    }
```

见config 文件下面的 dev.env.js 文件

```
{
    NODE_ENV: '"development"',
    ENV_CONFIG: '"dev"',
    BASE_URL:'""',
    BASE_API: '"/Web"'
}
```

BASE_URL 表示 访问gateway http 服务的地址。
BASE_API 表示请求url 地址的根的path，在dev 环境下面 主要是用于为了代理请求，防止浏览器访问的时候，报跨域错误。



### prod 环境配置
见config 文件下面的 prod.env.js 文件

```
{
    NODE_ENV: '"production"',
    ENV_CONFIG: '"prod"',
    BASE_URL:'"http://localhost:9093"', // 自行修改。
    BASE_API: '""'
}
```


## 部署到 nginx 静态服务器上面

```
npm run build
```

生成了 dist 文件夹，直接通过 nginx 配置一个 静态服务。

在通过浏览器访问的时候，会报跨域报错，自行百度下，如何关闭chrome 浏览器的跨域报错问题。

## nginx转发服务器，gateway 服务器，gateway-ui 服务器
当遇到 gateway 部署在单独一台服务器上面，例如1.0.0.1 ，gateway-ui 部署在另外一台服务器上面。

`ps: 这两个服务器只能通过nginx 转发服务器才能访问的到。`

### 步骤一

#### 情况一：直接部署gateway-ui的production环境

修改 config/prod.env.js 文件

```
{
    NODE_ENV: '"production"',
    ENV_CONFIG: '"prod"',
    BASE_URL:'"换成nginx转发服务器的域名"',
    BASE_API: '"换成nginx转发服务器转发到 gateway 服务器的路径，例如 /Web"'
}
```

需要在 gateway-ui 所在的服务器启动一个 http静态服务器。将 `npm run build` 生成的dist文件夹部署在上面。

### 情况二：直接部署gateway-ui的development环境
修改 config/dev.env.js

```
{
    NODE_ENV: '"development"',
    ENV_CONFIG: '"dev"',
    BASE_URL: '"换成nginx转发服务器的域名"',
    BASE_API: '"换成nginx转发服务器转发到 gateway 服务器的路径，例如 /Web"'
}
```

删除 webpack.config.js 文件里面的devServer 里面的

```
proxy: {
    '/Web': {
        target: 'http://localhost:9093',
        changeOrigin: true,
        pathRewrite: {
            '^/Web': ''
        }
    }
}
```

同时将，devServer.host 修改为 所在服务器的IP地址。

## 步骤二

配置 nginx 转发服务器

### 配置转发到 gateway-ui 服务器

```
location / {
    proxy_pass http://gateway-ui所在的服务器的ip:端口
}
```

### 配置转发到 gateway 服务器

```
location /Web {
    if($request_uri ~ /Web/(.+)){
        set $param $1;
    }

    proxy_pass http://gateway所在的服务器的ip:端口/$1
}
```

