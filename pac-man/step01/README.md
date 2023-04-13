# Step 01: Input and Output

 you will learn how to:

- 从文件中读取
- 打印到标准输出
- 处理多个返回值
- 处理错误
- 创建元素并将其添加到切片
- 在一片范围内循环
- 延迟函数调用
- 记录错误

## Overview

首先，我们要读取迷宫数据。 我们有一个名为 `maze01.txt` 的文件
它基本上是迷宫的 ASCII 表示（你可以根据需要在文本编辑器中打开它）

```
- # 代表一堵墙
- . 代表一个点
- P代表玩家
- G代表幽灵（敌人）
- X代表能量提升药丸
```

我们的第一个任务是将迷宫的 ASCII 表示形式加载到一段字符串中，然后将其打印到屏幕上

## Task 01: Load the Maze

让我们从阅读“maze01.txt”文件开始.

我们将使用 `os` 包中的 `Open` 函数打开它，并使用缓冲 IO 包 (`bufio`) 中的扫描器对象将其
读入内存（读入名为 `maze` 的全局变量）. 最后，我们需要通过调用 os.Close 来释放文件处理程序.

```go
var maze []string

func loadMaze(file string) error {
    f, err := os.Open(file)
    if err != nil {
        return err
    }
    defer f.Close()

    scanner := bufio.NewScanner(f)
    for scanner.Scan() {
        line := scanner.Text()
        maze = append(maze, line)
    }

    return nil
}
```

`loadMaze` 代码的另一个有趣方面是使用 `defer` 关键字. 即前面有defer最后运行.
 它对于清理目的非常有,在这种情况下,我们使用它来关闭我们刚刚打开的文件：

```go
func loadMaze(file) error {
    f, err := os.Open(file)
    // omitted error handling
    defer f.Close() // puts f.Close() in the call stack

    // rest of the code

    return nil
    // f.Close is called implicitly
}
```

这部分只是逐行读取文件并将其附加到迷宫切片:

```go
    scanner := bufio.NewScanner(f)
    for scanner.Scan() {
        line := scanner.Text()
        maze = append(maze, line)
    }
```

 当有内容要从文件中读取时，`scanner.Scan()` 将返回 true,而 `scanner.Text()` 将返回下一行输入.

`append` 内置函数负责向 `maze` 切片添加新元素.

## Task 02: Printing to the Screen

一旦我们将迷宫文件加载到内存中,我们就需要将它打印到屏幕上.

我用的方法是遍历"迷宫"切片中的每个条目并打印它.使用`for range`循环来完成：

```go
func printScreen() {
    for _, line := range maze {
        fmt.Println(line)
    }
}
```

如果我们没有写下划线`_`字符来忽略第一个值，则范围运算符将只返回索引（而不是值）. eg：

```go
for idx := range maze {
    fmt.Println(idx)
}
```
因为在这种情况下我们只关心内容而不关心索引,所以我们可以通过将索引分配给下划线来安全地忽略索引.

## Task 03: Updating the game loop

现在我们有了 `loadMaze` 和 `printScreen` 函数,我们更新 `main` 函数来初始化迷宫并在游戏循环中打印它.

```go
func main() {
    // initialise game

    // load resources
    err := loadMaze("maze01.txt")
    if err != nil {
        log.Println("failed to load maze:", err)
        return
    }

    // game loop
    for {
        // update screen
        printScreen()

        // process input

        // process movement

        // process collisions

        // check game over

        // Temp: break infinite loop
        break

        // repeat
    }
}
```
现在我们已经完成了game loopd的修改,我们可以用 go run 来跑一下或用 go build 编译它并将其作为独立程序运行.

```sh
go run main.go
```

[Take me to step 02!](../step02/README.md)
