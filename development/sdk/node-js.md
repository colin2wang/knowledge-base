# Node.js


node.js 的版本问题

因为 node.js V17版本中最近发布的OpenSSL3.0, 而OpenSSL3.0对允许算法和密钥大小增加了严格的限制，可能会对生态系统造成一些影响。故此以前的项目在升级 nodejs 版本后会报错

解决方法

修改package.json，在相关构建命令之前加入set NODE_OPTIONS=–openssl-legacy-provider

```shell
"scripts": {
    "serve": "SET NODE_OPTIONS=--openssl-legacy-provider && vue-cli-service serve",
    "build": "SET NODE_OPTIONS=--openssl-legacy-provider && vue-cli-service build"
},
```
