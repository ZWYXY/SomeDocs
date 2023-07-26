# Vue 笔记
## 环境准备
1. nodejs
2. 安装完成后换源（淘宝源）

## IDE
1. vscode 
2. Plugins
   1. VUE Language Features(Volar)
   2. TypeScript Vue Plugin(Volar)


## 常用包管理指令
1. npm i        安装依赖

2. npm run dev  程序运行

3. `npm list -g` 查看全局安装了哪些包

4. npm换源

   ```bash
   # 查看源, registry就是源地址，默认是 https://registry.npmmirror.com/
   npm config get registry 
   npm config list
   # 临时换源
   npm --registry=https://registry.npm.taobao.org
   # 永久换源
   npm config set registry https://registry.npm.taobao.org
   # 使用第三方工具，管理多种源
   npm i mxzs -g
   # 上述工具下载完成之后，查看其支持的源，和当前使用的源
   mmp ls 
   # 切换源 
   mmp use 
   # mmp 用例
   mmp -h
   ```

   

   

## 一些帮助开发的东西
1. 删除node_modules缓慢
    1. 全局安装 npm install rimraf -g
    2. 在项目中使用 rimraf node_modules
    3. [使用方式详解](https://blog.csdn.net/qq443068902/article/details/124666551)


### 开源的starter
1.  https://github.com/element-plus/element-plus-vite-starter 集成了vue +  element-plus + vite

### 涉及到的UI和打包工具
1. Element-plus + Vite



## 项目初始化的一些方式（构建VUE脚手架）

### vite

```bash	
npm init vite@latest
```



### vue

```bash	
npm init vue@latest
```



