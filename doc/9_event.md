# Event

事件是用于进程同步的sc_event类对象。
进程实例可以在事件发生时触发或恢复，即当被通知事件时。任何给定的事件都可能在不同的场合通知。

sc_event成员函数:

  1. void notify(): 创建即时通知
  2. void notify(const sc_time&), void notify(double, sc_time_unit):  
    a) 0时间:创建delta通知  
    b) 非0时间:创建定时通知
  3. cancel(): 删除此事件的任何待发通知  
    a) 任何给定事件最多只能有一个待处理通知。  
    b) 即时通知无法取消。

约束:

  1. sc_event类对象可在阐述或模拟过程中构建。
  2. 事件可在阐述或模拟过程中发出通知，但如果在阐述过程中或通过某个回调立即发出通知，则属于错误行为:  
    a) before_end_of_elaboration,  
    b) end_of_elaboration, or  
    c) start_of_simulation.

某一事件的待发通知不得超过一份：

  1. 如果事件已经有通知待发，则只有应该最早发的通知才会有效。
  2. 原定于稍后时间发生的通知应被取消（或从一开始就不安排）。
  3. 即时通知的发生时间早于delta通知，delta通知的发生时间早于定时通知。这与调用函数通知的顺序无关。

事件可以相互组合，也可以与计时器组合。此示例显示一个进程仅等待一个事件。

```cpp
// Learn with Examples, 2020, MIT license
#include <systemc>
using namespace sc_core;

SC_MODULE(EVENT) {
  sc_event e; // 声明事件
  SC_CTOR(EVENT) {
    SC_THREAD(trigger); // 注册一个触发进程
    SC_THREAD(catcher); // 注册一个捕获进程
  }
  void trigger() {
    while (true) { // 无限循环
      e.notify(1, SC_SEC); // 1秒后触发
      if (sc_time_stamp() == sc_time(4, SC_SEC)) {
        e.cancel(); // 4秒时取消事件触发
      }
      wait(2, SC_SEC); // 等2秒再触发
    }
  }
  void catcher() {
    while (true) { // 无限循环
      wait(e); // 等待事件
      std::cout << "Event cateched at " << sc_time_stamp() << std::endl; // 打印到控制台
    }
  }
};

int sc_main(int, char*[]) {
  EVENT event("event"); // 定义对象
  sc_start(8, SC_SEC); // 运行仿真8秒
  return 0;
}
```

> 结果：  
> Event cateched at 1 s  
> Event cateched at 3 s  
> Event cateched at 7 s  
