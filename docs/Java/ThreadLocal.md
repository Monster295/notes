# ThreadLocal

`ThreadLocal` 是 `Java` 提供的一种线程隔离机制，它为每个线程都提供了一个独立的变量副本。这意味着，当多个线程同时访问同一个 `ThreadLocal` 变量时，每个线程都会拥有自己独立的一个副本，对该副本的修改不会影响到其他线程所拥有的副本。从线程的角度看，每个线程都拥有该变量的独立“本地”副本，因此得名 ThreadLocal。

## ThreadLocal 的作用和应用场景：

`ThreadLocal` 主要用于解决多线程环境下的**资源共享**和**并发访问**的问题，可以帮助实现线程安全，避免由于多个线程共享同一个变量而导致的竞争条件和数据不一致性。

常见的应用场景包括：

- **数据库连接管理**：在 Web 应用中，每个请求通常由一个独立的线程处理。可以使用 ThreadLocal 来管理每个请求的数据库连接，确保每个线程拥有自己的连接，避免连接的混用和并发问题。
- **事务管理**: 在事务处理中，可以使用 ThreadLocal 来管理每个线程的事务状态（例如，是否已开启事务、事务是否已提交或回滚）。
- **Session 管理**: 在某些框架中，可以使用 ThreadLocal 来存储当前线程的会话信息，方便在同一个请求处理链路中访问。
- **简单工厂模式的线程安全**: 如果一个工厂类创建的对象是线程不安全的，可以使用 ThreadLocal 为每个线程创建一个独立的对象实例。
- **存储线程上下文信息**: 例如，存储用户的 ID、请求的 Trace ID 等信息，方便在整个线程执行过程中访问。

## ThreadLocal 的基本使用：

`ThreadLocal` 提供了一些核心方法来操作线程本地变量：

- `initialValue()`: 返回此线程局部变量的初始值。这个方法在第一次使用 `get()` 方法访问变量时会被调用（除非之前调用了 `set()` 方法）。子类可以重写此方法来提供自定义的初始值。默认实现返回 `null`。
- `get()`：返回当前线程所持有的该线程局部变量的值。如果这是当前线程第一次调用 `get()` 方法，并且之前没有调用过 `set()` 方法，那么会先调用 `initialValue()` 方法来初始化值。
- `set(T value)`：设置当前线程所持有的该线程局部变量的值。
- `remove()`：移除当前线程所持有的该线程局部变量的值。如果不再需要使用 `ThreadLocal` 变量，应该及时调用 `remove()` 方法，以防止内存泄漏。

## ThreadLocal 的实现原理：

`ThreadLocal` 的实现核心在于 `Thread` 类内部维护了一个 `ThreadLocalMap`。`ThreadLocalMap`是一个自定义的哈希表，他的 Key 是`ThreadLocal`对象本身（的弱引用），Value 是该 `ThreadLocal` 在当前线程中存储的实际值。

当一个线程调用 `ThreadLocal` 的`get()` 方法时，`ThreadLocal` 会获取当前线程，然后以当前 `ThreadLocal` 对象未 Key，从当前线程的 `ThrealLocalMap` 中查找对应的 Value。

当一个线程调用 `ThreadLocal` 的`set(value)` 方法时，`ThreadLocal` 同样会获取当前线程，然后以当前 `ThreadLocal` 对象为 Key，将 `value` 存储到当前相乘的 `ThreadLocalMap` 中。

`remove()` 方法则是从当前线程的 `ThreadLocalMap` 中移除以当前 `ThreadLocal` 对象为 Key 的 Entry。

### 存储线程上下文信息的demo

```java
package src;

import java.util.HashMap;
import java.util.Map;

// MapContext 类用于存储和管理线程隔离的上下文数据
public class MapContext {
    // ThreadLocal 用来存储每个线程的上下文，每个线程都有自己的 Map 实例
    private static final ThreadLocal<Map<String, Object>> context = ThreadLocal.withInitial(HashMap::new);

    // 从context中获取当前线程的 Map 示例
    public static Map<String, Object> get() {
        return context.get();
    }

    // 先获取当前线程的 Map 实例，然后将 key-value 存入其中
    public static void set(String key, Object value) {
        context.get().put(key, value);
    }

    // 获取当前线程的 Map 实例，然后根据 key 获取对应的 value
    public static Object get(String key) {
        return context.get().get(key);
    }

    // 根据 key 删除当前线程的 Map 实例中的对应value
    public static void remove(String key) {
        context.get().remove(key);
    }

    // 清空当前线程 ThreadLoacl 变量中的所有数据
    public static void clear() {
        context.remove();
    }

    public static void main(String[] args) throws Exception {
        Runnable task1 = () -> {
            MapContext.set("userId", 123L);
            MapContext.set("userName", "user1");
            System.out.println("Task 1 - Context: " + MapContext.get());
            MapContext.clear();
        };

        Runnable task2 = () -> {
            MapContext.set("orderId", "ORD-456");
            System.out.println("Task 2 - Context: " + MapContext.get());
            System.out.println("Task 2 - UserId: " + MapContext.get("userId")); // 输出 null，因为线程隔离
            MapContext.clear();
        };
        Thread thread1 = new Thread(task1);
        Thread thread2 = new Thread(task2);

        thread1.start();
        thread2.start();

        thread1.join();
        thread2.join();

        System.out.println("Main thread - Context: " + MapContext.get()); // 输出 {}，因为主线程没有设置
    }

}

```