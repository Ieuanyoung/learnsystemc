# Mutex

互斥锁:

  1. 是一个预定义通道，用于模拟互斥锁的行为，以控制对并发进程共享资源的访问。
  2. 应处于两个排他性状态之一(解锁或锁):
    a) 同一时间只能有一个进程给互斥锁上锁。
    b) 互斥锁只能由上锁的进程解锁。解锁后，互斥锁可以被不同的进程锁定。

成员函数:

  1. int lock():
    a) 如果互斥解锁，lock()将锁定互斥并返回。
    b) 如果互斥项被锁定，lock()将暂停运行直到被其他进程解锁。
    c) 如果多个进程试图在同一个delta循环中锁定互斥，那不确定哪个进程实例能获得锁。
    d) 将无条件返回0。
  2. int trylock():
    a) 如果互斥锁已解锁，将锁定互斥锁并返回0。
    b) 如果互斥项被锁定，将立即返回-1。保持锁定状态。
  3. int unlock():
    a) 如果互斥解锁，将返回-1。保持解锁状态。
    b) 如果互斥项被调用进程以外的进程实例锁定，返回-1。保持锁定状态。
    c) 如果互斥项被调用进程锁定，解锁互斥项并返回0。
即时通知应被用来向其他进程发出解锁互斥项的信号。

```cpp
// Learn with Examples, 2020, MIT license
#include <systemc>
using namespace sc_core;

SC_MODULE(MUTEX) {
  sc_mutex m;
  SC_CTOR(MUTEX) {
    SC_THREAD(thread_1);
    SC_THREAD(thread_2);
  }
  void thread_1() {
    while (true) {
      if (m.trylock() == -1) { // 尝试上锁
        m.lock(); // 失败，等待锁定
        std::cout << sc_time_stamp() << ": thread_1 obtained resource by lock()" << std::endl;
      } else { // 成功
        std::cout << sc_time_stamp() << ": thread_1 obtained resource by trylock()" << std::endl;
      }
      wait(1, SC_SEC); // 占据锁1秒
      m.unlock(); // 解锁
      std::cout << sc_time_stamp() << ": unlocked by thread_1" << std::endl;
      wait(SC_ZERO_TIME); // 给其他进程时间去上锁
    }
  }
  void thread_2() {
    while (true) {
      if (m.trylock() == -1) { // 尝试上锁
        m.lock(); // failed, wait to lock
        std::cout << sc_time_stamp() << ": thread_2 obtained resource by lock()" << std::endl;
      } else { // succeeded
        std::cout << sc_time_stamp() << ": thread_2 obtained resource by trylock()" << std::endl;
      }
      wait(1, SC_SEC); // 占据锁1秒
      m.unlock(); // 解锁
      std::cout << sc_time_stamp() << ": unlocked by thread_2" << std::endl;
      wait(SC_ZERO_TIME); // 给其他进程时间去上锁
    }
  }
};

int sc_main(int, char*[]) {
  MUTEX mutex("mutex");
  sc_start(4, SC_SEC);
  return 0;
}

```
