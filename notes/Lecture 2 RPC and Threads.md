**[课程首页](https://pdos.csail.mit.edu/6.824/index.html)**

**[课程安排](https://pdos.csail.mit.edu/6.824/schedule.html)**

##  **Lecture  2:** [RPC and Threads](https://pdos.csail.mit.edu/6.824/notes/l-rpc.txt)

### Threads

[Go example: crawler.go](https://pdos.csail.mit.edu/6.824/notes/crawler.go)

#### Serial crawler

```
can we just put a "go" in front of the Serial() call?
let's try it... what happened?
```

​		这种想法与我在阅读tutorial时想到的一致，但当时我并不知道为什么不奏效。实际上程序在尝试创建多线程进行并行爬取时，由于主线程迅速执行完毕且退出，被创建的多线程还来不及进行输出，所以最终在输出中我们只有一条网址，而被创建的多线程输出在了之后的内容中。

#### ConcurrentMutex crawler

```
How does the ConcurrentMutex crawler decide it is done?
    sync.WaitGroup
    Wait() waits for all Add()s to be balanced by Done()s
      i.e. waits for all child threads to finish
    [diagram: tree of goroutines, overlaid on cyclic URL graph]
    there's a WaitGroup per node in the tree
```

### Remote Procedure Call (RPC)

[Go example: kv.go](https://pdos.csail.mit.edu/6.824/notes/kv.go)

```
Go RPC is a simple form of "at-most-once"
  open TCP connection
  write request to TCP connection
  Go RPC never re-sends a request
    So server won't see duplicate requests
  Go RPC code returns an error if it doesn't get a reply
    perhaps after a timeout (from TCP)
    perhaps server didn't see request
    perhaps server processed request but server/net failed before reply came back
```