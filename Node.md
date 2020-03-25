# Node

## npm配置

### npm安装插件

npm 是nodejs的包管理器，用于node插件的管理；

为解决npm从国外服务器下载插件过慢与异常可进行如下设置

方法一： 改变npm下载地址到taobao

- `npm install -gd express --registry =http://registry.nom.taobao.org/ `手动指定

- `npm config set registry http://registry.npm.taobao.org/ `系统设置

方法二： 使用cnpm命令，前提是否安装cnpm

相关指令：

- `npm root -g`: 查看当前插件全局安装的目录

- `npm install -g 包名 `：全局安装插件，例`npm install -g vue-cli`

- `npm uninstall -g 包名`：卸载包

  

参数说明：

-S，-D参数：

-S，--save 安装包信息将加入到dependencies（生产阶段的依赖）

-D，--save--dev 安装包信息将加入到devDependencies（开发阶段的依赖）

## VUE

### vue创建项目

- `vue init webpack projectNmae`: 创建项目，webpack为模板

![image-20200221131501150](C:\Users\liu\AppData\Roaming\Typora\typora-user-images\image-20200221131501150.png)

### 向项目中添加包

`npm install 包名 --save-dev` 在项目中添加插件，添加在dev环境。--save添加在开发环境

使用vue2启动项目  `npm run dev`

### 安装vue3

- `npm install -g @vue/lic`: 安装vuecli3
- 使用vue3启动项目`npm run server`

