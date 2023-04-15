# Step 04: Ghosts!

you will learn how to:

- 创建map（dictionary）
- 生成随机数
- 使用指针

## Overview

 现在我们可以移动我们的玩家了，再加点敌人（幽灵）.

使用与玩家相同的移动机制，即“makeMove”函数，但我们将使用一个简单的算法，而不是从键盘
读取输入：生成一个介于 0 和 3 之间的随机数，并为每个值分配一个方向 .

就算 Ghost 撞墙也没关系，它会在下一次迭代中重试.

## Task 01: Making Ghosts

就像创建了一个结构体来保存玩家数据一样，将幽灵创建一个类似的结构体。 唯一的区别是我们没有
在内存中保存玩家全局变量，而是有一片指向幽灵的指针。 这样就可以非常有效地更新每个幽灵的位置.


```go
var ghosts []*sprite
```
在 `loadMaze` 函数中，向 switch 语句添加一个新的 case，用于处理map上的 `G` 符号：

```go
for row, line := range maze {
    for col, char := range line {
        switch char {
        case 'P':
            player = sprite{row, col}
        case 'G':
            ghosts = append(ghosts, &sprite{row, col})  //&添加指针
        }
    }
}
```

由于在 `loadMaze` 函数中已处理 `G`了，还需在 `printScreen` 中打印它们.打印player后还需添加以下：

```go
for _, g := range ghosts {
    simpleansi.MoveCursor(g.row, g.col)
    fmt.Print("G")
}
```

## Task 02: A very smart AI

使用随机数生成器来控制我们的幽灵（有点复杂）：

```go
func drawDirection() string {
    dir := rand.Intn(4)
    move := map[int]string{
        0: "UP",
        1: "DOWN",
        2: "RIGHT",
        3: "LEFT",
    }
    return move[dir]
}
```

`math/rand` 包中的函数 `rand.Intn` 在区间 `[0, n)` 之间生成一个随机数，其中 `n` 是给函数的参数（半闭区间 不包括n）


使用 `map` 将整数映射到实际运动。 映射是一种将一个值映射到另一个值的数据结构. 
即，在上面的例子中，映射“move”将一个整数映射到一个字符串.

## Task 03: Let's add some movement!

最后,定义一个函数来处理幽灵运动。 `moveGhosts` 函数如下所示：

```go
func moveGhosts() {
    for _, g := range ghosts {
        dir := drawDirection()
        g.row, g.col = makeMove(g.row, g.col, dir)
    }
}
```

更新循环,调用函数moveGhost()

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
    moveGhosts()

    // process collisions

    // check game over
    if input == "ESC" {
        break
    }

    // repeat
}
```

[Take me to step 05!](../step05/README.md)
