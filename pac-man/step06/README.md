# Step 06: Making things real(time)

you will learn how to:

- 使用协程
- 使用匿名函数（lambdas）
- 使用频道
- 使用 select 语句异步读取通道
- 使用时间包

## Overview

通过步骤5,吃豆人游戏的雏形已经有了,但有个问题Ghost不会自主移动
它是随着player的移动而移动的,在此步将完善它.

发生此问题是因为读取输入是阻塞操作,需要以某种方式使其异步,即协程的作用

## Task 01: Refactoring the input code

 首先,从游戏循环中删除输入处理代码，并在循环开始之前插入下面的代码。

```go
func main() {
    // init code omitted for brevity...

    // process input (async)
    input := make(chan string)
    go func(ch chan<- string) {
        for {
            input, err := readInput()
            if err != nil {
                log.Println("error reading input:", err)
                ch <- "ESC"
            }
            ch <- input
        }
    }(input)

    // game loop
    for {
        // loop code...
    }
}
```

此代码将创建一个名为`input`的通道，并将其作为参数传递给使用“go”语句调用的匿名函数

```go
// process movement
select {
case inp := <-input:
    if inp == "ESC" {
        lives = 0
    }
    movePlayer(inp)
default:
}
```
select 语句就像 switch 语句，但用于通道。 此 select 语句具有非阻塞性质，因为它有一个默认子句。 这意味着如果 `input` 通道有要读取的内容，它将被读取，否则将处理 `default` 情况.

引入 200 毫秒的延迟防止游戏将运行得太快：

```go
    // update screen
    printScreen()

    // check game over
    if numDots == 0 || lives <= 0 {
        break
    }

    // repeat
    time.Sleep(200 * time.Millisecond)
```
[Take me to step 07!](../step07/README.md)
