---
title: "Thread Simple"
date: 2022-06-08T00:02:56+08:00
draft: true
type: post
tags: ['Java']
---

## 线程的状态

- 新建状态
  - new 之后没有 start
- runnable 
  - start 之后，CPU调度
  - Ready 就绪状态，等待 CPU 执行
    - 同步代码块获取到锁就是就绪状态
  - Running 运行状态，CPU 执行
    - o.notifyAll()
    - LockSupport.unpark()
  - Timewaiting 等待，按照时间等待
    - Thread.sleeep(time)
    - o.wait(time)
    - t.join(time)
    - LockSupport.parkNanos()
    - LockSupport.parkUntil()
  - Waiting 等待
    - o.wait()
    - t.join()
    - LockSupport.park()
  - Blocked 阻塞
    - 同步代码块没有获取到锁就阻塞
  - Yield()  Running -> ready
- Terminated 
  - 执行结束
- 
