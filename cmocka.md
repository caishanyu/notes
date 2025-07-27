# CMOCKA

CMocka 是一款轻量级、功能丰富的 C 语言单元测试框架，支持模拟函数(mock)、对象、丰富的断言和内存检测，特别适合嵌入式与跨平台项目

[官方文档](https://cmocka.org/)

## 一、安装CMocka

Debian/Ubuntu通过apt安装

```bash
sudo apt-get update
sudo apt-get install libcmocka-dev

sudo apt install pkgconf
pkg-config --modversion cmocka  # 输出版本号即成功
```

添加到标准路径

```bash
export LD_LIBRARY_PATH=/path/to/cmocka/lib:$LD_LIBRARY_PATH
```

编译时需要链接cmocka库，添加链接标志：`-lcmocka`，注意链接标志放在最后，防止产生顺序依赖问题

## 二、编写测试用例

最基本的代码示例如下，关键点

- 测试函数需匹配签名 `void (*)(void **state)`
- `cmocka_unit_test` 宏将函数包装为测试用例
- `cmocka_run_group_tests` 执行所有用例并输出报告

```C
// 包含以下头文件
#include <stdarg.h>
#include <stddef.h>
#include <setjmp.h>
#include <cmocka.h>

static inline int add(int a, int b)    return (a+b);

// 测试函数签名：void test_func(void **state)
static void test_add(void **state) {
    (void)state; // 避免未使用警告
    assert_int_equal(add(2, 3), 5);  // 验证整数相等
}

int main(void) {
    const struct CMUnitTest tests[] = {
        cmocka_unit_test(test_add),  // 注册测试用例
        // 可添加更多用例
    };
    return cmocka_run_group_tests(tests, NULL, NULL); // 运行测试组
}
```

### 三、使用断言

CMocka 提供多种断言宏验证逻辑：

|断言宏|用途|示例|
|--|--|--|
|`assert_true(cond)`|验证条件为真|`assert_true(ptr != NULL)`|
|`assert_int_equal(a,b)`|验证整数相等|`assert_int_equal(sum, 5)`|
|`assert_string_equal(a,b)`|验证字符串相等|`assert_string_equal(str, "OK")`|
|`assert_memory_equal(a,b,size)`|验证内存块一致|`assert_memory_equal(buf1, buf2, 10)`|
|`assert_null(ptr)`|验证指针为 `NULL`|`assert_null(failed_ptr)`|

使用断言可以对运行结果进行判断，断言失败，则会立即终止当前测试，并标记为测试失败

### 四、内存泄漏检测

可以通过重定义内存函数以检测泄漏/越界：

```C
#ifdef UNIT_TESTING
// 重定向标准内存函数
#define malloc(size) _test_malloc(size, __FILE__, __LINE__)
#define free(ptr)    _test_free(ptr, __FILE__, __LINE__)
#endif

static void test_leak(void **state) {
    int *ptr = malloc(10);  // 未释放！
}  // 测试结束时报错：检测到内存泄漏
```

## gcov

gcov可以搭配cmocka，生成测试代码覆盖率

1. 需要开启GCC编译选项`-fprofile-arcs -ftest-coverage`，链接C库`-lgcov`
2. 确保安装gcov（通常随gcc一起安装）和lcov（`sudo apt-get install lcov`）
3. 运行可执行文件，生成.gcov和.gcda文件，用于后续分析
4. 具体命令可见如下脚本

```bash
#!/bin/bash

echo "[enter dlist path]"
cd obj/ds/dlist

echo "[generate soft link to src files]"
ln -s ../../../ds/dlist/dlist.c .

echo "[generate dlist gcov]"
gcov dlist.c

# 收集数据
lcov --capture --directory . --output-file coverage.info

# 生成HTML
genhtml coverage.info  --output-directory coverage_report

echo "[return top path]"
cd -
```
