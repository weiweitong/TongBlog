---
layout:     post

title:      Golang--Channel--Deadlock

date:       2018-12-11

author:     Augustine Tong

header-img: img/steve.jpg

catalog: true

tags:
    - Go
---

# Go-Channel--Deadlock
**代码均为自己手打过一次, 输出结果均为自己的输出结果**.  

流入与流出不配对就可能发生死锁  
为了不发生死锁：  
1. 明确保证流入和流出成对出现.  
2. 当明确无法保证流入和流出成对出现时，在流入完成后，关闭channel.  
3. 利用缓冲.  

## 1 Channels

```c
func main() {

    messages := make(chan string)

    go func() {messages <- "ping"}()

    msg := <-messages
    fmt.Println(msg)
}

Output:  
ping

```


## 2 Channel Buffering

```c
func main() {

    messages := make(chan string, 2)

    messages <- "buffered"
    messages <- "channel"

    fmt.Println(<-messages)
    fmt.Println(<-messages)
}

Output:  
buffered
channel


```


## 3 Channel Buffering Deadlock

```c
func chanBuDeadLockF2(ch chan int){
    fmt.Println(<-ch)
}

func main() {
    out := make(chan int, 1)
    out <- 2
    out <- 3

    go chanBuDeadLockF2(out) // 上一句已阻塞，不会执行
}


Output:  
ERROR

```


## 4 Channel Buffering Deadlock Solution

```c
import (
    "fmt"
    "time"
)

func chanBDLF2(ch chan int){
    fmt.Println(<-ch)
}

func main() {

    out := make(chan int, 10)

    go func() {
        out <- 2
        out <- 3
    }()

    go chanBDLF2(out)
    go chanBDLF2(out)
    time.Sleep(1e6)
}

Output:  
2
3

```


## 5 Channel Range Version 1.0

```c
func main() {
    var ch = make(chan int, 3)
    ch <- 1
    ch <- 2
    ch <- 3

    for v := range ch {
        fmt.Println(v)
        if len(ch) <= 0 {
            break
        }
    }
}


Output:  
1
2
3


```


## 6 Channel Range Version 2.0

```c
func main() {
    ch := make(chan int, 3)
    ch <- 1
    ch <- 2

    close(ch) // 显式地关闭信道

    for v := range ch {
        fmt.Println(v)
    }
}

Output:  
1
2

```


## 7 Channel Unbuffered Deadlock
```c
func deadF1(in chan int){
    fmt.Println(<-in)
}

func main() {
    out := make(chan int)
    out <- 2 // dead lock, block the following code
    // 没有其他goroutine 在接收，直接报错
    go deadF1(out) // 不会执行
}

Output:  
ERROR

```


## 8 Channel Unbuffered Deadlock Solution Version 1.0

```c
func chanUnSo1F1(in chan int)  {
    fmt.Println(<-in)
}

func main() {
    out := make(chan int)
    go chanUnSo1F1(out)
    out <- 2
}

Output:  
2

```


## 9 Channel Unbuffered Deadlock Solution Version 2.0

```c
import (
    "fmt"
    "time"
)

func chanUnSo2F1(in chan int){
    fmt.Println(<-in)
}

func main() {
    out := make(chan int)
    go func() {out <- 2}()
    go chanUnSo2F1(out)
    time.Sleep(1e6)
}


Output:  
2


```


## 10 Deadlock

```c
var cha1 = make(chan int)
var cha2 = make(chan int)

func say(s string){
    fmt.Println(s)
    cha1 <- <- cha2
}

func main() {
    go say("hello")
    cha2 <- 5 // 没有此行必死锁
    <- cha1
}

Output:  
hello

```


## 11 Foo Main

```c
var ch1 = make(chan int)
// 这里只能用var， 不能用 :=

func foo()  {
    ch1 <- 10
}
func main() {
    go foo()
    fmt.Println(<-ch1)
}


Output:  
10

```


## 12 For Deadlock

```c
func main() {
    ch := make(chan int)
    results := make(chan int)

    for i := 0; i < 2; i++ {
        go func() {
            // 把从channel里取得的数据， 再传回去
            x := <-ch   
            results <- x
        }()
    }

    // 向输入数据中传两个数据
    ch <- 1
    ch <- 2

    for re := range results{
        fmt.Printf("re:%v\n", re)
    } // 主程序在处理完1 和 2 两条数据后，还在一直等待results channel的内容
    // 无法结束，这样的话，Go判断产生了死锁
}

Output:  
ERROR

```


## 13 For Deadlock Solution 1.0

```c
import (
    "fmt"
    "time"
)

func main() {
    ch := make(chan int)
    results := make(chan int)

    for i := 0; i < 2; i++ {
        go func() {
            // 把从channel里取得的数据， 再传回去
            x := <-ch
            results <- x
        }()
    }

    // 向输入数据中传两个数据
    ch <- 1
    ch <- 2

    // 把 results channel 的接受处理，放到一个goroutine里去做
    // 这样的话，主程序就不会卡住不动，即产生死锁了
    go func() {
        for re := range results {
            fmt.Printf("re:%v\n", re)
        }
    }()

    // 等待3秒后，才结束主程序
    // 如果不加等待的话， 上面的go func 可能没有执行
    // 程序就结束了， go func就跟着结束了
    time.Sleep(1 * time.Second)
}


Output:  
re:1
re:2


```


## 14 For Deadlock Solution 2.0

```c
import (
    "fmt"
    "sync"
    "time"
)

func main() {
    var wg sync.WaitGroup
    ch := make(chan int)
    results := make(chan int)

    for i := 0; i < 2; i++ {
        wg.Add(1)
        go func() {
            // 把从channel里取得的数据，再传回去
            x := <-ch
            results <- x

            defer wg.Done()
        }()

    }

    // 在（for i := 0; i < 2; i++) 循环里的go func 都执行完后
    // 关闭 result channel , 这样主程序里的for results循环就可以正常结束了

    go func() {
        wg.Wait()
        close(results)
    }()

    ch <- 1
    ch <- 2

    for re := range results{
        fmt.Printf("re:%v\n", re)
    }

    // 在等待3秒后，才结束主程序
    // 如果不加等待的话，上面的go func 可能还没有执行
    // 程序就结束了， go func 跟着结束
    time.Sleep(1e6)
}

/*
最上面死锁的原因是，for result循环一直在等待输入，但实际上输入只有2个，
处理完后就没有了。为了解决这个问题，把 results channel 关闭，
for result循环就可以正常退出了。
具体细节是，在所有的 results channel 的输入处理之前，
wg.Wait()这个goroutine会处于等待状态。
当处理完后（wg.Done），
wg.Wait()就会放开执行，执行后面的close(results)

注意：要把wg.Add(1)放到go func外面。
如果放到里面的话，
(for i := 0; i < 2; i++)里的go func有可能会在wg.wait的go func之后执行
这样close(results)就会先执行
 */


Output:  
re:1
re:2

```


## 15 Loop

```c


Output:  


```


## 

```c


Output:  


```
