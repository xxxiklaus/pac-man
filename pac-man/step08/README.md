# Step 08: Command line parameters

 you will learn how to:

- 将标志添加到命令行应用程序

## Overview

在上一步中，我们添加了一个配置文件 config.json 来处理表情符号翻译
还有一个名为“config_noemoji.json”的文件，它可以转换为游戏的原始表示

我们还使用 `maze01.txt` 文件来表示我们的迷宫.所有这些名称都直接写入源代码
但以硬编码方式处理这些文件并不理想，因此我们将对其进行更改.

## Task 01: Create flags for each file

标准库的 `flag` 包负责处理命令行标志
 我们将使用它来创建两个标志：`--config-file` 和 `--maze-file`

在文件的开头，就在导入之后，添加以下全局变量:

```go
var (
    configFile = flag.String("config-file", "config.json", "path to custom configuration file")
    mazeFile   = flag.String("maze-file", "maze01.txt", "path to a custom maze file")
)
```

```go
func main() {
    flag.Parse()

    // initialise game
    initialise()
    defer cleanup()

    // rest of the function omitted...
}
```

## Task 02: Replacing the hard coded files with the flags

我们已经处理了解析，现在需要用它们的标志等价物替换硬编码值

这是通过用标志值替换硬编码值来完成的（注意取消引用运算符，因为标志是指针）

In `main`:

```go
    // load resources
    err := loadMaze(*mazeFile)
    if err != nil {
        log.Println("failed to load maze:", err)
        return
    }

    err = loadConfig(*configFile)
    if err != nil {
        log.Println("failed to load configuration:", err)
        return
    }
```

现在尝试在命令行中运行：

```sh
go build
./step08 --help
```

你应该看到以下显示：

```sh
$ ./step08 --help
Usage of ./step08:
  -config-file string
        path to custom configuration file (default "config.json")
  -maze-file string
        path to a custom maze file (default "maze01.txt")
```

现在尝试先使用 `--config-file config_noemoji.json` 运行 `step08`，
然后再运行 `--config-file config.json` 以查看区别

也可以尝试将 `maze01.txt` 复制到一个新文件并对其进行编辑以进行实验

也许您现在可以创建自己的主题...尝试访问 [Full Emoji List](https://unicode.org/emoji/charts/full-emoji-list.html) 寻找灵感 :)

[Take me to step 09!](../step09/README.md)
