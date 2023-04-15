# Step 03: Adding Movement

 you will learn how to:

- 创建一个结构
- 使用 switch 语句
- 处理方向键
- 使用命名的返回值

## Overview

现在我们有了一个迷宫,我们可以优雅地退出游戏……但我们还需增加一些功能来让它更有趣.

在此步骤中,将添加player并使用箭头键启用其移动.

## Task 01: Tracking player position

首先是创建一个变量来保存玩家数据. 由于我们将跟踪二维坐标（行和列），我们将定义一个结构体来保存该信息：

```go
type sprite struct {
    row int
    col int
}

var player sprite
```

简单起见,依然将玩家定义为全局变量.

接下来需要在加载迷宫后立即捕获玩家位置，在 `loadMaze` 函数中：

```go
// traverse each character of the maze and create a new player when it locates a `P`
for row, line := range maze {
    for col, char := range line {
        switch char {
        case 'P':
            player = sprite{row, col}
        }
    }
}
```

注：此次使用的是“范围”运算符的完整形式，找到玩家的行和列。

```go
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

    for row, line := range maze {
        for col, char := range line {
            switch char {
            case 'P':
                player = sprite{row, col}
            }
        }
    }

    return nil
}
```

---

### Optional: A note about visibility

Go 在定义可见性方面有一个有趣的机制 它没有使用 public 关键字，而是将
名称以大写字母开头的每个符号视为 public.名称以小写字符开头为私有符号.

---

## Task 02: Handling arrow key presses

接下来，需要修改 readInput 来处理方向键：

```go
if cnt == 1 && buffer[0] == 0x1b {
    return "ESC", nil
} else if cnt >= 3 {
    if buffer[0] == 0x1b && buffer[1] == '[' {
        switch buffer[2] {
        case 'A':
            return "UP", nil
        case 'B':
            return "DOWN", nil
        case 'C':
            return "RIGHT", nil
        case 'D':
            return "LEFT", nil
        }
    }
}
```

方向键的转义序列长 3 个字节，以“ESC+[”开头，然后是从 A 到 D 的字母.

定义一个函数来处理运动：

```go
func makeMove(oldRow, oldCol int, dir string) (newRow, newCol int) {
    newRow, newCol = oldRow, oldCol

    switch dir {
    case "UP":
        newRow = newRow - 1
        if newRow < 0 {
            newRow = len(maze) - 1
        }
    case "DOWN":
        newRow = newRow + 1
        if newRow == len(maze) {
            newRow = 0
        }
    case "RIGHT":
        newCol = newCol + 1
        if newCol == len(maze[0]) {
            newCol = 0
        }
    case "LEFT":
        newCol = newCol - 1
        if newCol < 0 {
            newCol = len(maze[0]) - 1
        }
    }

    if maze[newRow][newCol] == '#' {
        newRow = oldRow
        newCol = oldCol
    }

    return
}
```

上面的函数利用“命名返回值”在移动后返回新位置（“newRow”和“newCol”）.
 基本上，该函数首先“尝试”移动，如果新位置碰壁（`#`），移动将被取消.

定义一个函数来移动player：

```go
func movePlayer(dir string) {
    player.row, player.col = makeMove(player.row, player.col, dir)
}
```

## Task 03: Updating the maze

我们已经准备好所有的移动逻辑，但我们需要让屏幕反映出来。 我们将重构 `printScreen` 函数
以仅打印我们想要打印的内容， 而不是the whole map.

这将为我们提供更多控制，使我们能够使用“moveCursor”函数在任意位置打印player：

```go
func printScreen() {
    simpleansi.ClearScreen()
    for _, line := range maze {
        for _, chr := range line {
            switch chr {
            case '#':
                fmt.Printf("%c", chr)
            default:
                fmt.Print(" ")
            }
        }
        fmt.Println()
    }

    simpleansi.MoveCursor(player.row, player.col)
    fmt.Print("P")


    // Move cursor outside of maze drawing area
    simpleansi.MoveCursor(len(maze)+1, 0)
}
```

目前，我们忽略任何不是墙或玩家的东西.

## Task 04: Animation!

最后，需要从the game loop中调用“movePlayer”：

```go
// game loop
for {
    // update screen
    printScreen()

    // process input
    input, err := readInput()
    if err != nil {
        log.Println("error reading input:", err)
        break
    }

    // process movement
    movePlayer(input)

    // process collisions

    // check game over
    if input == "ESC" {
        break
    }

    // repeat
}
```

[Take me to step 04!](pac-man/step04/README.md)
