# Step 05: Game over man, game over!

 you will learn how to:

- 在switch中使用 fallthrough 语句
- 使用slice

## Overview

这步给这个游戏增加一些难度.此外,即设置清除棋盘上的所有圆点为游戏获胜条件.

## Task 01: Preparation

基于游戏获胜条件,需要跟踪棋盘上有多少点,并在该点数为零值时宣布获胜.

一旦Player站在圆点上方,即从棋盘上移除该圆点.同时将跟踪分数以打印出来显示给玩家.

对于游戏结束条件,我将给予Player一条命,当Ghost击中他时，这条生命将归零. 
然后在游戏循环中测试,若生命值为零值游戏终止. 

添加以下全局变量以跟踪以上所有内容:

```go
var score int
var numDots int
var lives = 1
```

接下来，需要在 loadMaze 中初始化 numDots 变量.只需在原有基础上
处理 `.` 字符的开关中插入一个新的 case：

```go
for row, line := range maze {
    for col, char := range line {
        switch char {
        case 'P':
            player = sprite{row, col}
        case 'G':
            ghosts = append(ghosts, &sprite{row, col})
        case '.':
            numDots++
        }
    }
}
```

更新 `printScreen` 函数来再次打印这些点

```go
func printScreen() {
    simpleansi.ClearScreen()
    for _, line := range maze {
        for _, chr := range line {
            switch chr {
            case '#':
                fallthrough
            case '.':
                fmt.Printf("%c", chr)
            default:
                fmt.Print(" ")
            }
        }
        fmt.Println()
    }
    // rest of the function omitted for brevity...
}
```

最后,在 `printScreen` 函数的末尾添加我们获得的分数和生命面值:

```go
func printScreen() {
    // code omitted...

    // print score
    simpleansi.MoveCursor(len(maze)+1, 0)
    fmt.Println("Score:", score, "\tLives:", lives)
}
```

## Task 02: Game over

在任何给定的时刻，如果Player和Ghost在同一个地方，即Ghost击中Player. 
将检测到这一点的代码添加到游戏循环中.

并完善游戏退出条件，添加 `lives <= 0` 和 `numDots == 0`：

```go
// game loop
for {
    // code ommited...

    // process collisions
    for _, g := range ghosts {
        if player == *g {
            lives = 0
        }
    }

    // check game over
    if input == "ESC" || numDots == 0 || lives <= 0 {
        break
    }

    // repeat
}
```

注:检查Player位置的更详细的方法: `player.row == g.row && player.col == g.col`
但由于Player和Ghost都属于是Sprites,他们可以使用简单的比较 `player = = *g`.
我们仍然需要取消引用 `g`，因为我们无法比较指针和非指针类型.

## Task 03: Game win

把从游戏中删除点并增加分数的这段代码添加到“movePlayer”函数中：

```go
func movePlayer(dir string) {
    player.row, player.col = makeMove(player.row, player.col, dir)
    switch maze[player.row][player.col] {
    case '.':
        numDots--
        score++
        // Remove dot from the maze
        maze[player.row] = maze[player.row][0:player.col] + " " + maze[player.row][player.col+1:]
    }
}
```

注：Go中的字符串是不可变的. 不能简单地将空格分配给字符串中的给定位置，这行不通.

因此，这里使用了一个技巧，即创建一个由原始字符串的两个片段组成的新字符串.
这两个切片一起构成完全相同的原始字符串，除开一个位置，我们用一个空格来替换它.


[Take me to step 06!](../step06/README.md)
