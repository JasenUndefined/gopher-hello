# 实践 VS Code 插件开发

> 携手创作，共同成长！这是我参与「掘金日新计划 · 8 月更文挑战」的第7天，[点击查看活动详情

## 环境配置

需先安装[Node.js](https://nodejs.org/en/)和Git,接着需安装 Yeoman 和 VS Code Extension Generator

```shell
npm install -g yo generator-code
```

然后可以通过脚手架生成可开发的项目。生成过程中可根据自己的情况选择环境。执行以下命令：`yo code`。 比如可以选择语言 `typescript` ，安装过程中也可以不选择 Git 环境。

安装完成后，可以直接按 `F5`，然后会看到另一个VSCode 窗口，其中就运行着插件，然后在命令面板(`Ctrl+Shift+P`)中输入`Hello World`命令。就可以看到 `Hello World`提示弹窗。

## 源码

打开`/src/extension.ts`文件，该文件就是插件入口文件。插件的相关配置都是在 `package.json` 文件中。运行时弹出的提示就是 `extension.ts` 文件中 activate 方法

```ts
export function activate(context: vscode.ExtensionContext) {
  // 现在 package.json 文件中定义命令
  // 现在用注册表命令提供该命令的实现
  // 命令Id 必须和 package.json 文件中命令项匹配
  let disposable = vscode.commands.registerCommand('vsplus.helloWorld', () => {
   // 每次执行命令时就会执行这里的逻辑
    vscode.window.showInformationMessage('Hello from vsplus!');
  });

  context.subscriptions.push(disposable); //入栈
}
```

需要注意 vscode.commands.registerCommand 里的命令要和 package.json 文件配置的命令一致。

```json
{
  "activationEvents": [
    "onCommand:vsplus.helloWorld",
  ],
  "main": "./out/extension.js",
  "contributes": {
    "commands": [
      {
        "command": "vsplus.helloWorld",
        "title": "Hello World"
      }]
    }
}
```

- activationEvents： 插件激活数组，即配置什么情况下插件会被激活。比如onCommand： 在执行对应命令时；onView： 侧边栏中设置的 id 项目展开时

- contributes ：插件的贡献点，是整个插件的最重要配置。其中 commands 就是通过 cmd+shit+p 输入的。

这样，一个简单的 hello world  就完成了。接下来可以更深入的学习开发下去。

## 生命周期

当我们按下 `F5`时发生了什么？

按下 F5 进行调试时，自动打开一个新的 vscode 窗口，触发 package.json 中的activationEvents 配置项，extension.ts文件就开始执行，其中activate 方法就会将功能进行注册，通过使用 `vscode.commands.register...`相关的`api`将为`package.json`中的`contributes`配置项中的事件绑定方法或者监听器。

这样在命令框输入命令时，插件就会被激活，就会执行命令所绑定的事件。
