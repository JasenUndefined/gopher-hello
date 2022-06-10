## 初学者指南之GitHub CI / CD和自动化

CI/CD 和工作流自动化是 GitHub 平台上的原生功能。 以下是如何开始使用它们并加快您的工作流程。

## Github Actions

Github Actions 是一个本地 CI/CD 工具，可与你在 GitHub 中的代码一起运行。进入你的 Github 中的任意一个 repository，在选项卡上有一个  “Actions” 菜单。如果你第一次打开 “Actions” 这个菜单，你会看到 Actions 是什么的快速说明，和一些 repository 工作流程的建议。

Actions 带有 13000 多个预先编写和测试的 CI/CD 工作流和 Github 市场中的预构建自动化，以及使用 yaml 文件来编写自己的工作流或自定义现有的工作流。

Github Actions 工作流也支持响应 GitHub 上的任何 webhook 事件。那么意味着你可以将 GitHub 上的任何 webhook 转换为 CI/CD 管道中的自动化触发器，当然也支持第三方的 webhook 事件。

## Github Actions Workflow

```yaml
name: CI

on:
	push:
		branches: [master]
	pull_request:
  	branches: [master]
  workflow_dispatch:

jobs:
	build:
		runs-on: ubuntu-latest
		
		steps:
		
			- name: 
			  uses:
			  
			- name: 
			  uses:
      - name: 
			  uses:
```

以上的工作流程命令组成各种不同的事情：

- Runners 一个Runner 是一个 Github Action server。每个 runner 可以托管或自托管在一个本地服务上。GitHub-hosted runners 是基于 linux，window ，macOS系统上运行，你可以在你的工作流中指定。
- Events  是触发启动一个工作流的事件
- Jobs 是一系列执行步骤
- Steps 在作业中运行命令的单个任务。 这些可以是一个动作或一个 shell 命令。
- Actions 在运行器上执行的命令，也是以它命名的 GitHub Actions 的核心元素。



参考资料：

- https://github.blog/2022-06-03-a-beginners-guide-to-ci-cd-and-automation-on-github/?continueFlag=b4b2acb83bdbc1b6c5ab4d0c11c5ed56