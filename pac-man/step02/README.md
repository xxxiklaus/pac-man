# Step 02: Handling Player Input

 you will learn how to:

- 使用不同的终端模式
- 从 Go 代码调用外部命令
- 向终端发送转义序列
- 从标准输入读取
- 创建一个返回多个值的函数

## Overview

这一步将处理 Esc 键,我们将在步骤 03 中看到如何处理箭头键.

在这个游戏中,我们将处理一组受限的运动：上、下、左、右。 除此之外,我们将
使用的唯一另一个键是Esc键,以使玩家能够优雅地退出游戏. 移动将映射到箭头键.

## Intro to terminal modes

在开始实施之前，我们需要了解一些终端模式.

终端可以以三种可能的运行[模式](https://en.wikipedia.org/wiki/Terminal_mode):

1. Cooked Mode
2. Cbreak Mode
3. Raw Mode

Cooked Mode是我们习惯使用的模式。 在这种模式下，终端接收到的
每个输入都经过预处理，这意味着系统会拦截特殊字符以赋予它们特殊的含义.

注：特殊字符包括 backspace, delete, Ctrl+D, Ctrl+C, 方向键等...

Raw mode恰恰相反：数据按原样传递，没有任何预处理.

Cbreak Mode:有些字符经过预处理，有些则没有.eg:Ctrl+C 仍然导致程序中止，但箭头键按原样传递给程序.

我们将使用 Cbreak Mode 来处理与转义键和方向键对应的转义序列。

## Task 01: Enabling Cbreak Mode

为了启用 Cbreak Mode ,我们将调用一个控制终端行为的外部命令即`stty`命令
我们还将禁用终端回显,这样我们就不会用按键输出污染屏幕.

以下是我的 init 的定义:

```go
func initialise() {
    cbTerm := exec.Command("stty", "cbreak", "-echo")
    cbTerm.Stdin = os.Stdin

    err := cbTerm.Run()
    if err != nil {
        log.Fatalln("unable to activate cbreak mode:", err)
    }
}
```

## Task 02: Restoring Cooked Mode

Restoring the cooked mode ：
```go
func cleanup() {
    cookedTerm := exec.Command("stty", "-cbreak", "echo")
    cookedTerm.Stdin = os.Stdin

    err := cookedTerm.Run()
    if err != nil {
        log.Fatalln("unable to restore cooked mode:", err)
    }
}
```

在 `main` 函数中调用这两个函数：

```go
func main() {
    // initialise game
    initialise()
    defer cleanup()

    // load resources
    // ...
```

## Task 03: Reading from Stdin

从标准输入读取的过程涉及使用给定的读取缓冲区调用函数`os.Stdin.Read`.

`os.Stdin.Read` 返回两个值：读取的字节数和错误值

```go
func readInput() (string, error) {
    buffer := make([]byte, 100)

    cnt, err := os.Stdin.Read(buffer)
    if err != nil {
        return "", err
    }

    if cnt == 1 && buffer[0] == 0x1b {
        return "ESC", nil
    }

    return "", nil
}
```

`make` 函数是一个[内置函数](https://golang.org/pkg/builtin/#make)，用于分配和初始化对象
它仅用于slice、map和channel 在本例中，我们创建了一个大小为 100 的字节数组,并返回一个指向它的slice.

在通常的错误处理之后（将错误向上传递到调用堆栈）测试是否只读取一个字节以
及该字节是否是转义键。 （0x1b 是代表 Esc 的十六进制代码）

## Task 04: Updating the Game Loop

更新游戏循环,确保每次迭代都调用 readInput 函数. 注：如果发生错误,我们也需要中断循环.

```go
// process input
input, err := readInput()
if err != nil {
    log.Print("error reading input:", err)
    break
}
```

摆脱那个永久性的“break”语句并开始测试“ESC”键.

```go
if input == "ESC" {
    break
}
```

## Task 05: Clearing the Screen

由于现在有一个适当的游戏循环,需要在每次循环后清除屏幕以便在下一次迭代中有一个空白屏幕用于绘制.
 为此，我们将使用一些特殊的“转义序列”.

[Escape sequences](https://en.wikipedia.org/wiki/ANSI_escape_code#Escape_sequences) 之所以这样称呼，是因为它们以 ESC 字符 (0x1b) 开头，后跟一个或多个字符。 这些字符用作终端仿真器的命令.

导入名为 simpleansi 的包来完成工作：

```go
import "github.com/danicat/simpleansi" //外部拓展包
```


更新 print Screen 函数以在打印前调用 `simple ansi.Clear Screen`确保每帧都使用空白屏幕：

```go
func printScreen() {
    simpleansi.ClearScreen()
    for _, line := range maze {
        fmt.Println(line)
    }
}
```

现在再次运行游戏并尝试按下“ESC”键

注：如果偶然按下 Ctrl+C，程序将在不调用清理函数的情况下终止，因此将无法在终端中看到正在输入的内容（因为 `-echo` 标志.

遇到这种情况，要么关闭终端并重新打开它，要么再次运行游戏并使用“ESC”键正常退出.

[Take me to step 03!](pac-man/step03/README.md)
