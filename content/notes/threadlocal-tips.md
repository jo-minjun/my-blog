---
title: "사소한 ThreadLocal Tips"
author: ["조민준"]
date: 2023-07-13T18:40:44+09:00
draft: false
categories: ["Dev"]
tags: ["Java"]
ShowToc: true
---

1. 가능하다면 로컬 변수를 사용합니다.
2. 프레임워크에 위임합니다.
    - e.g. `RequestContextHolder`
3. `ConcurrentHashMap` 같은 요소를 `ThreadLocal` 변수로 변경할 수 있는지 검토합니다.

```java
public class UserContextHolder {

  public static ThreadLocal<User> holder = new ThreadLocal();
}
```

```java
class HoldingService {

  public void holdUser() {
    // Set user for this thread
    User user = getUser();
    UserContextHolder.holder.set(user);
  }
}
```

```java
class SomeService {

  public void getUser() {
    // Get user for this thread
    User user = UserContextHolder.holder.get();
    // Remove user; user no longer required
    UserContextHolder.holder.remove();
  }
}
```
