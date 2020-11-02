---
layout: post
title: "Few Thoughts on Concurrency in Singleton"
subtitle: "Double Checked Locking"
date: 2020-11-01
author: "Luyi"
header-img: "img/in-post/2020-11-01-concurrency-in-singleton/post-bg.jpg"
tags: 
    - Java
    - Design Pattern
---



Making singleton thread safe is a traditional topic in concurrency programming. There has a lot of implementations here but one of the most popular is [Double-Checked Locking](https://en.wikipedia.org/wiki/Double-checked_locking#Usage_in_Java). A project bug I found recently encouraged me to review this topic again.

### Problem Statement
One of my current working project has a process that needs to make HTTP call every 60s to retrieve some configurations. I have 4 machines runs in production, and I expect these HTTP traffic should be balanced in each server because I only need one process to do the job. However, these traffic is not distributed evenly, see `Figure_1`:
![Figure_1](/img/in-post/2020-11-01-concurrency-in-singleton/Figure_1.png)
<div style="text-align:center">Figure_1: Traffic are not balanced in each color</div>

### Bug Triage
I have a singleton class runs in each server, I used [DCL](https://en.wikipedia.org/wiki/Double-checked_locking#Usage_in_Java) to creat this instance:
```java
public static Instance getInstance() {
    if (instance == null) {
        synchronized (Instance.class) {
            if (instance == null) {
                instance = new Instance();
            }
        }
    }
    return instance;
}
```
Looks like correct, right? Yes, and I used [VisualVM](https://visualvm.github.io/) confirmed there has only one instance created in each server. Let's continue, inside the singleton class I also have a `init()` method to initialize the HTTP call:
```java
public static void init() {
    Timer timer = new Timer();
    timer.scheduleAtFixedRate(new TimerTask() {
        @Override
        public void run() {
            // http logic
        }
    } catch (Exception e) {
        log.error("Error loading Http config: ", e);
    }
}
```
No errors here, but I found in [VisualVM](https://visualvm.github.io/) that there has multiple timers created in each machine, so that's why caused the redundant HTTP traffic. Then, let's see where is this method called. In the `onConnect()` function, I have:
```java
public void onConnect() {
    instance().init();
}
```
Further more, this `onConnect()` method will be called everytime whenever a new connection comes in. So that's cause the problem. **Even though the singleton creation is synchronized, but the `init()` method is not which means any connection comes in will execute `init()` method.**

### My Solution and Results
To synchronize this `init()` method, I put it in `getInstance()`, so the HTTP timer will be created once:
```java
public static Instance getInstance() {
    if (instance == null) {
        synchronized (Instance.class) {
            if (instance == null) {
                instance = new Instance();
                init();
            }
        }
    }
    return instance;
}
```
See `Figure_2` results:
![Figure_2](/img/in-post/2020-11-01-concurrency-in-singleton/Figure_2.png)
<div style="text-align:center">Figure_2: Traffic are balanced in each color</div>

### Further Thoughts
I looked at the [DCL](https://en.wikipedia.org/wiki/Double-checked_locking#Usage_in_Java) implementation again and wonder why need double `null` check, here is the reason: think about a situation that there has 2 process arrive at the external `instance == null` same time, they all can pass the first check here. Then, only one process can enter the internal `instance == null` because of the `synchronized` keyword. Once the instance created, the first process will come out, and the second process will come in. If there has no second `instance == null` check, a second singleton instance will be created so that's will break the singleton rule.
```java
public static Instance getInstance() {
    if (instance == null) { // 2 process comes same time and they can all pass here.
        synchronized (Instance.class) { // only one process can enter because of the synchronized.
            if (instance == null) { // if no null check, the second process will create a new singleton when the first process comes out.
                instance = new Instance();
            }
        }
    }
    return instance;
}
```

