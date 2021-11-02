---
title: npm机制
date: 2021-11-02 14:37:02
author: chenhaijia
img: 
top: false
hide: false
cover: false
coverImg: 
password: 
toc: false
mathjax: false
summary: true
categories: npm
tags:
  - npm
  - node
  - dependencies
  - script
---

## 1. npm install 机制
假设项目App中有如下三个依赖：
```javascript
"dependencies": {
    A: "1.0.0",
    B: "1.0.0",
    C: "1.0.0"
}
```
A、B、C三个模块又有如下依赖：
```javascript
A@1.0.0 -> D@1.0.0
B@1.0.0 -> D@2.0.0
C@1.0.0 -> D@2.0.0
```
### npm 2.x 时代 嵌套安装
```javascript
├── node_modules
│   ├── A@1.0.0
│   │   └── node_modules
│   │   │   └── D@1.0.0
│   ├── B@1.0.0
│   │   └── node_modules
│   │   │   └── D@2.0.0
│   └── C@1.0.0
│   │   └── node_modules
│   │   │   └── D@2.0.0

```
优点：

- 层级结构明显
- 简单的实现了多版本兼容
- 保证了对依赖包无论是安装还是删除都会有统一的行为和结构

缺点：

- 可能造成相同模块大量冗余问题
- 可能造成目录结构嵌套比较深的问题
### npm 3.x开始 -扁平安装
```javascript
├── node_modules
│   ├── A@1.0.0
│   │   └── node_modules
│   │   │   └── D@1.0.0
│   ├── B@1.0.0
│   ├── C@1.0.0
│   └── D@2.0.0
```
可以看到，D@2.0.0模块被安装在一级node_modules中，而D@1.0.0仍被安装在A@1.0.0中。所以可以得出结论，**在执行npm install安装时，如果遇到相同依赖的包，会优先将高版本（大版本）的包放在一级node_modules中，低版本的包则会按照npm 2.x的方式依次挂在依赖包的node_modules中。**


再在项目中安装模块E@1.0.0（依赖于模块D@1.0.0），目录结构变为：
```javascript
├── node_modules
│   ├── A@1.0.0
│   │   └── node_modules
│   │   │   └── D@1.0.0
│   ├── B@1.0.0
│   ├── C@1.0.0
│   ├── D@2.0.0
│   ├── E@1.0.0
│   │   └── node_modules
│   │   │   └── D@1.0.0
```
再在项目中安装模块F@1.0.0（依赖于模块D@2.0.0），目录结构变为：
```javascript
├── node_modules
│   ├── A@1.0.0
│   │   └── node_modules
│   │   │   └── D@1.0.0
│   ├── B@1.0.0
│   ├── C@1.0.0
│   ├── D@2.0.0
│   ├── E@1.0.0
│   │   └── node_modules
│   │   │   └── D@1.0.0
│   └── F@1.0.0
```
可以看到，只会安装F模块。所以可以得出结论，**在一级node_moudles中已经存在依赖包的情况下，新安装的依赖包如果不存在版本冲突，则会忽略安装**。
从以上示例可以看出，npm 3.x并没有完美的解决npm 2.x中的问题，甚至还会退化到npm 2.x的行为。
### package-lock.json
从npm 5.x开始，执行npm install时会自动生成一个[package-lock.json](https://link.juejin.cn/?target=https%3A%2F%2Fdocs.npmjs.com%2Ffiles%2Fpackage-lock.json) 文件。
​

npm为了让开发者**在安全的前提下使用最新的依赖包**，在package.json中通常做了锁定大版本的操作，这样在每次npm install的时候都会拉取依赖包大版本下的最新的版本。这种机制最大的一个缺点就是当有依赖包有小版本更新时，可能会出现协同开发者的依赖包不一致的问题。


package-lock.json的详细描述主要由version、resolved、integrity、dev、requires、dependencies这几个字段构成

- version：包唯一的版本号
- resolved：安装源
- integrity：表明包完整性的hash值（验证包是否已失效）
- dev：如果为true，则此依赖关系仅是顶级模块的开发依赖关系或者是一个的传递依赖关系
- requires：依赖包所需要的所有依赖项，对应依赖包package.json里dependencies中的依赖项
- dependencies：依赖包node_modules中依赖的包，与顶层的dependencies一样的结构
```javascript
"dependencies": {
  "sass-loader": {
    "version": "7.1.0",
    "resolved": "http://registry.npm.taobao.org/sass-loader/download/sass-loader-7.1.0.tgz",
    "integrity": "sha1-Fv1ROMuLQkv4p1lSihly1yqtBp0=",
    "dev": true,
    "requires": {
      "clone-deep": "^2.0.1",
      "loader-utils": "^1.0.1",
      "lodash.tail": "^4.1.1",
      "neo-async": "^2.5.0",
      "pify": "^3.0.0",
      "semver": "^5.5.0"
    },
    "dependencies": {
      "pify": {
        "version": "3.0.0",
        "resolved": "http://registry.npm.taobao.org/pify/download/pify-3.0.0.tgz",
        "integrity": "sha1-5aSs0sEB/fPZpNB/DbxNtJ3SgXY=",
        "dev": true
      }
    }
  }
}
```
package-lock.json文件和node_modules目录结构是一一对应的，即项目目录下存在package-lock.json可以让每次安装生成的依赖目录结构保持相同。
在开发一个应用时，建议把package-lock.json文件提交到代码版本仓库，从而让你的团队成员、运维部署人员或CI系统可以在执行npm install时安装的依赖版本都是一致的。
但是在开发一个库时，则不应把package-lock.json文件发布到仓库中。实际上，npm也默认不会把package-lock.json文件发布出去。之所以这么做，是因为库项目一般是被其他项目依赖的，在不写死的情况下，就可以复用主项目已经加载过的包，而一旦库依赖的是精确的版本号那么可能会造成包的冗余。
​

## 2. npm 中的依赖包
### 依赖包分类

- **dependencies - 业务依赖**
- **devDependencies - 开发依赖**
- **peerDependencies - 同伴依赖**
- **bundledDependencies / bundleDependencies - 打包依赖**
- **optionalDependencies - 可选依赖**

作为npm的使用者，我们常用的依赖是dependencies和devDependencies，剩下三种依赖则是作为包的发布者才会使用到的字段。
#### **dependencies**
这种依赖在项目最终上线或者发布npm包时所需要，即其中的依赖项应该属于线上代码的一部分。比如框架vue，第三方的组件库element-ui等，这些依赖包都是必须装在这个选项里供生产环境使用。
通过命令npm install/i packageName -S/--save把包装在此依赖项里。如果没有指定版本，直接写一个包的名字，则安装当前npm仓库中这个包的最新版本。如果要指定版本的，可以把版本号写在包名后面，比如npm i vue@3.0.1 -S。
```git
npm install/i packageName -S/--save

// 从npm 5.x开始，可以不用手动添加-S/--save指令，直接执行npm i packageName把依赖包添加到dependencies中去。
```
​

#### **devDependencies**
这种依赖只在项目开发时所需要，即其中的依赖项不应该属于线上代码的一部分。比如构建工具webpack、gulp，预处理器babel-loader、scss-loader，测试工具e2e、chai等，这些都是辅助开发的工具包，无须在生产环境使用。
```erlang
npm install/i -D/--save-dev
npm i --production // 线上机器（或者QA环境）上使用

提示：千万别以为只有在dependencies中的模块才会被一起打包，而在devDependencies中的不会！模块能否被打包，取决于项目里是否被引入了该模块！
```
在业务项目中dependencies和devDependencies没有什么本质区别，只是单纯的一个规范作用，在执行npm i时两个依赖下的模块都会被下载；而在发布npm包的时候，包中的dependencies依赖项在安装该包的时候会被一起下载，devDependencies依赖项则不会。
#### **peerDependencies**
这种依赖的作用是提示宿主环境去安装插件在peerDependencies中所指定依赖的包，然后插件所依赖的包永远都是宿主环境统一安装的npm包，最终解决插件与所依赖包不一致的问题。
这句话听起来可能有点拗口，举个例子来给大家说明下。element-ui@2.6.3只是提供一套基于vue的ui组件库，但它要求宿主环境需要安装指定的vue版本，所以你可以看到element项目中的package.json中具有一项配置：
```javascript
"peerDependencies": {
    "vue": "^2.5.16"
}
```
它要求宿主环境安装3.0.0 > vue@ >= 2.5.16的版本，也就是element-ui的运行依赖宿主环境提供的该版本范围的vue依赖包
​

总结：大白话：如果你安装我，那么你最好也要按照我的要求安装A、B和C。
#### **bundledDependencies **
这种依赖跟npm pack打包命令有关。假设package.json中有如下配置：
```javascript
{
  "name": "font-end",
  "version": "1.0.0",
  "dependencies": {
    "fe1": "^0.3.2",
    ...
  },
  "devDependencies": {
    ...
    "fe2": "^1.0.0"
  },
  "bundledDependencies": [
    "fe1",
    "fe2"
  ]
}
```
执行打包命令npm pack，会生成front-end-1.0.0.tgz压缩包，并且该压缩包中包含fe1和fe2两个安装包，这样使用者执行npm install front-end-1.0.0.tgz也会安装这两个依赖。
tips: 在bundledDependencies中指定的依赖包，必须先在dependencies和devDependencies声明过，否则打包会报错。
#### **optionalDependencies**
这种依赖中的依赖项即使安装失败了，也不影响整个安装的过程。需要注意的是，如果一个依赖同时出现在dependencies和optionalDependencies中，那么optionalDependencies会获得更高的优先级，可能造成一些预期之外的效果，所以尽量要避免这种情况发生。
​

tips:  在实际项目中，如果某个包已经失效，我们通常会寻找它的替代者，或者换一个实现方案。不确定的依赖会增加代码判断和测试难度，所以这个依赖项还是尽量不要使用。


### 依赖包版本号
npm采用了semver规范作为依赖版本管理方案。
​

一个npm依赖包的版本格式一般为：**主版本号.次版本号.修订号**（x.y.z），每个号的含义是：

- **主版本号**（也叫大版本，major version）

大版本的改动很可能是一次颠覆性的改动，也就意味着可能存在与低版本不兼容的API或者用法，（比如 vue 2 -> 3)。

- **次版本号**（也叫小版本，minor version）

小版本的改动应当兼容同一个大版本内的API和用法，因此应该让开发者无感。所以我们通常只说大版本号，很少会精确到小版本号。
tips: 如果大版本号是 0 的话，表示软件处于开发初始阶段，一切都可能随时被改变，可能每个小版本之间也会存在不兼容性。所以在选择依赖时，尽量避开大版本号是 0 的包。

- **修订号**（也叫补丁，patch）一般用于修复bug或者很细微的变更，也需要保持向前兼容。

常见的几个版本格式如下：

- **"1.2.3"**

表示精确版本号。任何其他版本号都不匹配。在一些比较重要的线上项目中，建议使用这种方式锁定版本。

- **"^1.2.3"**

表示兼容补丁和小版本更新的版本号。官方的定义是“能够兼容除了最左侧的非 **0** 版本号之外的其他变化。
```javascript
"^1.2.3" 等价于 ">= 1.2.3 < 2.0.0"。即只要最左侧的 "1" 不变，其他都可以改变。所以 "1.2.4", "1.3.0" 都可以兼容。

"^0.2.3" 等价于 ">= 0.2.3 < 0.3.0"。因为最左侧的是 "0"，那么只要第二位 "2" 不变，其他的都兼容，比如 "0.2.4" 和 "0.2.99"。

"^0.0.3" 等价于 ">= 0.0.3 < 0.0.4"。大版本号和小版本号都为 "0" ，所以也就等价于精确的 "0.0.3"。
```

- **"~1.2.3"**

表示只兼容补丁更新的版本号。关于 ~ 的定义分为两部分：如果列出了小版本号（第二位），则只兼容补丁（第三位）的修改；如果没有列出小版本号，则兼容第二和第三位的修改。我们分两种情况理解一下这个定义：
```javascript
"~1.2.3" 列出了小版本号 "2"，因此只兼容第三位的修改，等价于 ">= 1.2.3 < 1.3.0"。

"~1.2" 也列出了小版本号 "2"，因此和上面一样兼容第三位的修改，等价于 ">= 1.2.0 < 1.3.0"。

"~1" 没有列出小版本号，可以兼容第二第三位的修改，因此等价于 ">= 1.0.0 < 2.0.0"
```

- **"1.x" 、"1.X"、1.*"、"1"、"*"**
```javascript
"*" 、"x" 或者 （空） 表示可以匹配任何版本。

"1.x", "1.*" 和 "1" 表示匹配主版本号为 "1" 的所有版本，因此等价于 ">= 1.0.0 < 2.0.0"。

"1.2.x", "1.2.*" 和 "1.2" 表示匹配版本号以 "1.2" 开头的所有版本，因此等价于 ">= 1.2.0 < 1.3.0"。
```

- **"1.2.3-alpha.1"、"1.2.3-beta.1"、"1.2.3-rc.1"**
```javascript
alpha(α)：预览版，或者叫内部测试版；一般不向外部发布，会有很多bug；一般只有测试人员使用。

beta(β)：测试版，或者叫公开测试版；这个阶段的版本会一直加入新的功能；在alpha版之后推出。

rc(release candidate)：最终测试版本；可能成为最终产品的候选版本，如果未出现问题则可发布成为正式版本。
```
```javascript
"~1.2.4-alpha.1" 表示 ">=1.2.4-alpha.1 < 1.3.0"。这样 "1.2.5", "1.2.4-alpha.2" 都符合条件，而 "1.2.5-alpha.1", "1.3.0" 不符合。

"^1.2.4-alpha.1" 表示 ">=1.2.4-alpha.1 < 2.0.0"。这样 "1.2.5", "1.2.4-alpha.2", "1.3.0" 都符合条件，而 "1.2.5-alpha.1", "2.0.0" 不符合。

// >1.2.4-alpha.1"表示接受 "1.2.4-alpha" 版本下所有大于 1 的预发布版本。因此 "1.2.4-alpha.7" 是符合要求的，但 "1.2.4-beta.1" 和 "1.2.5-alpha.2" 都不符合。此外如果是正式版本（不带预发布关键词），只要版本号符合要求即可，不检查预发布版本号，例如 "1.2.5", "1.3.0" 都是认可的。
```
以包开发者的角度来考虑这个问题：假设当前线上版本是 "1.2.3"，如果我作了一些改动需要发布版本 "1.2.4"，但我不想直接上线（因为使用 "~1.2.3" 或者 "^1.2.3" 的用户都会直接静默更新），这就需要使用预发布功能。因此我可能会发布 "1.2.4-alpha.1" 或者 "1.2.4-beta.1" 等等。
### 依赖包版本管理

1. 在大版本相同的前提下，如果一个模块在package.json中的小版本要**大于**package-lock.json中的小版本，则在执行npm install时，会将该模块更新到大版本下的最新的版本，并将版本号更新至package-lock.json。如果**小于**，则被package-lock.json中的版本锁定。
```javascript
// package-lock.json 中原版本
"clipboard": {
  "version": "1.5.10", 
},
"vue": {
  "version": "2.6.10",
}
// package.json 中修改版本
"dependencies": {
  "clipboard": "^1.5.12",
  "vue": "^2.5.6"
  ...
}

// 执行完 npm install 后，package-lock.json 中
"clipboard": {
  "version": "1.7.1", // 更新到大版本下的最新版本
},
"vue": {
  "version": "2.6.10", // 版本没发生改变
}
```

2. 如果一个模块在package.json和package-lock.json中的大版本不相同，则在执行npm install时，都将根据package.json中大版本下的最新版本进行更新，并将版本号更新至package-lock.json。
```javascript
// package-lock.json 中原版本
"clipboard": {
  "version": "2.0.4",
}
// package.json 中修改版本
"dependencies": {
  "clipboard": "^1.6.1",
}

// 执行完npm install后，package-lock.json 中
"clipboard": {
  "version": "1.7.1", // 更新到大版本下的最新版本
}
```

3. 如果一个模块在package.json中有记录，而在package-lock.json中无记录，执行npm install后，则会在package-lock.json生成该模块的详细记录。同理，一个模块在package.json中无记录，而在package-lock.json中有记录，执行npm install后，则会在package-lock.json删除该模块的详细记录。



## 3. npm scripts 脚本
package.json中的 [scripts](https://link.juejin.cn/?target=https%3A%2F%2Fdocs.npmjs.com%2Fmisc%2Fscripts) 字段可以用来自定义脚本命令，它的每一个属性，对应一段脚本。以vue-cli3为例：
```javascript
"scripts": {
  "serve": "vue-cli-service serve",
  ...
}
```
这样就可以通过npm run serve脚本代替vue-cli-service serve脚本来启动项目，而无需每次敲一遍这么冗长的脚本。
### 工作原理
#### package.json 中的 bin 字段
package.json中的字段 [bin](https://link.juejin.cn/?target=https%3A%2F%2Fdocs.npmjs.com%2Ffiles%2Fpackage.json.html%23bin) 表示的是一个**可执行文件到指定文件源的映射**。通过npm bin指令显示当前项目的bin目录的路径。例如在@vue/cli的package.json中：
例如:在chj-ip-tool的package.json中：
![image.png](https://cdn.nlark.com/yuque/0/2021/png/12582850/1634778717199-d5160d94-06bf-4a78-bc8b-cee18b0967e8.png#clientId=ubcbcb43a-f3df-4&from=paste&height=75&id=u78df9db5&margin=%5Bobject%20Object%5D&name=image.png&originHeight=75&originWidth=259&originalType=binary&ratio=1&size=2649&status=done&style=none&taskId=u3ca378d9-3c35-4301-a6b8-c398725a4a1&width=259)
全局安装: 
如果全局安装chj-ip-tool的话， npm会在全局可执行bin文件安装目录			C:\Users\Administrator\AppData\Roaming\npm下创建一个指向C:\Users\Administrator\AppData\Roaming\npm\node_modules\chj-ip-tool\bin\cli.js文件的名为my-ip的软链接，这样就可以直接在终端输入my-ip来执行相关命令。
![image.png](https://cdn.nlark.com/yuque/0/2021/png/12582850/1629032050100-a436e440-cf65-4a1b-a1e8-88eae7c7d5d8.png#clientId=ue2a260f2-efff-4&from=paste&height=40&id=u73dbfd80&margin=%5Bobject%20Object%5D&name=image.png&originHeight=62&originWidth=907&originalType=binary&ratio=1&size=29245&status=done&style=none&taskId=u7f4aa3cd-80fb-439e-80b7-18c11e136b2&width=588.4977722167969)
局部安装:
如果局部安装chj-ip-tool的话，npm则会在本地项目./node_modules/.bin目录下创建一个指向./node_moudles/haijia-tool/bin/cli.js名为my-ip的软链接，这个时候如果想要执行my-ip命令有三种方式：
```javascript
1. 直接输入.\node_modules\.bin\my-ip来执行。 
2. 使用npx vue命令来执行（npx 的作用就是为了方便调用项目内部安装的模块） 	
3. 使用npm run命令来执行（npm run会将当前项目的./node_modules/.bin的绝对路径加入全局环境变量中）
```
#### PATH 环境变量
在terminal中执行命令时，**命令会在PATH环境变量里包含的路径中去寻找相同名字的可执行文件**。局部安装的包只在./node_modules/.bin中注册了它们的可执行文件，不会被包含在PATH环境变量中，这个时候在terminal中输入命令将会报无法找到的错误。
**那为什么通过npm run可以执行局部安装的命令行包呢？**
​

npm 脚本的原理非常简单。每当执行npm run，就会自动新建一个 Shell，在这个 Shell 里面执行指定的脚本命令。因此，只要是 Shell（一般是 Bash）可以运行的命令，就可以写在 npm 脚本里面。
比较特别的是，npm run新建的这个 Shell，会将当前目录的node_modules/.bin子目录加入PATH变量，执行结束后，再将PATH变量恢复原样。
我们来验证下这个说法。首先执行 cmd =>path 查看当前所有的环境变量，可以看到PATH环境变量为：
```javascript
PATH=C:\Windows\system32;C:\Windows;C:\Windows\System32\Wbem;C:\Windows\System32\WindowsPowerShell\v1.0\;C:\Windows\System32\OpenSSH\;D:\前端软件\Microsoft VS Code\bin;;D:\前端软件\Git\cmd;D:\前端软件\node\;C:\Users\ybchjia\AppData\Local\Microsoft\WindowsApps;C:\Users\ybchjia\AppData\Roaming\npm
```
	再在当前项目下执行npm run env查看脚本运行时的环境变量，可以看到PATH环境变量为：
```javascript
Path=D:\前端软件\node\node_modules\npm\node_modules\npm-lifecycle\node-gyp-bin;D:\project-demo\ip-tool\node_modules\.bin;C:\Windows\system32;C:\Windows;C:\Windows\System32\Wbem;C:\Windows\System32\WindowsPowerShell\v1.0\;C:\Windows\System32\OpenSSH\;D:\前端软 
件\Microsoft VS Code\bin;;D:\前端软件\Git\cmd;D:\前端软件\node\;C:\Users\ybchjia\AppData\Local\Microsoft\WindowsApps;C:\Users\ybchjia\AppData\Roaming\npm
```
这意味着，当前目录的node_modules/.bin子目录里面的所有脚本，都可以直接用脚本名调用，而不必加上路径。比如，当前项目的依赖里面有 Mocha，只要直接写mocha test就可以了。
```javascript
"test": "mocha test"
"test": "./node_modules/.bin/mocha test" // 不必带上路径
```
由于 npm 脚本的唯一要求就是可以在 Shell 执行，因此它不一定是 Node 脚本，任何可执行文件都可以写在里面。
npm 脚本的退出码，也遵守 Shell 脚本规则。如果退出码不是0，npm 就认为这个脚本执行失败。
​

PATH环境变量，是告诉系统，当要求系统运行一个程序而没有告诉它程序所在的完整路径时，系统除了在当前目录下面寻找此程序外，还应到哪些目录下去寻找。
​

### 用法指南
#### 传入参数
node处理scripts参数其实很简单，比如：
![image.png](https://cdn.nlark.com/yuque/0/2021/png/12582850/1629032201141-537a223f-f405-4a01-989f-fd10365bc906.png#clientId=ue2a260f2-efff-4&from=paste&height=132&id=u6bc9ef8c&margin=%5Bobject%20Object%5D&name=image.png&originHeight=150&originWidth=631&originalType=binary&ratio=1&size=24772&status=done&style=none&taskId=u9e4d9430-50ea-428a-a084-4784028eb23&width=553.4920349121094)
除了第一个可执行的命令，以空格分割的任何字符串都是参数，并且都能通过process.argv属性访问。执行npm run serve3命令，process.argv的具体内容为：
![image.png](https://cdn.nlark.com/yuque/0/2021/png/12582850/1629032241556-ac230118-702b-4ca3-a0ea-f2e66fbaae7f.png#clientId=ue2a260f2-efff-4&from=paste&height=176&id=u21b3f8d9&margin=%5Bobject%20Object%5D&name=image.png&originHeight=200&originWidth=631&originalType=binary&ratio=1&size=29444&status=done&style=none&taskId=u1dbb903d-9093-4da8-aa23-7cfc9594ad1&width=554.4859008789062)
很多命令行包之所以这么写，都是依赖了 minimist 或者  yargs ，coa等参数解析工具来对命令行参数进行解析。以minimist对vue-cli-service serve --mode=dev --mobile -config build/example.js解析为例，解析后的结果为：
![image.png](https://cdn.nlark.com/yuque/0/2021/png/12582850/1629032274531-2b333d35-2c45-47c5-aa23-9589eff91557.png#clientId=ue2a260f2-efff-4&from=paste&height=84&id=u5742ba09&margin=%5Bobject%20Object%5D&name=image.png&originHeight=167&originWidth=777&originalType=binary&ratio=1&size=11794&status=done&style=none&taskId=ub57736d4-bd43-4121-8ea8-3599438e970&width=388.5)
我们还可以通过命令行传参的形式来进行参数传递：
```javascript
npm run serve --params  // 参数params将转化成process.env.npm_config_params = true
npm run serve --params=123 // 参数params将转化成process.env.npm_config_params = 123
npm run serve -params  // 等同于--params参数

npm run serve -- --params  // 将--params参数添加到process.argv数组中
npm run serve params  // 将params参数添加到process.argv数组中
npm run serve -- params  // 将params参数添加到process.argv数组中
```
#### 多命令运行
##### 串行执行
串行执行，要求前一个任务执行成功以后才能执行下一个任务，使用&&符号来连接。
```javascript
npm run script1 && npm run script2
串行命令执行过程中，只要一个命令执行失败，则整个脚本终止。
```
##### 并行执行
并行执行，就是多个命令可以同时的平行执行，使用&符号来连接。
```javascript
npm run script1 & npm run script2
// 这两个符号是Bash的内置功能。此外，还可以使用第三方的任务管理器模块：script-runner、npm-run-all、redrun。
```
## 4.实用技巧
### 模块管理
```javascript
npm list/ls <packageName>
// 检查当前项目依赖的所有模块，包括子模块以及子模块的子模块;
  
npm view/info <packageName> version
// 模块已经发布的最新的版本信息（不包括预发布版本）

npm view/info <packageName> versions 
// 模块所有的历史版本信息（包括预发布版本）

npm view/info <packageName> <package.json中的key值> 
 // 还能查看package.json中字段对应的值
  
npm view/info <packageName>
// 查看某个模块的所有信息，包括它的依赖、关键字、更新日期、贡献者、仓库地址和许可证等：
  
npm outdated
// 查看当前项目中可升级的模块：
```
### 查看模块文档
```javascript
打开模块的主页：npm home <packageName> 
```
![image.png](https://cdn.nlark.com/yuque/0/2021/png/12582850/1629031781587-128c4c69-8fd0-46eb-aa47-9ac493bba51e.png#clientId=ue2a260f2-efff-4&from=paste&height=40&id=ua9be24f4&margin=%5Bobject%20Object%5D&name=image.png&originHeight=42&originWidth=419&originalType=binary&ratio=1&size=2988&status=done&style=none&taskId=u1ba823a5-c6a0-4b2b-8e05-7d71ffa0f4b&width=396.4918670654297)
```javascript
打开模块的代码仓库：npm repo <packageName> 
```
![image.png](https://cdn.nlark.com/yuque/0/2021/png/12582850/1629031812795-184e7b19-ebcd-419d-922d-eff34ed70de5.png#clientId=ue2a260f2-efff-4&from=paste&height=70&id=ua720c18e&margin=%5Bobject%20Object%5D&name=image.png&originHeight=98&originWidth=569&originalType=binary&ratio=1&size=6405&status=done&style=none&taskId=ucfef75a1-157a-4f70-b349-a10fa3ffc20&width=407.493896484375)
```javascript
打开模块的 issues 地址：npm bugs <packageName>
```
![image.png](https://cdn.nlark.com/yuque/0/2021/png/12582850/1629031842936-4c11d87c-7132-4d06-b237-be77ee85ac9d.png#clientId=ue2a260f2-efff-4&from=paste&height=61&id=ua2b9f783&margin=%5Bobject%20Object%5D&name=image.png&originHeight=77&originWidth=543&originalType=binary&ratio=1&size=4683&status=done&style=none&taskId=u21710b30-ef8a-4bba-b6f3-50cacfb4dcf&width=428.49822998046875)


### 本地开发模块调试
方案一：本地打包引用     
```javascript
本地包：npm pack
项目：npm i < package >（本地路径）
```
方案二：npm发布引用
```javascript
npm i  < packageName >
```
方案三：npm link（重点）    本地包：npm link这会创建一个软连接，并保存到目录
C:\Users\Administrator\AppData\Roaming\npm\node_modules 下面
```javascript
npm link < packageName >
```
这就将这个公共的项目通过软连接的方式引入到项目里面来了;
这时修改common项目下面的任意代码都会实时生效，不用打包，不用更新引入包，也不用重启。需要注意的是，当项目包依赖更新后，也就是执行了 npm install xxx 之后，需要重新link项目。
