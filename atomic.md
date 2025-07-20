# ATOMIC

## 概念

原子操作指的是在多线程（或多核处理器）环境下，不可被中断地完成的操作。当一个线程执行一个原子操作时，其他线程无法观察到该操作执行到一半的状态，也不会在操作中途插入并修改操作所涉及的数据。原子操作是构建无锁数据结构、同步原语（如自旋锁）以及高效并发计数器等的基础。

传统锁 vs 原子操作：

锁 (Mutex)： 通过互斥访问来解决数据竞争。线程在访问共享数据前加锁，操作完成后解锁。这保证了操作的临界区的原子性，但引入了线程阻塞、上下文切换等开销。

原子操作： 直接在硬件指令级别保证特定操作的原子性（如读-修改-写）。通常比锁更轻量级（开销更小），避免了线程阻塞。但原子操作通常只适用于非常简单的操作（如加/减、位操作、交换、比较并交换），对于复杂的操作序列，仍然需要锁。

C11 标准正式将原子操作纳入语言核心，通过头文件 `<stdatomic.h>` 提供了一系列宏、类型和函数。

## API

C11标准通过`<stdatomic.h>`提供一套完整的原子操作API

### 原子类型声明

1. 内置原子类型：`atomic_x`，表示对应类型的原子类型，比如`atomic_int`表示原子整型
2. 泛型原子类型：`_Atomic(T)`，表示任意类型`T`的原子变量
3. 原子标志类型：`atomic_flag`

### 原子操作核心API

- `explicit`版本可以指定**内存序**

1. 加载操作（LOAD） —— 原子读

```C
C atomic_load(const volatile A* obj);
C atomic_load_explicit(const volatile A* obj, memory_order order);
// example
int value = atomic_load(&counter);  // 默认顺序一致性
int relaxed = atomic_load_explicit(&counter, memory_order_relaxed);
```

2. 存储操作（STORE） —— 原子写

```C
void atomic_store(volatile A* obj, C desired);
void atomic_store_explicit(volatile A* obj, C desired, memory_order order);
// example
atomic_store(&flag, 1);
atomic_store_explicit(&flag, 0, memory_order_release);
```

3. 交换操作（EXCHANGE）

原子替换为新值，并返回旧值

```C
C atomic_exchange(volatile A* obj, C desired);
C atomic_exchange_explicit(volatile A* obj, C desired, memory_order order);
// example
int old = atomic_exchange(&lock, 1);  // 获取锁
```

4. **原子比较交换（CAS）**

核心并发原语，用于实现无锁数据结构

```C
// 强版本
bool atomic_compare_exchange_strong(volatile A* obj, C* expected, C desired);
bool atomic_compare_exchange_strong_explicit(...);

// 弱版本
bool atomic_compare_exchange_weak(volatile A* obj, C* expected, C desired);
bool atomic_compare_exchange_weak_explicit(...);
```

其操作逻辑可以表述为

```C
// 注意这里仅为了方便理解，并不是原子操作
if (*obj == *expected) {
    *obj = desired;
    return true;
} else {
    *expected = *obj;
    return false;
}
```

|特性|strong|weak|
|--|--|--|
|虚假失败|不会发生|可能发生|
|性能|较低|较高|
|典型用法|独立操作|循环重试|
|硬件支持|所有平台|LL/SC 架构 (ARM, RISC-V)|

5. 算数运算

原子加减，返回旧值

```C
// 加法
C atomic_fetch_add(volatile A* obj, M operand);
C atomic_fetch_add_explicit(...);

// 减法
C atomic_fetch_sub(volatile A* obj, M operand);
C atomic_fetch_sub_explicit(...);
```

6. 位运算

原子位运算，返回旧值

```C
// 位与
C atomic_fetch_and(volatile A* obj, M operand);

// 位或
C atomic_fetch_or(volatile A* obj, M operand);

// 位异或
C atomic_fetch_xor(volatile A* obj, M operand);
```

## 内存序

在并发编程中，内存顺序（Memory Order）是控制多线程间内存操作可见性的关键机制。它定义了原子操作与非原子操作之间的顺序关系，决定了线程间如何观察内存修改。

在单线程中，代码的执行顺序（程序顺序）和实际执行顺序（由于编译器和处理器优化）可能不同，但保证最终结果与顺序执行一致。然而，在多线程环境下，由于每个线程的执行顺序可能被打乱，并且每个线程有自己的缓存，因此不同线程观察到的内存操作顺序可能不一致。

内存顺序模型就是用来规定一个线程内的内存操作如何影响其他线程的可见性。

C11以枚举的形式提供如下内存序

```C
typedef enum memory_order {
    memory_order_relaxed,  // 最弱约束
    memory_order_consume,  // 数据依赖排序
    memory_order_acquire,  // 获取语义
    memory_order_release,  // 释放语义
    memory_order_acq_rel,  // 获取+释放
    memory_order_seq_cst   // 顺序一致性 (默认)
} memory_order;
```

### memory_order_relaxed

最宽松的内存序

- 只保证原子操作的原子性和修改顺序的一致性（即同一个原子变量的修改在所有线程看来顺序相同）。
- 不提供任何跨线程的同步：即线程A对非原子变量的修改，在原子操作之后发生，但线程B看到原子操作完成并不保证能看到非原子变量的修改。
- 适用场景：简单的计数器，对执行顺序没有要求。

### memory_order_consume

- 针对“数据依赖”的操作：如果后续操作依赖于当前原子操作读取的值，则这些操作不会被重排序到当前原子操作之前。
- 比acquire更弱，因为它只限制与当前原子操作有数据依赖的操作。
- 注意：由于实现复杂且效果在多数平台与acquire相同，实际使用较少。

### memory_order_acquire

- 通常用于读操作（load）。保证后续的内存操作（包括非原子操作）不会被重排序到该原子操作之前。
- 同时，如果另一个线程使用release对同一个原子变量进行写操作，那么当前线程可以看到release操作之前的所有内存操作（即同步释放线程的修改）。

### memory_order_release

- 通常用于写操作（store）。保证前面的内存操作（包括非原子操作）不会被重排序到该原子操作之后。
- 当另一个线程使用acquire操作读取到该原子操作写入的值时，该线程可以看到release操作之前的所有内存操作。

### memory_order_acq_rel

- 结合了acquire和release，用于读-修改-写操作（如compare_exchange，fetch_add等）。
- 对于当前线程，该操作之前的内存操作不会被重排到该操作之后，之后的操作不会被重排到该操作之前。
- 同时，该操作具有同步效果：它会与使用acquire操作读取该原子操作的线程，以及使用release操作写入该原子操作的线程进行同步。

### memory_order_seq_cst

- 默认的内存顺序，最严格的顺序。
- 除了包含acq_rel的语义外，还保证所有线程看到的所有seq_cst操作的顺序都是一致的（全局顺序）。
- 该模型会阻止所有可能的重排序，因此性能开销最大，但最符合直觉。

### 同步关系

释放（release）和获取（acquire）操作可以建立同步关系：

线程A：写原子变量（release） -> 线程B：读同一个原子变量（acquire）

这样，线程A在release操作之前的所有写操作（包括非原子变量）在线程B执行acquire之后都是可见的。

