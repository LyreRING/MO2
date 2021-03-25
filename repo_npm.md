## 学习时间：
* 2021/3/25
## 学习内容
* 编译前端
## 学习计划：
* 了解基本工具，动手实践
## 学习总结：

#### Node.js：
* Node.js是运行在服务端的JavaScript，是一个基于Chrome JavaScript运行时建立的一个平台。
* node -v查看当前Node版本，path可检测环境变量是否配置，node --version查看版本
* node xx.js可执行Node.js程序，若直接输入node，可进入命令交互模式输入代码语句。
#### npm：
* NPM是随同NodeJS一起安装的包管理工具，能解决NodeJS代码部署上很多问题。
* 允许用户从NPM服务器下载别人编写的第三方包/编写的命令行程序到本地使用或自己编写的上传供别人使用。
* npm -v可测试是否安装成功，npm install npm -g可升级
* npm install <Module Name> （如express）安装Node.js模块 （直接跟网站仓库名应该也能安装下载文件）
#### 编译前端：
* fork项目仓库（mo2-1)
* git clone远程克隆一个本地库
* cd切换到目录（mo2front）
* npm install安装
* npm run serve运行前端，根据命令行提示打开对应网页可以看到前端界面
#### 问题处理：
* Q：npm install中出现了大量warning与error？
* A：Typescript为JS超集，ESLint可通过配置解析器，规范TS代码。ESlint很严苛，各种细节都报错，一般不影响情况下可忽略。
* Q：Localhost可以访问，但DevServer无法正常加载？
* A：网站设计重视安全，DevServer（开发服务器）必须对cookie进行重写才能正常访问网页。
* Q：为什么再localhost上更改会同步至原网站？
* A: 通过反向代理对localhost的请求被DevServer转发给了原网站（motwo.cn）。
#### 代理：
* 正向代理：隐藏了真实的请求客户端，客户端请求的服务都被代理服务器代替来请求。
* 反向代理：隐藏了真实的服务端，反向代理服务器会将客户端请求转发到真实的服务器。