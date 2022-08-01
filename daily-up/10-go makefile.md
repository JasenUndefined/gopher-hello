# 使用 Makefile 来构建 Golang 项目

构建和测试大型项目时都会很耗时，且容易出错。开发者在开发过程中需要不断执行go build、go run 、go test等相关命令。还可能需要多个命令来构建不同平台的二进制文件。在正式部署时候，我们可能还需要安装一些依赖项，或者在发布之前进行代码覆盖率测试等相关前置工作。

整个过程需要很多步骤，但我们有一种简单的方法可以解决这些复杂琐碎的步骤。使用 Make 进行自动执行任务。它通过单个命令简化开发并自动执行重复性任务。

Make 可以帮我们做很多事情：测试、构建、清理、安装 Go 项目。

## 创建项目并运行

首先我们先创建一个简单的项目，创建一个 main.go 文件。为了运行项目，我们需要构建项目并运行二进制文件：

```bash
go build main.go
```

当我们创建 Go 项目然后会遇到需要不同的二进制名称并且需要在特定的操作系统中创建构建，那么可以通过指定环境进行构建：

```bash
# 指定 macos系统
GOARCH=amd64 GOOS=darwin go build -o hello-darwin main.go
# 指定 Linux 操作系统
GOARCH=amd64 GOOS=linux go build -o hello-linux main.go

go run hello
```

如果在开发部署过程中，我们不仅要记住这些命令然后遇到不同环境输入命令执行。在这个过程中你可能会发生输错命令等行为。

使用 Makefile 可以协助我们高效工作，因为它可以帮助我们简化上述命令，甚至可以为特定命令指定规则并运行简单的 make 命令，这样不仅可以让你免去记住这些命令和环境的关系。

## 添加 Makefile 文件

在当前项目的根目录下创建一个 Makefile 文件，设置内容如下：

```makefile
BINARY_NAME=hello

build:
 GOARCH=amd64 GOOS=darwin go build -o hello-darwin main.go
 GOARCH=amd64 GOOS=linux go build -o hello-linux main.go
run:
 ./${BINARY_NAME}

build_and_run:build run
clean:
  go clean
  rm ${BINARY_NAME}-darwin
  rm ${BINARY_NAME}-linux
```

创建完 Makefile 文件后，就可以通过这些简单的命令进行编译和运行你的 Go 项目：

```bash
make run
make build
# 也可以使用在 makefile 中定义的一个命令：build_and_run
make build_and_run
# 使用清除命令清除二进制文件
make clean
```

以上命令相对于一开始的命令更简单，使用简单，也可以避免因输出命令导致的相关错误。

## Makefile

### 概念

make 命令都是来源于 Makefile 文件的。其都是由一系列的规则构成。每条规则都是由目标、依赖项、命令组成。

- 目标 **Target**：make 命令通过目标名称执行具体命令。如上的 make run

- 依赖项 **Dependencies**：目标可以具有需要在运行目标之前执行的依赖项

- 配方 **recipe**：运行目标时将执行的实际命令

### 变量

Makefiles 也有使用变量的机制。在以上的 Makefile 文件中，可以看到 ${BINARY_NAME} 的变量。所以当我们有相同的内容时可以通过添加变量进行替换。

可以使用 = 或 := 定义变量。 

= 将递归扩展变量。 这将替换它被替换时的值。以下例子在运行 all 命令时，它会将 x 的值替换为最后更新的值。因此将打印：`later bar`。 例如：

```makefile
x = foo
y = $(x) bar
x = later

all:
 echo $(y)
```

但是当你使用 := 进行变量赋值时，将打印第一次的值，比如：

```makefile
x := foo
y := $(x) bar
x := later

all:
 echo $(y)
> foo bar
```

更多使用 makefile 命令，可以参考：https://makefiletutorial.com/

## 使用 Makefile 自动化任务

在开发项目时，可以将一些测试，运行测试覆盖，管理依赖等工作，我们都可以创建一个 Makefile ，并在文件中包含自动化这些任务的所有规则：

```makefile
BINARY_NAME=hello

build:
 GOARCH=amd64 GOOS=linux go build -o ${BINARY_NAME}-linux main.go

run:
 ./${BINARY_NAME}

build_and_run: build run

clean:
 go clean
 rm ${BINARY_NAME}-linux


test:
 go test ./...

test_coverage:
 go test ./... -coverprofile=coverage.out

dep:
 go mod download

vet:
 go vet

lint:
 golangci-lint run --enable-all
```

使用这个简单的 Makefile，您现在可以轻松地执行命令来运行任务。这样就轻松自动化解决工作。

```bash
make test
make test_coverage
make dep
make vet
make lint
```

参考资料：

- [使用makefile构建golang项目_风花雪月无情王的博客-CSDN博客_golang makefile](https://blog.csdn.net/u010230971/article/details/80335613)

- [Makefiles for Go Developers | TutorialEdge.net](https://tutorialedge.net/golang/makefiles-for-go-developers/)
