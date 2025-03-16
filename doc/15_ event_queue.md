# Event Queue

事件队列:

  1. 与事件相同,具有成员函数notify()；
  2. 是一个分层通道，可以有多个待发通知，这与事件不同，事件只能安排一个待发通知；
  3. 只能在阐述过程中构造；
  4. 不支持即时通知。

成员函数:

  1. void notify(double, sc_time_unit)或void notify(const sc_time&):
    a) 0时(SC_ZERO_TIME): delta通知
    b) 非0时: 相对于notify函数调用时的模拟时间安排
  2. void cancel_all(): 立即删除此事件队列对象的所有待发通知，包括 delta和定时通知。

```cpp
// Learn with Examples, 2020, MIT license
#include <systemc>
using namespace sc_core;

SC_MODULE(QUEUE) {
  sc_event e;
  sc_event_queue eq;
  SC_CTOR(QUEUE) {
    SC_THREAD(trigger);
    SC_THREAD(catch_e);
    sensitive << e; // catch_e()将会被事件e触发
    dont_initialize(); // cach_e()不在初始化阶段运行
    SC_THREAD(catch_eq);
    sensitive << eq; // cach_eq()将会被事件eq触发
    dont_initialize(); // catch_eq()不在初始化阶段运行
  }
  void trigger() {
    while (true) {
      e.notify(2, SC_SEC); // 2秒后触发事件e
      e.notify(1, SC_SEC); // 1秒后触发，替代前面的触发
      eq.notify(2, SC_SEC); // 2秒后触发事件eq
      eq.notify(1, SC_SEC); // 1秒后触发，两个触发都有效
      wait(10, SC_SEC); // 另一轮
    }
  }
  void catch_e() {
    while (true) {
      std::cout << sc_time_stamp() << ": catches e" << std::endl;
      wait(); // 没参数 --> 等待静态敏感比如事件e
    }
  }
  void catch_eq() {
    while (true) {
      std::cout << sc_time_stamp() << ": catches eq" << std::endl;
      wait(); // 等待事件eq
    }
  }
};

int sc_main(int, char*[]) {
  QUEUE queue("queue"); // 实例化
  sc_start(20, SC_SEC); // 运行仿真20秒
  return 0;
}
```

1 s: catches e
1 s: catches eq
2 s: catches eq
11 s: catches e
11 s: catches eq
12 s: catches eq
