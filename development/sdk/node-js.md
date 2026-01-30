# Node.js

## Node.js Version Issues

In Node.js V17, the recently released OpenSSL 3.0 has introduced strict restrictions on allowed algorithms and key sizes, which may cause some impact on the ecosystem. Therefore, previous projects may encounter errors after upgrading the Node.js version.

## Solution

Modify package.json by adding `set NODE_OPTIONS=--openssl-legacy-provider` before the relevant build commands:

```shell
"scripts": {
    "serve": "SET NODE_OPTIONS=--openssl-legacy-provider && vue-cli-service serve",
    "build": "SET NODE_OPTIONS=--openssl-legacy-provider && vue-cli-service build"
},
```
