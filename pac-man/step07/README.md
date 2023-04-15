# Step 07: Finally, emojis!

 you will learn how to:

- 加载一个 json 文件
- 打印表情符号！

## Overview

经过前6步已经设法在终端中创建了一个合适的游戏,这一步将加入趣味性的emoji！

在此步骤中，将创建一个名为“config.json”的文件.在这个文件中，将存储我们在
游戏中使用的每个符号的映射.在2D游戏中，我们通常将移动的棋子称为“sprite”

现在大多数终端基本都支持 unicode,可以使用表情符号作为sprite,而非借助其他图形库


```json
{
    "player": "😋",
    "ghost": "👻",
    "wall": "🌵",
    "dot": "🧀",
    "pill": "🍹",
    "death": "💀",
    "space": "  ",
    "use_emoji": true
}
```

配置文件重要的是“use_emoji”配置.当我们使用表情符号时，我们使用此标志向游戏发出信号 这是必要的，因为表情符号通常在屏幕上占用多个字符（大多数使用 2 个）.

使用该标志，我们可以有备用代码路径来进行调整以补偿该差异。 否则迷宫看起来会扭曲.

## Task 01: Load a json

我们首先需要定义一个结构来保存 json 数据。 反引号 (\`) 之间的文本称为“结构标签” json 解码器使用它来了解 struct 的哪个字段对应于 json 文件中的每个字段

```go
// Config holds the emoji configuration
type Config struct {
    Player   string `json:"player"`
    Ghost    string `json:"ghost"`
    Wall     string `json:"wall"`
    Dot      string `json:"dot"`
    Pill     string `json:"pill"`
    Death    string `json:"death"`
    Space    string `json:"space"`
    UseEmoji bool   `json:"use_emoji"`
}

var cfg Config
```

注: `Config` 结构使用了公共成员.这是 json 解码器工作所必需的

解析 json 并将其存储在 `cfg` 全局变量中:

```go
func loadConfig(file string) error {
    f, err := os.Open(file)
    if err != nil {
        return err
    }
    defer f.Close()

    decoder := json.NewDecoder(f)
    err = decoder.Decode(&cfg)
    if err != nil {
        return err
    }

    return nil
}
```

在主函数中loadMaze 之后添加 loadConfig 调用：

```go
err = loadConfig("config.json")
if err != nil {
    log.Println("failed to load configuration:", err)
    return
}
```

## Task 02: Adjusting the horizontal displacement

定义一个 `moveCursor` 函数来纠正设置表情符号标志时的水平位移：

```go
func moveCursor(row, col int) {
    if cfg.UseEmoji {
        simpleansi.MoveCursor(row, col*2)
    } else {
        simpleansi.MoveCursor(row, col)
    }
}
```

将 col 值缩放 2 倍将确保我们将每个字符放置在正确的位置，同时使迷宫看起来更大

## Task 03: Replace hardcoded characters with configuration

最后一部分是将硬编码字符替换为 printScreen 函数中的配置对应字符, 我们还将使用 `simpleansi.WithBlueBackground` 函数更改墙壁的颜色，使其更能代表原始游戏.

```go
func printScreen() {
    simpleansi.ClearScreen()
    for _, line := range maze {
        for _, chr := range line {
            switch chr {
            case '#':
                fmt.Print(simpleansi.WithBlueBackground(cfg.Wall))
            case '.':
                fmt.Print(cfg.Dot)
            default:
                fmt.Print(cfg.Space)
            }
        }
        fmt.Println()
    }

    moveCursor(player.row, player.col)
    fmt.Print(cfg.Player)

    for _, g := range ghosts {
        moveCursor(g.row, g.col)
        fmt.Print(cfg.Ghost)
    }

    moveCursor(len(maze)+1, 0)
    fmt.Println("Score:", score, "\tLives:", lives)
}
```

## Task 04: Game over and pills

```go
// check game over
if numDots == 0 || lives == 0 {
    if lives == 0 {
        moveCursor(player.row, player.col)
        fmt.Print(cfg.Death)
        moveCursor(len(maze)+2, 0)
    }
    break
}
```

此外，将增强药丸视为一个值更多分的点，作为实际增强机制的占位符:

```go
func movePlayer(dir string) {
    player.row, player.col = makeMove(player.row, player.col, dir)

    removeDot := func(row, col int) {
        maze[row] = maze[row][0:col] + " " + maze[row][col+1:]
    }

    switch maze[player.row][player.col] {
    case '.':
        numDots--
        score++
        removeDot(player.row, player.col)
    case 'X':
        score += 10
        removeDot(player.row, player.col)
    }
}
```

关于上面的代码，有一个有趣的地方是我们正在定义一个内联函数，以便在发生碰撞时从游戏中删除点和 X。 我们也可以重复代码，这会使其更具可读性和可维护性.


[Take me to step 08!](../step08/README.md)
