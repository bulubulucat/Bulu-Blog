# 从零开发一个node命令行工具

##什么是命令行工具？

命令行工具（Cmmand Line Interface）简称cli，顾名思义就是在命令行终端中使用的工具。我们常用的 `git` 、`npm`、`vim` 等都是 cli 工具，比如我们可以通过 `git clone` 等命令简单把远程代码复制到本地。

##为什么要用cli工具？

和 cli 相对的是图形用户界面（gui），windows 环境中几乎都是 gui 工具，而 linux 环境中则几乎都是 cli 工具，因为两者用户不同，gui 侧重于易用，cli 则侧重于效率。对于熟悉 gui 和集成开发环境（IDE）的程序员，这似乎很难理解。毕竟用鼠标点点拽拽，不是更方便么?

很遗憾，答案是否定的。gui对于某些简单操作，可能更快、更方便。比如移动文件、阅读邮件或写word文档。但如果你依赖 gui 完成全部工作，你将会错过环境的某些能力，比如使常见任务自动化，或是利用各种工具的全部功能。并且，你也无法将工具组合，创建出定制的宏工具。gui 的好处是`所见即所得(what you see is what you get)`。缺点是`所见即全部所得(what you see is all you get)`。

作为注重实效的程序员，你不断的想要执行特别的操作（gui 可能不支持的操作）。当你想要快速地组合一些命令，以完成一次查询或某种其他的任务时，cli 要更为合适。比如：查看上周哪些js文件没有改动过：

```
# cli：
find . -name '*.js' -mtime +7 -print
# gui：
1.点击并转到"查找文件",点击"文件名"字段，敲入"*.js",选择"修改日期"选项卡；
2.然后选择"介于".点击"开始日期"，敲入项目开始的日期。
3.点击"结束日期",敲入1周以前的日期(确保手边有日历)，点击"开始查找"；
```

##如何开发一个 cli 工具？
基本上，使用任何成熟的语言都可以开发 cli 工具，作为一个前端小白，还是 JavaScript 比较顺手，因此我们选用 node 作为开发语言。

### 创建一个项目
```shell
# 1.创建一个目录：
mkdir kid-cli && cd kid-cli
# 2.因为最终我们要把cli发布到npm上，所以需要初始化一个程序包: 
npm init
# 3.创建一个index.js文件
touch index.js
# 4.打开编辑器(vscode)
code  .
```
> 安装 `code` 命令，运行 `VS code` 并打开命令面板（ `⇧⌘P `），然后输入 `shell command` 找到: `Install 'code' command in PATH` 就行了。

打开index.js文件，添加一段测试代码：
```
#!/usr/bin/env node

console.log('hello world!’)
```
终端运行 node 程序，需要先输入 node 命令，比如
```shell
node index.js
```
可以正确输出 `hello world!`，代码顶部的 `#!/usr/bin/env node`是告诉终端，这个文件要使用 node 去执行。

###创建一个命令

一般 cli都有一个特定的命令，比如 `git`，刚才使用的 `code` 等，我们也需要设置一个命令，就叫  `kid` 吧！如何让终端识别这个命令呢？很简单，打开 package.json 文件，添加一个字段 `bin`，并且声明一个命令关键字和对应执行的文件：

```
# package.json
......
  "bin": {
    "kid": "index.js"
  },
......
```
> 如果想声明多个命令，修改这个字段就好了。

然后我们测试一下,在终端中输入 `kid`，会提示:
```shell
zsh: command not found: kid
```
为什么会这样呢？回想一下，通常我们在使用一个 cli 工具时，都需要先安装它，比如 vue-cli，使用前需要全局安装:
```shell
npm i vue-cli -g
```
而我们的 kid-cli 并没有发布到 npm 上，当然也没有安装过了，所以终端现在还不认识这个命令。通常我们想本地测试一个 npm 包，可以使用：`npm link` 这个命令，本地安装这个包，我们执行一下：
```shell
npm link
```
然后再执行
```
kid
```
命令，看正确输出 `hello world!` 了。

到此，一个简单的命令行工具就完成了，但是这个工具并没有任何卵用，别着急，我们来一点一点增强它的功能。

### 查看版本信息

首先是查看 cli 的版本信息，希望通过如下命令来查看版本信息：
```
kid -v
```
这里有两个问题
1. 如何获取 `-v` 这参数？
2. 如何获取版本信息？

在 node 程序中，通过 `process.argv` 可获取到命令的参数，以数组返回，修改 index.js，输出这个数组：
```
console.log(process.argv)
```
然后输入任意命令，比如：
```shell
kid -v -h -lalala
```
控制台会输出
```shell
[ '/Users/shaolong/.nvm/versions/node/v8.9.0/bin/node',
  '/Users/shaolong/.nvm/versions/node/v8.9.0/bin/kid',
  '-v',
  '-h',
  '-lalala' ]
```
这个数组的第三个参数就是我们想要的 `-v`。

第二个问题，版本信息一般是放在package.json 文件的 version 字段中, require 进来就好了，改造后的 index.js 代码如下：
```
#!/usr/bin/env node
const pkg = require('./package.json')
const command = process.argv[2]

switch (command) {
    case '-v':
    console.log(pkg.version)
    break
    default:
    break
}
```

然后我们再执行kid -v，就可以输出版本号了。

###初始化一个项目

接下来我们来实现一个最常见的功能，利用 cli 初始化一个项目。

整个流程大概是这样的：
1. ` cd` 到一个你想新建项目的目录；
2. 执行 `kid init` 命令，根据提示输入项目名称；
3. cli 通过 git 拉取模版项目代码，并拷贝到项目名称所在目录中；

为了实现这个流程，我们需要解决下面几个问题：

#### 执行复杂的命令

上面的例子中，我们通过 process.argv 获取到了命令的参数，但是当一个命令有多个参数，或者像新建项目这种需要用户输入项目名称（我们称作“问答”）的命令时，一个简单的`swith case` 就显得捉襟见肘了。这里我们引用一个专门处理命令行交互的包：`commander`。

```shell
npm i commander --save
```
然后改造index.js
```
#!/usr/bin/env node

const program = require('commander')

program.version(require('./package.json').version)
program.parse(process.argv)
```

运行
```shall
kid -h
```
会输出
```shall
Usage: kid [options] [command]

Options:

  -V, --version  output the version number
  -h, --help     output usage information
```

commander已经为我们创建好了帮助信息，以及两个参数 `-V` 和 `-h`，上面代码中的program.version 就是返回版本号，和之前的功能一致，program.parse 是将命令参数传入commander 管道中，一般放在最后执行。

#### 添加问答操作

接下来我们添加 ` kid init` 的问答操作，这里有需要引入一个新的包：`inquirer`, 这个包可以通过简单配置让 cli 支持问答交互。
```shall
npm i inquirer --save
```

index.js：

```
#!/usr/bin/env node

const program = require('commander')
var inquirer = require('inquirer')

const initAction = () => {
    inquirer.prompt([{
        type: 'input',
        message: '请输入项目名称:',
        name: 'name'
    }]).then(answers => {
        console.log('项目名为：', answers.name)
        console.log('正在拷贝项目，请稍等')
    })
}

program.version(require('./package.json').version)

program
    .command('init')
    .description('创建项目')
    .action(initAction)

program.parse(process.argv)
```
program.command 可以定义一个命令，description 添加一个描述，在 `--help` 中展示，action 指定一个回调函数执行命令。inquirer.prompt 可以接收一组问答对象，type字段表示问答类型，name 指定答案的key，可以在 answers 里通过 name 拿到用户的输入，问答的类型有很多种，这里我们使用 input，让用户输入项目名称。

运行 kid init，然后会提示输入项目名称，输入后会打印出来。

#### 运行 shell 脚本

熟悉 git 和 linux 的同学几句话便可以初始化一个项目：

```shell
git clone xxxxx.git --depth=1
mv xxxxx my-project
rm -rf ./my-project/.git
cd my-project
npm i
```
那么如何在 node 中执行 shell 脚本呢？只需要安装 `shelljs` 这个包就可以轻松搞定。

```shell
npm i shelljs --save
```
假定我们想克隆 github 上 vue-admin-template 这个项目的代码，并自动安装依赖，改造index.js，在 initAction 函数中加上执行shell脚本的逻辑：

```
#!/usr/bin/env node

const program = require('commander')
const inquirer = require('inquirer')
const shell = require('shelljs')

const initAction = () => {
    inquirer.prompt([{
        type: 'input',
        message: '请输入项目名称:',
        name: 'name'
    }]).then(answers => {
        console.log('项目名为：', answers.name)
        console.log('正在拷贝项目，请稍等')
        
        const remote = 'https://github.com/PanJiaChen/vue-admin-template.git'
        const curName = 'vue-admin-template'
        const tarName = answers.name

        shell.exec(`
                git clone ${remote} --depth=1
                mv ${curName} ${tarName}
                rm -rf ./${tarName}/.git
                cd ${tarName}
                npm i
              `, (error, stdout, stderr) => {
            if (error) {
                console.error(`exec error: ${error}`)
                return
            }
            console.log(`${stdout}`)
            console.log(`${stderr}`)
        });
    })
}

program.version(require('./package.json').version)

program
    .command('init')
    .description('创建项目')
    .action(initAction)

program.parse(process.argv)
```
shell.exec 可以帮助我们执行一段脚本，在回调函数中可以输出脚本执行的结果。

测试一下我们初始化功能：

```shell
cd ..
kid init
# 输入一个项目名称
```

可以看到，cli已经自动从github上拉取vue-admin-template的代码，放在指定目录，并帮我们自动安装了依赖。

## 切换网络代理

因为安全策略，公司对开发网有一些网络限制，比如想要直接使用 npm 安装依赖包，通常需要配置网络代理，而使用 tnpm 时则需要切换回来。每次手动切换代理实在是麻烦。因此我们将切换网络代理的功能也添加到 cli 中。

通过之前的介绍，这个功能就很简单了，通过 shelljs 可以轻松完成，index.js:

```
......

program
  .command('proxy')
  .description('对npm和git进行代理')
  .action(() => {
    shell.exec(`
      git config --global http.proxy http://web-proxy.tencent.com:8080
	  npm config set https-proxy http://web-proxy.oa.com:8080
      npm config set registry https://registry.npm.taobao.org
    `);
    console.log('成功对npm和git进行代理');
    console.log('您可以在内网用git去拷贝外网的代码仓库，也可以用npm在内网安装依赖了。');
  });

program
  .command('tencent')
  .description('去除npm和git代理')
  .action(() => {
    shell.exec(`
      git config --global --unset http.proxy
	  npm config rm https-proxy
      npm config set registry https://registry.npm.taobao.org
    `);
    console.log('成功去除npm和git代理');
    console.log('您可以在内网用git去拷贝内网的代码仓库，但是没法再内网用npm安装依赖了。');
  });
......
```

通过 `kid proxy` 和  `kid tencent` 两个命令即可轻松切换网络代理，妈妈再也不用担心我 npm 安装不上依赖了～

## 尾声
最后别忘了将你的 cli 工具发布到 npm （tnpm）上，给更多的同学使用。

```shell
npm publish
```

怎么样，是不是感觉看似神秘的命令行开发其实也没有什么技术含量，上文列举的只是 cli 开发的冰山一角，想要开发出强大的 cli 工具，除了需要熟悉 node 和常用工具包，更重要的是了解 linux 常用命令和文件系统，希望各位同学可以受到启发，开发出属于自己的 cli 工具。