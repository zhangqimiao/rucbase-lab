<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->

- [任务操作指南](#任务操作指南)
  - [补全代码](#补全代码)
  - [自我测试](#自我测试)
    - [扩展](#扩展)


<!-- END doctoc generated TOC please keep comment here to allow auto update -->

# 任务操作指南

> 本文档将以任务一（存储管理）为例示范你应该如何完成一个任务，并进行自我测试

根据[Rucbase-Lab1存储管理](Rucbase-Lab1[实验文档].md)查看相应任务，进入`rucbase`项目对应的代码目录，你将看到实验文档中说明的需要你本人完成的项目源码`.cpp/.h`文件，除此之外，你还可能看到类似`*_test.cpp/*_test.sh`的文件，其中`*_test.cpp`文件是`GoogleTest`测试源码文件，而`.sh`文件则是在其他实验中使用的非`GoogleTest`测试脚本，在本实验中并不涉及。

在阅读本指南前，请确保自己已经按照[Rucbase环境配置文档](Rucbase环境配置文档.md)配置好了环境，并根据[Rucbase使用文档](Rucbase使用文档.md)创建好了`build`目录，测试了`cmake`工具

## 补全代码

我们以`任务1.1.2 缓冲池替换策略`为例，进入`replacer`目录，其中源码文件如下：

```bash
Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
-a----         2022/9/18     10:07           1352 clock_replacer.cpp
-a----         2022/9/18     10:07           1252 clock_replacer.h
-a----         2022/9/18     10:07           2039 clock_replacer_test.cpp
-a----         2022/9/18     10:07            434 CMakeLists.txt
-a----         2022/9/18     10:07           1590 lru_replacer.cpp
-a----         2022/9/18     10:07           2351 lru_replacer.h
-a----         2022/9/18     10:07           9615 lru_replacer_test.cpp
-a----         2022/9/18     10:07           1021 replacer.h
```

为完成本任务，你需要完成[Rucbase-Lab1实验文档](Rucbase-Lab1[实验文档].md)中`任务1.1.2`中指出的`Replacer`类相应接口，其中必做任务是LRU策略，你需要打开`lru_replacer.cpp`，在源码文件中，对于每个你需要实现的接口方法进行了功能描述和实现提示，示例如下：

```cpp
/**
 * @brief 使用LRU策略删除一个victim frame，这个函数能得到frame_id
 * @param[out] frame_id id of frame that was removed, nullptr if no victim was found
 * @return true if a victim frame was found, false otherwise
 */
bool LRUReplacer::Victim(frame_id_t *frame_id) {
    // C++17 std::scoped_lock
    // 它能够避免死锁发生，其构造函数能够自动进行上锁操作，析构函数会对互斥量进行解锁操作，保证线程安全。
    std::scoped_lock lock{latch_};

    // Todo:
    //  利用lru_replacer中的LRUlist_,LRUHash_实现LRU策略
    //  选择合适的frame指定为淘汰页面,赋值给*frame_id

    return true;
}
```

请注意，返回值代码语句只是为了保证初始代码不会有语法检查错误，事实上整个接口内部的实现逻辑你可以完全修改。

如果你需要额外的数据结构，你可以在`.h`文件中对类进行添加，在`LRUReplacer`中，进入`lru_replacer.h`，你可以看到已经给出的部分数据结构。

```cpp
std::mutex latch_;               // 互斥锁
std::list<frame_id_t> LRUlist_;  // 按加入的时间顺序存放unpinned pages的frame id，首部表示最近被访问
std::unordered_map<frame_id_t, std::list<frame_id_t>::iterator> LRUhash_;  // frame_id_t -> unpinned pages的frame id
size_t max_size_;  // 最大容量（与缓冲池的容量相同）
```

你也可以自行修改或添加，只要最后正确实现功能即可。**但是，强烈建议你不要擅自修改每个接口的声明**

当你完成LRU策略后，你也可以自行考虑是否实现CLOCK策略，整个实现步骤与LRU任务类似。



## 自我测试

当你完成相应任务后，你可以对自己的实验进行功能测试。请首先回到整个项目的根目录

```bash
cd rucbase-lab
cd build
make [target_test]
./bin/[target_test]
```

如你想测试`lru_replacer_test.cpp`提供的测试，你的编译执行命令就是

```bash
make lru_replacer_test
./bin/lru_replacer_test
```

如果测试通过，你得到的输出应该类似于

```
aaron@DESKTOP-U1TJ26P:~/rucdeke/rucbase-lab/build$ ./bin/lru_replacer_test 
Running main() from /home/aaron/rucdeke/rucbase-lab/deps/googletest/googletest/src/gtest_main.cc
[==========] Running 6 tests from 1 test suite.
[----------] Global test environment set-up.
[----------] 6 tests from LRUReplacerTest
[ RUN      ] LRUReplacerTest.SampleTest
[       OK ] LRUReplacerTest.SampleTest (0 ms)
[ RUN      ] LRUReplacerTest.Victim
[       OK ] LRUReplacerTest.Victim (1 ms)
[ RUN      ] LRUReplacerTest.Pin
[       OK ] LRUReplacerTest.Pin (1 ms)
[ RUN      ] LRUReplacerTest.Size
[       OK ] LRUReplacerTest.Size (8 ms)
[ RUN      ] LRUReplacerTest.ConcurrencyTest
[       OK ] LRUReplacerTest.ConcurrencyTest (122 ms)
[ RUN      ] LRUReplacerTest.IntegratedTest
[       OK ] LRUReplacerTest.IntegratedTest (12 ms)
[----------] 6 tests from LRUReplacerTest (146 ms total)

[----------] Global test environment tear-down
[==========] 6 tests from 1 test suite ran. (146 ms total)
[  PASSED  ] 6 tests.
```

测试输出会给出各个测试点是否通过的信息以及每个测试点花费的时间。

如果没有通过，会在对应测试点打印不匹配的信息。你可以进行比对分析为什么自己的逻辑没能给出正确的输出。

### 扩展

整个项目是是由`cmake`管理的，如果你想知道编译命令为什么这么写，请打开每个模块下的`CMakeLists.txt`

```cmake
set(SOURCES lru_replacer.cpp clock_replacer.cpp)
add_library(lru_replacer STATIC ${SOURCES})
add_library(clock_replacer STATIC ${SOURCES})

add_executable(lru_replacer_test lru_replacer_test.cpp)
target_link_libraries(lru_replacer_test lru_replacer gtest_main)  # add gtest



add_executable(clock_replacer_test clock_replacer_test.cpp)
target_link_libraries(clock_replacer_test clock_replacer gtest_main)  # add gtest
```

这里将`lru_replacer_test.cpp`生成可执行文件`lru_replacer_test`，因此你需要`make lru_replacer_test`来生成测试可执行文件，其他实验测试命名以依据此规范。
