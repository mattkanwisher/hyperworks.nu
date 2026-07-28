---
title: "Handling Blocking IO in Go"
date: 2012-07-19
description: "How to deal with blocking IO in Go — spawning processes, reading stdout without freezing your server, channels, and killing hung tails safely."
tags: [go, archive]
image: /images/og/blog-go-io.jpg
---

*Originally published on kanwisher.com, July 19, 2012. Restored from the [Wayback Machine](https://web.archive.org/web/20120720065758/http://www.kanwisher.com/2012/07/19/handling-blocking-io-in-go/).*

One of the most frustrating things when you start with Go is that fact that **IO is blocking**. So for example today I needed to spawn a process and read the stdout — easy right?

```go
cmd := exec.Command("tail", "-f", "/tmp/matt.txt")
stdout, err := cmd.StdoutPipe()
if err != nil {
    log.Fatal(err)
}
if err := cmd.Start(); err != nil {
    log.Fatal(err)
}

// Ok here is the read
reader := bufio.NewReader(stdout)
s, err := reader.ReadString('\n')
```

Ok that's great, but now the whole process waits while you read the IO, which sucks if you have a server with lots of clients connecting.

So lets improve it a bit and make it a **goroutine** so it doesn't block at all.

```go
func readLogData(filename string, log_id int, logOutputChan chan<- *LogTuple, deathChan chan<- *string) {
    cmd := exec.Command("tail", "-f", filename)
    stdout, err := cmd.StdoutPipe()
    if err != nil {
        log.Fatal(err)
    }
    if err := cmd.Start(); err != nil {
        log.Fatal(err)
    }

    err = nil
    reader := bufio.NewReader(stdout)
    sbuffer := ""
    lines := 0
    for ; err == nil; {
        s, err := reader.ReadString('\n')
        sbuffer += s
        lines += 1
        if lines > 5 { // || time > 1 min
            logOutputChan <- &LogTuple{log_id, sbuffer}
            sbuffer = ""
            lines = 0
        }
        if err != nil {
            deathChan <- &filename
            return
        }
    }

    // SetFinalizer
}

go readLogData(alog.Log.Path, alog.Log.Id, logOutputChan, gorDeathChan)
```

A little bit more complicated, but lets explain it. We broke this code into a separate function that takes in **2 channels**. Channels in Go let you send data between goroutines. We have one channel to get data from the goroutine, and another that tells the goroutine to die. Why we need this? Cause there is no way to kill a goroutine safely.

First problem you might notice is that if the underlying IO blocks forever and never returns we can never end this goroutine. When would this happen? A good example is `tail -f` or a process that has gone haywire and is not responding. So how do we fix this?

```go
package main

import (
    "fmt"
    "log"
    "os/exec"
    "time"
)

func main() {
    cmd := exec.Command("tail", "-f", "/tmp/foo.txt")
    stdout, err := cmd.StdoutPipe()
    if err != nil {
        log.Fatal(err)
    }
    if err := cmd.Start(); err != nil {
        log.Fatal(err)
    }

    ch := make(chan string)
    quit := make(chan bool)
    go func() {
        buf := make([]byte, 1024)
        for {
            n, err := stdout.Read(buf)
            if n != 0 {
                ch <- string(buf[:n])
            }
            if err != nil {
                break
            }
        }
        fmt.Println("Goroutine finished")
        close(ch)
    }()

    time.AfterFunc(time.Second, func() { quit <- true })

loop:
    for {
        select {
        case s, ok := <-ch:
            if !ok {
                break loop
            }
            fmt.Print(s)
        case <-quit:
            cmd.Process.Kill()
        }
    }
}
```

So basically we **kill the underlying process** so the IO sends an EOF. This is a simplified example. In my server I do keep track of when I want processes to die and spawn and its not on a simple timer.

Special thanks to mrlauer on the [Golang-nuts](https://groups.google.com/forum/?fromgroups#!forum/golang-nuts) message board.

---

*Originally written while building Errplane — a Go service for Rails exception tracking, log aggregation, and alerting / monitoring (site no longer online).*
