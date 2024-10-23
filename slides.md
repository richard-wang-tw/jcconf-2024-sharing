---
# You can also start simply with 'default'
theme: apple-basic
# random image from a curated Unsplash collection by Anthony
# like them? see https://unsplash.com/collections/94734566/slidev
background: https://cover.sli.dev
# some information about your slides (markdown enabled)
title: JCConf 2024
info: Java 21 and Beyond - State of Amber, Loom, and Valhalla
# apply unocss classes to the current slide
class: text-center
# https://sli.dev/features/drawing
drawings:
  persist: false
# slide transition: https://sli.dev/guide/animations.html#slide-transitions
transition: fade
# enable MDC Syntax: https://sli.dev/features/mdc
mdc: true
# take snapshot for each slide in the overview
overviewSnapshots: true
layout: intro-image
image: "https://cdn.jsdelivr.net/gh/slidevjs/slidev-covers@main/static/jtsW--Z6bFw.webp"
---

<!-- ### JCConf Taiwan 2024

# Java 21 and Beyond

### State of Amber, Loom, and Valhalla -->

<div class="absolute top-10">
  <span class="font-700">
    <Date/>
  </span>
</div>

<div class="absolute bottom-10 text-align-start">
  <h1>Java 21 and Beyond</h1>
  <p>State of Amber, Loom, and Valhalla</p>
</div>
<!-- 
<div class="abs-br m-6 flex gap-2">
  <button @click="$slidev.nav.openInEditor()" title="Open in Editor" class="text-xl slidev-icon-btn opacity-50 !border-none !hover:text-white">
    <carbon:edit />
  </button>
  <a href="https://github.com/slidevjs/slidev" target="_blank" alt="GitHub" title="Open in GitHub"
    class="text-xl slidev-icon-btn opacity-50 !border-none !hover:text-white">
    <carbon-logo-github />
  </a>
</div> -->

<!--
The last comment block of each slide will be treated as slide notes. It will be visible and editable in Presenter Mode along with the slide. [Read more in the docs](https://sli.dev/guide/syntax.html#notes)
-->

---

# SDK 8 → 21

<div class="flex flex-col flex-items-center flex-justify-center">
  <div class="w-[75%] flex flex-col flex-items-start flex-justify-center">
    <img src="https://i.imgur.com/ZIytrI4.png" />
    17 → 21: 2 years (Sept. 2023)<br>
    21 → 25: ? years
  </div>
</div>

---

# SDK 8 → 21 → 25

<div class="flex flex-col flex-items-center flex-justify-center">
  <div class="w-[75%] flex flex-col flex-items-start flex-justify-center">
    <img src="https://i.imgur.com/ZIytrI4.png" />
    17 → 21: 2 years (Sept. 2023)<br>
    21 → 25: 2 years (Sept. 2025)
  </div>
</div>

---
layout: center
---
# Every non-LTS SDK are production ready
### Upgrading early not only allows us to enjoy the enhanced development experience
### but also helps avoid being forced to rush into upgrades in the future
---



# <span class="color-yellow">Amber</span>, Loom, and Valhalla

  ```java
  // var (10) text block (13)
  var json = """ 
          {
              "name": "Alice",
              "age": 30
          }
          """;

  // record (14)
  public record Person(String name, int age) {}
  Object object = new Person("Alice", 30);

  // elementary pattern matching (16)
  if (object instanceof Person) {
      System.out.println("It's a person !");
  }
  if (object instanceof Integer) {
      System.out.println("It's a integer !");
  }
  ```
---

# <span class="color-yellow">Amber</span>, Loom, and Valhalla

  ```java
  // JEP 455: Primitive Types in Patterns, instanceof, and switch
  // Status:	    Closed / Delivered
  // Release:	    23
  // Summary:     .... This is a preview language feature.
  // Goals:       ... Provide easy-to-use constructs that eliminate the risk of losing information due to unsafe casts ...


  double a = 1.3d;
  double b = 1.0d;

  // checks if lossless conversion to int is possible
  a instanceof int // false
  b instanceof int // true

  String result = switch (a) {
          case int i -> "case 1 " + i; // safe parse, X
          case float i when i >= 100 -> "case 2"; // safe parse + condition, X
          case float i -> "case 4"; // equal to case i is float and i < 100, V
      };

  // is there any other issue with the switch code?
  ```
---

# <span class="color-yellow">Amber</span>, Loom, and Valhalla


  ```java
  public sealed interface JsonValue {
    record JsonString(String s) implements JsonValue { }
    record JsonNumber(double d) implements JsonValue { } // number is double
    record JsonObject(Map<String, JsonValue> map) implements JsonValue { }
  }

  var jsonObject = new JsonObject(Map.of("name", new JsonString("John"), "age", new JsonNumber(30)));

  // as is
  if (jsonObject instanceof JsonObject(var map) && map.get("age") instanceof JsonNumber(double i)) {
      int age = (int) i;  // unavoidable (and potentially lossy!) cast
  }
  ```

  ```java
  // SDK 23
  if (jsonObject instanceof JsonObject(var map) && map.get("age") instanceof JsonNumber(int i)) {
      int age = i; // no more cast
  }
  ```

---

# Amber, <span class="color-yellow">Loom</span>, and Valhalla
  ```java
  // jdk 1.5
  ExecutorService executor = Executors.newFixedThreadPool(3);
  for (int i = 0; i < 10; i++) {
      final int taskId = i;
      executor.submit(() -> {
          try {
              Thread.sleep(1000);
              System.out.println("Task " + taskId + " completed by " + Thread.currentThread().getName());
          } catch (InterruptedException e) {
              e.printStackTrace();
          }
      });
  }
  // threads are too "fat" for most requests, it takes 4 seconds to complete the 10 tasks
  // threads may cause memory leak
  executor.shutdown();
  ```

---

# Amber, <span class="color-yellow">Loom</span>, and Valhalla
  ```java
  // JEP 444: Virtual Threads
  // Release:	    21
  // Summary:     ... dramatically reduce the effort of writing, maintaining, and observing high-throughput concurrent applications
  // Goals:       - ... simple thread-per-request style ... scale with near-optimal hardware utilization ...
  //              - Enable existing code ... to adopt virtual threads with minimal change.
  var futures = new ArrayList<Future>();
  try (var executor = Executors.newVirtualThreadPerTaskExecutor()) {
      for (int i = 0; i < 10; i++) {
          var taskId = i;
          var future = executor.submit(() -> {
              try {
                  Thread.sleep(Duration.ofSeconds(1));
                  System.out.println("Task " + taskId + " completed");
              } catch (InterruptedException e) {
                  throw new RuntimeException(e);
              }
          });
          futures.add(future);
      }
  }
  for (Future future : futures) {
      future.get();
  } // 10 tasks ~= 1 second, 1000000 tasks ~= 2.6 seconds
  ```
---

# Amber, <span class="color-yellow">Loom</span>, and Valhalla

popluar languages structured concurrency support

- F# 2.0（2007）
- Python 3.5（2015）
- JypeScript 1.7（2015）
- C++20（2020）
- <span class="color-yellow">Java（2023）</span>


---

# How to try latest JDK ?
<div class="flex flex-col flex-items-center flex-justify-center">
  <div class="w-[60%] flex flex-col flex-items-start flex-justify-center">
    <img src="https://i.imgur.com/Y7bubls.png" />
     <a href="https://jdk.java.net/24" >https://jdk.java.net/24</a>
  </div>
</div>

---

# How to try latest JDK ?
<div class="flex flex-col flex-items-center flex-justify-center">
  <div class="w-[60%] flex flex-col flex-items-start flex-justify-center">
    <img src="https://i.imgur.com/GMZDG2b.png" /><br>
    <img src="https://i.imgur.com/i9i1jHM.png" />
  </div>
</div>


---

# Amber, Loom, and <span class="color-yellow">Valhalla</span>

- Java **generic API** relies on a **type erasure** mechanism
  ```
  compile time → run time  
  Box<Integer> → Box
  ```
  
- This results in java lose the following 3 features
  ```java
  // 1. reflection
  a instanceof Box<Integer>

  // 2. specialization
  Box<Integer> // ref in stack, data in heap
  Box<int> // no ref, only data in stack, more predictable life cycle, more efficient garbage collection

  // 3. runtime type checking
  Box<String> bs = new Box<>("hi!");   // safe
  Box<?> bq = bs;                      // safe, via subtyping
  Box<Integer> bi = (Box<Integer>) bq; // unchecked cast -- warning issued
  Integer i = bi.get();                // heap pollution
  ```
---
layout: quote
---

# "Codes like a class, works like an int."

---

# Amber, Loom, and <span class="color-yellow">Valhalla</span>

```java
// JEP 401: Value Classes and Objects (Preview)
// Status	Draft
// Effort	XL

// 1. immutibility, every filed should be final
value class Color {
  private final int red; 
  ...
  public Color(int red, ...){};
}
// 2. efficiency: JVM covert value class to memory #646464
// 3. equality
Color a = new Color(...);
Color b = new Color(...);
if( a == b )
// 4. a nested value class is still value class
value class Colors {
  private final Color a
}
// 5. a value class can be null
Color a = null
// 6. null-restricted type: run time type checking (not only value class)
Color! orange = new Color(...);
```

---

# Amber, Loom, and <span class="color-yellow">Valhalla</span>

```java
// JEP 402: Enhanced Primitive Boxing (Preview)
// JEP draft: Null-Restricted Value Class Types
// Status	Draft
// Effort	XL

// 1. int codes like Integer
int a = 0;
a.MAX_VALUE
// 2. Integer works like int
Integer b = 0; // use literal 0 instead of Integer object ref in stack
// 3. Null-restricted type
Person? b = null; // run time error
```

---
layout: fact
---

# `\{*v*}/`
Happy Coding ! Thank you !
<!-- <div class="flex flex-col gap-4 flex-items-center">
  <p class="font-size-[50px]"></p>
  <p class="font-size-[80px]">\{*v*}/</p>
</div> -->

