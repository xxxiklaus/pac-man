# Step 10: Ghosts with power ups!

 you will learn how to:

- 使用定时器
- 何时以及如何使用互斥锁

## Overview

此步骤中，将为应用程序添加对 power up pill 的支持。 我们将使用新设置更新配置并添加代码以在迷宫中绘制药丸.吃豆人吞下一颗药丸并与Ghost相撞后，还将管理该过程.最后，处理吃豆人在前一个药丸处于活动状态时试图吞下药丸的情况，以及如何解决这个问题.

## Task 01: Drawing the Pills

在开始之前，应该更新配置以支持 power up pills 因此，对于 `config_noemoji.json` 和 `config.json`，我们必须添加 `ghost_blue`（字符串）和 `pill_duration_secs`（整数）配置.因此更新 `Config` 结构：

```go
type config struct {
    ...
	GhostBlue        string        `json:"ghost_blue"`
	PillDurationSecs time.Duration `json:"pill_duration_secs"`
}
```

## Task 02: Enable Pill swallowing

为了让 吃豆人 吞下药丸，我们应该在 movePlayer 函数中为药丸盒添加另一个 case

```go
case 'X':
	score += 10
	removeDot(player.row, player.col)
	go processPill()
```
Where `X` is the pill config character. 

现在，在移动到 `processPill` 函数之前，应该为幽灵添加更多代码以支持“GhostBlue”！ 应该添加一个新的字符串类型的“GhostStatus”，它将保存幽灵的状态,必须支持的两种状态是`Normal`和`Blue`


```go
type GhostStatus string

const (
	GhostStatusNormal GhostStatus = "Normal"
	GhostStatusBlue   GhostStatus = "Blue"
)
```

现在，每个Ghost都应该与它的当前位置保持一致，即它被吃豆人吃掉后生成的 `initialPosition` 以及它的当前状态.

```go
type ghost struct {
	position sprite
	status   GhostStatus
}
```
因此，`loadMaze` 函数最初将以`Normal` 状态绘制幽灵并存储其初始位置:

```go
ghosts = append(ghosts, &ghost{sprite{row, col, row, col}, GhostStatusNormal})
```

`printScreen` 函数也应该更新以支持两种类型去打印Ghost 
- GhostNormal和GhostBlue！

```go
for _, g := range ghosts {
		moveCursor(g.position.row, g.position.col)
		if g.status == GhostStatusNormal {
			fmt.Printf(cfg.Ghost)
		} else if g.status == GhostStatusBlue {
			fmt.Printf(cfg.GhostBlue)
		}
	}
```

最后剩下的就是之前添加的 `processPill` 函数 此函数应在“PillDurationSecs”配置定义的时间段内将所有幽灵的状态更改为`Blue`

对于药丸处理，将使用 ['time' 包](https://golang.org/pkg/time/) 中的“Timer”
使用 `NewTimer` 函数创建一个新的计时器，该计时器将在至少指定的持续时间后在
其通道上发送当前时间.

processPill 代码将所有幽灵的状态更改为“GhostStatusBlue”，然后阻塞“PillDurationSecs”，然后将所有幽灵的状态更改回“GhostStatusNormal”.

```go
var pillTimer *time.Timer

func processPill() {
	for _, g := range ghosts {
		g.status = GhostStatusBlue
	}
	pillTimer = time.NewTimer(time.Second * cfg.PillDurationSecs)
	<-pillTimer.C
    for _, g := range ghosts {
		g.status = GhostStatusNormal
    }
}
```

## Task 03: Support simultaneous pill swallowing

 `processPill` 函数有一个简单的问题.如果 pacman 试图吞下一颗能量药丸，而另一颗药丸仍处于活动状态，会发生什么情况? 
 
目前，使用提议的 `processPill` 函数，当pacman吞下第二颗药丸时，当第一颗药丸仍然有效时，当第一颗药丸的效果结束时（在 PillDurationSecs 之后），所有幽灵将恢复正常.为了克服这个问题，我们应该通过检查计时器来检查药丸是否已经激活，然后停止它并在它已经激活时重新初始化它.

```go
var pillTimer *time.Timer

func processPill() {
	updateGhosts(ghosts, GhostStatusBlue)
	if pillTimer != nil {
		pillTimer.Stop()
	}
	pillTimer = time.NewTimer(time.Second * cfg.PillDurationSecs)
	<-pillTimer.C
	pillTimer.Stop()
	updateGhosts(ghosts, GhostStatusNormal)
}
```

## Task 04: Avoiding Race Conditions

在游戏场景中，有两种可能的竞争条件. 第一个是刚才提到的药丸计时器 `processPill` 函数被异步调用.因此，如果第一个 processPill 函数紧跟在 pillTimer.Stop() 之后，而第二个函数在 `if pillTimer != nil { `块内.在这种罕见的情况下，似乎当一颗药丸处于活动状态时，在代码处于此时消耗下一颗药丸时，可能会松开第二颗药丸，因为幽灵会恢复正常

出于这个原因，需要引入一个 pillMx 互斥锁来解决，在 `processPill` 函数开始时获取它，并在开始等待计时器通道之前释放它
此外，我们将在阻塞函数之后立即获取它并在函数结束时释放它.

```go
var pillTimer *time.Timer
var pillMx sync.Mutex

func processPill() {
	pillMx.Lock()
	updateGhosts(ghosts, GhostStatusBlue)
	if pillTimer != nil {
		pillTimer.Stop()
	}
	pillTimer = time.NewTimer(time.Second * cfg.PillDurationSecs)
	pillMx.Unlock()
	<-pillTimer.C
	pillMx.Lock()
	pillTimer.Stop()
	updateGhosts(ghosts, GhostStatusNormal)
	pillMx.Unlock()
}
```

执行期间可能出现的另一种可能的竞争条件是当我们更新幽灵的状态时. 为此，还需使用 RWMutex 锁. 每当读取或更新幽灵的状态时，都必须获取锁. RWMutex 甚至支持读或写访问的锁定. 因此，我们引入了 `var ghostsStatusMx sync.RWMutex` 和更新一个或多个 ghost 状态的 `updateGhosts` 函数.

```go 
var ghostsStatusMx sync.RWMutex

func updateGhosts(ghosts []*Ghost, ghostStatus GhostStatus) {
	ghostsStatusMx.Lock()
	defer ghostsStatusMx.Unlock()
	for _, g := range ghosts {
		g.status = ghostStatus
	}
}
```

每当读取幽灵的状态时，也必须获得一个 RLock.即可以同时获取多个读锁，但只能获取一个写锁. 我们将在读取幽灵状态时使用`ghostsStatusMx.RLock()` 和`ghostsStatusMx.RUnlock()`. 在更新幽灵的状态之前，必须始终解锁 RLock，否则会发生死锁的情况.

Now we have a more challenging pacman! Happy gaming/coding! :) 

## Congratulations! 

如果你有兴趣为新步骤做出贡献或任何未解决的问题并提交 PR！