# Semaphore

A semaphore:

  1. 是一个预定义通道，用于模拟软件信号的行为，以提供对共享资源的有限并发访问。
  2. 有一个整数值，即semaphore值，它在构建时被设置为允许的并发访问次数。如果初始值为1，则该semaphore相当于一个互斥器。

成员函数:

  1. int wait():
    a) 如果semaphore值大于0，则递减semaphore值并返回。
    b) 如果semaphore值等于0，则暂停，直到信号量值递增（通过另一个进程）。
    c) 无条件返回0。
  2. int trywait():
    a) 如果semaphore值大于0，将递减寄存器值并返回0。
    b) 如果semaphore值等于0，立即返回-1,不修改。semaphore值。
  3. int post():
    a) 将递增semaphore值；
    b) 给所有等待的进程发出增大semaphore值的即时通知。
    c) 无条件返回0。
  4. int get_value(): 返回semaphore值.

```cpp
// Learn with Examples, 2020, MIT license
#include <systemc>
using namespace sc_core;

SC_MODULE(SEMAPHORE) {
  sc_semaphore s; // 声明semaphore
  SC_CTOR(SEMAPHORE) : s(2) { // 初始化semaphore值为2
    SC_THREAD(thread_1); // 注册3个进程竞争2个资源
    SC_THREAD(thread_2);
    SC_THREAD(thread_3);
  }
  void thread_1() {
    while (true) {
      if (s.trywait() == -1) { // 尝试获取资源
        s.wait(); // 没成功就继续等到可用
      }
      std::cout<< sc_time_stamp() << ": locked by thread_1, value is " << s.get_value() << std::endl;
      wait(1, SC_SEC); // 占用资源1秒
      s.post(); // 释放资源
      std::cout<< sc_time_stamp() << ": unlocked by thread_1, value is " << s.get_value() << std::endl;
      wait(SC_ZERO_TIME); // 给其他进程时间获取
    }
  }
  void thread_2() {
    while (true) {
      if (s.trywait() == -1) { // 尝试获取资源
        s.wait(); // 没成功就继续等到可用
      }
      std::cout<< sc_time_stamp() << ": locked by thread_2, value is " << s.get_value() << std::endl;
      wait(1, SC_SEC); // 占用资源1秒
      s.post(); // 释放资源
      std::cout<< sc_time_stamp() << ": unlocked by thread_2, value is " << s.get_value() << std::endl;
      wait(SC_ZERO_TIME); // 给其他进程时间获取
    }
  }
  void thread_3() {
    while (true) {
      if (s.trywait() == -1) { // 尝试获取资源
        s.wait(); // 没成功就继续等到可用
      }
      std::cout<< sc_time_stamp() << ": locked by thread_3, value is " << s.get_value() << std::endl;
      wait(1, SC_SEC); // 占用资源1秒
      s.post(); // 释放资源
      std::cout<< sc_time_stamp() << ": unlocked by thread_3, value is " << s.get_value() << std::endl;
      wait(SC_ZERO_TIME); // 给其他进程时间获取
    }
  }
};

int sc_main(int, char*[]) {
  SEMAPHORE semaphore("semaphore");
  sc_start(4, SC_SEC);
  return 0;
}
```
