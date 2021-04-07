## 安装遇到2502 2503

管理员运行命令行
msiexec /package node-v8.7.0-64.msi

##新建cache文件夹

node_cache

##配置npm的全局模块的存放路径、cache的路径

npm config set prefix "D:\nodejs\node_global"

npm config set cache"D:\nodejs\node_cache"

##添加NODE_PATH

D:\nodejs\node_global\node_modules

Path:D:\nodejs\node_global

##安装cnpm

npm install -g cnpm --registry=https://registry.npm.taobao.org

##安装webpack

cnpm install -g webpack

##安装webpack-dev-server

cnpm install -g --save-dev webpack-dev-server


##安装项目

cnpm install

##dev运行

cnpm run dev

111