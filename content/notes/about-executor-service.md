---
title: "Java ExecutorService에 대하여"
author: ["조민준"]
date: 2023-07-14T10:18:29+09:00
draft: false
categories: ["Dev"]
tags: ["Java"]
ShowToc: true
---

## ExecutorService
- ExecutorService를 사용하면 간단하게 스레드풀을 생성해서 병렬처리를 할 수 있습니다.
- Executors의 스태틱 메서드를 통해 간단하게 ExecutorService를 사용할 수 있습니다.
  - 생성자를 이용해 커스텀하게 생성할 수도 있습니다.
- 이상적인 풀 사이즈
    | Task Type | Ideal pool size | Considerations                                                            |
    | --------- | --------------- | ------------------------------------------------------------------------- |
    | CPU bound | CPU Core count  | 얼마나 많은 작업이 같은 CPU에서 동작하는지 (Context switching 비용)       |
    | I/O bound | High            | 각 작업의 평균 대기 시간, 너무 큰 스레드 풀은 메모리 사용량을 고려해야함. |

```java
int coreCount = Runtime.getRuntime().availableProcessors();
// int coreCount = taskExecutionProperties.getPool().getCoreSize();

ExecutorService service = Executors.newFixedThreadPool(coreCount);
```

## ThreadPoolExecutor 생성자 파라미터

| Parameter            | Type                     | Meaning                                                  |
| -------------------- | ------------------------ | -------------------------------------------------------- |
| corePoolSize         | Integer                  | pool 최소 크기                                           |
| maxPoolSize          | Integer                  | pool 최대 크기                                           |
| keepAliveTime + unit | Long                     | Thread를 idle 상태로 유지할 시간 (시간 초과 후에는 kill) |
| workQueue            | BlockingQueue            | 작업을 저장해두는 큐                              |
| threadFactory        | ThreadFactory            | 새로운 Thread를 생성하는 Factory                         |
| handler              | RejectedExecutionHandler | 작업 실행이 거부되었을 때 사용할 callback                |

- core pool thread는 `allowCoreThreadTimeOut(boolean value)`를 `true`로 설정하지 않으면 Kill되지 않습니다.

## Executors 스태틱 메서드로 제공하는 풀의 종류
- FixedThreadPool
    - 고정된 수의 Thread를 가집니다.
    - Blocking queue에 작업을 쌓아두고 Thread가 작업을 하나씩 수행합니다.
- CachedThreadPool
    - 고정된 수의 Thread가 없습니다.
    - 하나의 작업만 저장할 수 있는 Syncronus queue에 작업을 저장해두고 사용 가능한 Thread에 할당합니다.
    - 사용 가능한 Thread가 없으면 새로운 Thread를 생성합니다.
- ScheduledThreadPool
    - 고정된 수의 Thread를 가집니다.
    - 작업을 일정 시간 지연 후에 수행하거나 일정 시간 간격으로 실행시킵니다.
    - `schedule(Runnable runnable, Long delay, TimeUnit timeunit)`
        - 일정 시간 뒤에 작업을 한번 실행시킵니다.
    - `scheduleAtFixedRate(Runnable runnable, Long delay, Long period, TimeUnit timeunit)`
        - 작업을 일정 시간 간격으로 반복적으로 실행시킵니다.
    - `scheduleAtFixedDelay(Runnable runnable, Long initDelay, Long period, TimeUnit timeunit)`
        - 이전 작업 완료 시 작업을 일정 시간 간격으로 반복적으로 실행시킵니다.
- SingleThreadExecutor
    - 1개의 Thread를 가집니다.
    - Blocking queue에 작업을 쌓아두고 Thread가 작업을 하나씩 수행합니다.
    - Thread가 kill되면 다시 Thread생성됩니다.

### 풀 종류 별 Queue

| Pool                 | Queue Type          | Why?                                                                                      |
| -------------------- | ------------------- | ----------------------------------------------------------------------------------------- |
| FixedThreadPool      | LinkedBlockingQueue | Unbounded queue에 모든 작업을 저장해두고, 한정된 Thread에서 작업을 순서대로 처리한다.     |
| SingleThreadExecutor | LinkedBlockingQueue | Unbounded queue에 모든 작업을 저장해두고, 한정된 Thread에서 작업을 순서대로 처리한다.     |
| CachedThreadPool     | SynchronousQueue    | Thread가 Unbounded이므로, 작업을 queue에 저장해두지 않아도 된다. 단 하나의 slot만 가진다. |
| ScheduledThreadPool  | DelayWorkQueue      | 시간 딜레이를 가지는 특별한 queue.                                                        |


## 작업이 거부되었을 때 정책
| Policy              | Meaning                                                                                                                             |
| ------------------- | ----------------------------------------------------------------------------------------------------------------------------------- |
| AbortPolicy         | 제출된 새로운 작업이 RejectedExecutionException을 발생시킨다. 정책을 등록하지 않으면 기본으로 사용된다.                             |
| DiscardPolicy       | 새로운 작업이 삭제된다.                                                                                                             |
| DiscardOldestPolicy | 가장 오래된 작업이 삭제되고, 새로운 작업이 queue에 저장된다.                                                                        |
| CallerRunsPolicy    | 작업을 요청한 스레드에서 작업이 실행된다. 작업을 요청한 스레드에서 제출한 작업이 실행되는 동안 새로운 작업을 생성하지 못할 수 있다. |

- 정책은 아래와 같은 생성자 방식으로 사용할 수 있습니다.
  ```java
  ExecutorService executorService = new ThreadPoolExecutor(
      10,                             // corePoolSize
      100,                            // maxPoolSize
      120, TimeUnit.SECONDS,          // keepAliveTime
      new ArrayBlockingQueue<>(300),  // Queue
      new DiscardPolicy()             // Policy
  );
  ```

### RejectedExecutionException 핸들링
<방법 1>
```java
ExecutorService executorService = new ThreadPoolExecutor(
    10,                             // corePoolSize
    100,                            // maxPoolSize
    120, TimeUnit.SECONDS,          // keepAliveTime
    new ArrayBlockingQueue<>(300)   // Queue
);

try {
    executorService.execute(new Task());
} catch (RejectedExecutionException e) {
    log.error("task rejected", e)
}
```
<방법2>
```java
ExecutorService executorService = new ThreadPoolExecutor(
    10,                             // corePoolSize
    100,                            // maxPoolSize
    120, TimeUnit.SECONDS,          // keepAliveTime
    new ArrayBlockingQueue<>(300),  // Queue
    new CustomRejectionHandler()    // 커스텀 핸들러를 Policy로 등록한다.
);

executorService.execute(new Task());
```
```java
public class CustomRejectionHandler implements RejectedExecutionHandler {

    @Overrid
    public void rejectedExecution(Runnable runnble, ThreadPoolExecutor executor) {
        // logging, operations to perform on rejection
    }
}
```

## LifeCycle 메서드

```java
ExecutorService executorService = new ThreadPoolExecutor(
    10,                             // corePoolSize
    100,                            // maxPoolSize
    120, TimeUnit.SECONDS,          // keepAliveTime
    new ArrayBlockingQueue<>(300)   // Queue
);

executorService.execute(new Task());

// init shutdown
executorService.shutdown();

// throw RejectionExecutionException
executorService.execute(new Task());

// return true, since shutdown has begun
executorService.isShutdown();

// return true if all tasks are completed
executorService.isTerminated();

// block until all tasks are completed or timeouted
executorService.awaitTermination(10, TimeUnit.SECOND);

// init shutdown and return all queued tasks
List<Runnable> runnables = executorService.shutdownNow();
```
