# 怎么使用 Node.js 编写 CLI 脚手架


[完整的脚手架代码][1]

### 什么是前端脚手架？
先查查百度百科里对“脚手架”的定义吧：
> 脚手架是为了保证各施工过程顺利进行而搭设的工作平台。

随着前端工程化的概念越来越深入人心，前端脚手架应运而生。

 > 简单来说，「前端脚手架」就是指通过选择几个选项快速搭建项目基础代码的工具,可以有效避免我们 ctrl + C 和 ctrl + V 相同的代码框架和基础配置。

###  背景

 当我们准备开发一个新项目的时候：
-  在较短的时间内搭建出一个完备的项目基础环境（技术栈完整，辅助功能丰富，兼顾不同环境），是比较困难的;
-  不同的项目需求和团队情况，导致我们在搭建项目基础环境的时候不能一成不变;

而前端脚手架工具：


- 可以帮助我们快速生成项目的基础代码;
- 脚手架工具的项目模板经过了开发者的提炼和检验，一定程度上代表了某类项目的最佳实践;
- 脚手架工具支持使用自定义模板，我们可以根据不同的项目进行“定制”;

 > 总结的话：脚手架是帮你减少重复性工作而做的重复性工作的工具


比较稳定且出彩的前端脚手架“脚手架”有以下几类：
-  Vue/React 脚手架

- 使用 Node、yeoman 打造自己的脚手架

- 从零搭建 webpack 脚手架

### 什么是vue脚手架？
- Vue CLI 是一个基于 Vue.js 进行快速开发的完整系统
- 作者帮我们把开发环境大部分东西都配置好了，我们把脚手架下载下来就可以直接开发了，不用再考虑搭建这些工具环境。

可以使用下列任一命令安装这个新的包：
```
npm install -g @vue/cli
# OR
yarn global add @vue/cli
```
运行以下命令来创建一个新项目：
```
vue create kk-test
```
你会被提示选取一个 preset。你可以选默认的包含了基本的 Babel + ESLint 设置的 preset，也可以选“手动选择特性”来选取需要的特性。
![默认的目录](https://cli.vuejs.org/cli-new-project.png)

生成的代码目录如下：
![<T> 语法](../images/kk-test-目录.png)

这个默认的设置非常适合快速创建一个新项目的原型，而手动设置则提供了更多的选项，它们是面向生产的项目更加需要的。
![手动设置](https://cli.vuejs.org/cli-select-features.png)



现在就让我们以any-cli为例，编写自己的脚手架工具吧。

### 核心原理

`yoeman`搭建项目需要提供`yoeman-generator`。`yoeman-generator`本质上就是一个具备完整文件结构的模板，用户需要手动地把这些模板下载到本地，然后`yoeman`就会根据这些模板自动生成各种不同的项目。

vue-cli提供了相当丰富的选项和设定功能，但是其本质也是从远程仓库把不同的模版拉取到本地，而并非是什么“本地生成”的黑科技。

**这样看来，思路也就有了——首先建立不同的模板，然后脚手架根据用户的指令引用模板生成实际项目。**

**模板既可以内置在脚手架当中，也可以部署在远程仓库。**

- 内置在脚手架中，使用`node file`操作来把模板克隆到本地。

    优点是不用新建仓库保存模板。尤其是有多项目模板时候，比如`init pc`与`init mobile`分别生成两个不同项目，我们只需要一个仓库保存脚手架即可。

    第二个优点：无论脚手架还是模板变更，只需要提交一次。

- 部署在远程仓库，使用`git clone`把项目克隆到本地。

    优点是每次模板有代码变更时，无需让用户本地升级脚手架。

    如果模板内置在脚手架里，每次模板变更，因为是在同一仓库，所以用户需要升级脚手架。而部署在远程仓库则不同，我们只需要git clone即可获取最新模板。

### 整体流程

按照标准惯例，先看下整体流程

```
添加模板->输入模板名->是否有重名模板?-添加成功:给出提示

删除模板->输入模板名->是否有模板?-删除成功:给出提示

模板列表->列出所有模板

初始化模板-输入模板名-是否有模板?-输入模块名-克隆远程仓库到本地模块-切换分支:给出提示
```

### 技术要点
#### process.cwd()与__dirname
命令都位于脚手架中，而执行命令的地方常常在模板项目中。

比如脚手架路径是`D:\Documents\Downloads\any-cli\src\command`，执行全局脚手架命令路径是` C:\Users\Administrator`。

列如：脚手架代码想读脚手架目录下的`a.js`文件 写入模板项目（执行全局脚手架命令位置）`b.js`中，

那么readFile的路径为`path.resolve(__dirname,'a.js')`,

writeFile路径为`path.resolve(process.cwd(),'b.js')`。

* process.cwd():当前Node.js进程执行时的工作目录:
* __dirname:当前模块的目录名

#### bin
许多npm模块有**可执行文件**希望被安装到全局系统路径。

需要在package.json中提供一个bin字段，它是一个命令名称到文件路径的映射。如下：
```
"bin": {
    "any": "./bin/any.js"
},
```
这样会把any命令和本地可执行文件./bin/any建立映射。也就是说，当你在命令行执行any命令时，会执行./bin/any可执行文件。

- 全局安装，npm将会使用符号链接把这些文件链接到/usr/local/bin目录下，系统的二进制命令全部在这里。

- 如果是本地安装，会链接到./node_modules/.bin目录下。只有当前目录运行any命令时，才会生效。

如果你只有一个可执行文件，并且名字和包名一样。那么你可以只用一个字符串(文件路径)，比如：
```
{
    "name": "any-cli",
    "version": "1.2.5",
    "bin": "./bin/any"
}
```
等同于：

```
{
    "name": "any-cli",
    "version": "1.2.5",
    "bin": {
        "any-cli": "./bin/any.js"
    }
}
```

#### npm link
+ 本地目录链接到全局模块

    npm link可以把本地目录链接到全局模块下。

    对于开发模块者而言，这算是最有价值的命令了。比如我们开发`any-cli`模块时，需要在命令行中使用`any`来测试我们的代码（开发中没有发布，也就无法全局安装模块）。不要担心，使用npm link一切变得非常容易。

    比如我们`any-cli`项目package.json里，有一条命令如下:
    ```
    "bin": {
        "any": "./bin/any.js"
    },
    ```
    命令行中使用npm link
    ```
    $ npm link
    ```
    得到以下结果
    ```
    /usr/local/bin/any -> /usr/local/lib/node_modules/any-cli/bin/any.js
    /usr/local/lib/node_modules/any-cli -> /Users/{Username}/work/any-cli
    ```
    window下:
    ```
    C:\Users\Administrator\AppData\Roaming\npm\any -> C:\Users\Administrator\AppData\Roaming\npm\node_modules\any-cli\bin\any.js
    C:\Users\Administrator\AppData\Roaming\npm\node_modules\any-cli -> D:\Documents\Downloads\any-cli

    ```

    **分别进入/usr/local/bin与/usr/local/lib/node_modules目录下查看。我们发现里面分别多了`any`可执行文件与`any-cli`目录。
    这样，每次本地仓库有改动时，全局命令也随之更新。我们就可以边开发边测试了**。

+ 本地目录引用全局模块

    如果你还有其他模块`temp-cli`依赖于`any-cli`模块，你可以使用如下命令把全局`any`链接到当前模块下。
    ```
    $ cd C:\temp-cli
    $ npm link any-cli # 把全局模式的模块链接到本地
    ```
    npm link any-cli 命令会去`/usr/local/lib/node_modules`目录下查找 any-cli的模块，找到这个模块后把`/usr/local/lib/node_modules/any-cli` 的目录链接到当前`temp-cli`下的`./node_modules/any-cli` 目录上。

    现在任何 any-cli 模块上的改动都会直接映射到 temp-cli 上来。

#### 其他字段：engine与engineStrict (pakeage.js里)
    node7.6.0开始支持async，如何保证用户本地安装node7.6.0以上版本呢
+ engine
    你可以在本地安装node特定版本：
    ```
    "engines": { "install-node": "7.6.0" }
    ```
    你也可以用`engines`字段来指定哪一个`npm`版本能更好地初始化你的程序，如：

    ```
    { "engines" : { "npm" : "~1.0.20" } }
    ```

+ engineStrict

如果确定模块只能能在engines 参数指定的版本正常工作，你可以在`package.json`文件中设置`engineStrict:true`，它会重写用户的`engine-strict`设置。
这个特性在npm 3.0.0中已经废弃。

#### 第三方包：pre-commit/ora/commander/chalk
- pre-commit :Git钩子脚本对于在提交代码审查之前识别简单问题很有用;
- commander:是一个轻巧的nodejs模块，提供了用户命令行输入和参数解析强大功能;
- chalk:用来在命令行输出不同颜色文字;
- ora:用于显示加载中的效果，类似于loading效果;

### 代码文件

> 创建`any-cli`项目

```
mkdir any-cli
cd any-cli
git init && npm init
```
package.json内容
```
{
  "name": "any-cli",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "pub": "npm version patch && npm publish",
    "pre-commit": "eslint src"
  },
  "author": "kk",
  "devDependencies": {
    "eslint": "^3.16.1",
    "eslint-config-airbnb": "^12.0.0",
    "eslint-plugin-babel": "^3.0.0",
    "eslint-plugin-import": "^1.6.1",
    "eslint-plugin-jsx-a11y": "^2.0.1",
    "eslint-plugin-markdown": "*",
    "eslint-plugin-react": "^6.3.0",
    "eslint-tinker": "^0.3.2",
    "pre-commit": "^1.2.2"
  },
  "dependencies": {
    "chalk": "^1.1.3",
    "child_process": "^1.0.2",
    "commander": "^2.9.0",
    "prompt": "^1.0.0"
  },
  "engines": {
    "install-node": "7.6.0"
  },
  "pre-commit": [
    "pre-commit"
  ],
  "bin": {
    "any": "./bin/any.js"
  },
  "license": "ISC"
}

```

- 字段bin下面配置被当做命令行可执行命令。指向/bin下面的any文件。
- 字段engines用来当前目录安装7.6.0版本node，可直接使用async函数而无需要再使用co模块。
- pre-commit用来做提交前代码检查，运行pre-commit脚本，也就是eslint。

> 在根目录下建立/bin 文件夹，创建any文件。

```
mkdir bin
```
这个 /bin/any是整个脚手架的入口文件，所以我们首先对它进行编写。

```
cd bin
cd.> any.js
```

```
#!/usr/bin/env node //解决不同的用户node路径不同的问题，可以让系统动态的去查找node来执行你的脚本文件
const add = require('../src/command/add')
const list = require('../src/command/list')
const init = require('../src/command/init')
const del = require('../src/command/del')
const program = require('commander')
const { version } = require('../package')

// 定义当前版本
program
.version(version)

program.parse(process.argv)
if (!program.args.length) {
  program.help()
}
```

> 我们继续在/bin/any中添加代码

```
// 定义使用方法
/*
* 添加模板
* */
program
.command('add')
.description('add template')
.alias('a')
.action(add)

/*
 * 删除模板
 * */
program
.command('del')
.description('Delete a template')
.alias('d')
.action(del)

/*
 * 模板列表
 * */
program
.command('list')
.description('List all the templates')
.alias('l')
.action(list)

/*
 * 删除初始化
 * */
program
.command('init')
.description('Generate a new project')
.alias('i')
.action(init)
```
command用来配置any命令的参数，alias配置缩写，action配置运行什么函数。

其他：

    commander 的具体使用方法在这里就不展开了，可以直接到 [官网][2] 去看详细的文档。

>  运行npm link，把当前项目链接到全局。这样就可以直接在命令行使用any命令测试/bin/any下面的代码


```
// D:\Documents\Downloads\any-cli
npm link
```

> 使用any命令，看到输出如下，证明入口文件已经编写完成了。

```
  Usage: any [options] [command]


  Commands:

    add|a    add template
    del|d    Delete a template
    list|l   List all the templates
    init|i   Generate a new project

  Options:

    -h, --help     output usage information
    -V, --version  output the version number
```

>  接着，我们创建src/command目录，

下面分别创建刚才的4个参数所对应的文件。

文件内容是具体业务代码（分别对应增删查初始化），

在此不做介绍。请参考[github链接][3]。



  [1]: http://10.10.16.8/kangxiangli/any-cli
  [2]: https://github.com/tj/commander.js/blob/HEAD/Readme_zh-CN.md
  [3]: http://10.10.16.8/kangxiangli/any-cli/tree/master/src/command
