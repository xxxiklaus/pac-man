# Step 09: Buffer "The String Concatenation Slayer"

 you will learn how to:

- 使用缓冲区连接字符串

## Overview

在本步中，我们将为应用程序添加对多个生命的支持. 我们将更新碰撞跟踪代码
以减少生命数，而不是在碰撞时将生命数设置为 0. 我们还将跟踪起始玩家位置
以便在玩家死亡时在那里重生玩家.最后，我们将玩家表情符号添加到游戏记分牌
中以跟踪剩余生命数，而不是将生命数显示为整数值.

## Task 01: Create Point type and update Player struct to use Point type.

跟踪玩家的初始位置，以便在与幽灵碰撞后重置位置 
通过更新 sprite 结构以包含 `startRow` 和 `startCol` 属性来做到这一点

```go
type sprite struct {
    row      int
    col      int
    startRow int
    startCol int
}
```

在 `loadMaze` 函数中为我们的player(和Ghost)填充这些属性：

```go
func loadMaze() error {
    //...omitted for brevity

    for row, line := range maze {
        for col, char := range line {
            switch char {
            case 'P':
                player = sprite{row, col, row, col}
            case 'G':
                ghosts = append(ghosts, &sprite{row, col, row, col})
            case '.':
                numDots++
            }
        }
    }

    return nil
}
```

注:由于我们有额外的 `startRow` 和 `startCol` 属性，我们不能再对碰撞检测
进行简单的比较，因为玩家永远不会从与幽灵相同的位置开始

## Task 02: Update initial `lives` to be greater than 1 and decrement lives on ghost collision

将初始生命数设置为 3

```go
var lives = 3
```

然后,我们将更新处理碰撞的代码，以在每次发生碰撞时将生命数减 1。 最后，我们将检查以确保我们没有失去生命并将我们的玩家表情符号重置为初始位置以重新开始游戏.

```go
    // process collisions
    for _, g := range ghosts {
        if player.row == g.row && player.col == g.col {
            lives = lives - 1
            if lives != 0 {
                moveCursor(player.row, player.col)
                fmt.Print(cfg.Death)
                moveCursor(len(maze)+2, 0)
                time.Sleep(1000 * time.Millisecond) //dramatic pause before resetting player position
                player.row, player.col = player.startRow, player.startCol
            }
        }
    }
```

## Task 03: Update scoreboard to display Player emojis corresponding to number of lives

之前，生命数在游戏记分牌中显示为整数. 我们现在将更新记分牌以显示带有玩家表情符号的生命数.我们将添加一个 `getLivesAsEmoji` 函数，以根据游戏中剩余的生命连接正确数量的玩家表情符号。此函数创建一个缓冲区，然后根据生命数将玩家表情符号字符串写入缓冲区，然后将该值作为字符串返回.该函数在 `printScreen` 函数的最后一行被调用以更新记分板.

```go
func printScreen() {
    //...omitted for brevity

    moveCursor(len(maze)+1, 0)

    livesRemaining := strconv.Itoa(lives) //converts lives int to a string
    if cfg.UseEmoji {
        livesRemaining = getLivesAsEmoji()
    }

    fmt.Println("Score:", score, "\tLives:", livesRemaining)
}

//concatenate the correct number of player emojis based on lives
func getLivesAsEmoji() string{
    buf := bytes.Buffer{}
    for i := lives; i > 0; i-- {
        buf.WriteString(cfg.Player)
    }
    return buf.String()
}
```


[Take me to step 10!](../step10/README.md)
