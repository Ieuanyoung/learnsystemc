# Sensitivity

进程实例的敏感度是指可能导致进程恢复或触发的一系列事件和超时。如果事件被添加到进程实例的静态敏感度或动态敏感度中，则该进程实例对该事件敏感。当超过给定的时间间隔，将发生超时。

两种敏感度:

  1. 静态敏感度在阐述过程中确定下来，模块中的每个进程都有敏感列表。
  2. 动态敏感度可在进程控制下随时间变化，支持线程的wait()或方法的next_trigger()。

```cpp
// Learn with Examples, 2020, MIT license
#include <systemc>
using namespace sc_core;

SC_MODULE(SENSITIVITY) {
  sc_event e1, e2; // 用于进程内部触发的事件
  SC_CTOR(SENSITIVITY) {
    SC_THREAD(trigger_1); // 注册进程
    SC_THREAD(trigger_2);
    SC_THREAD(catch_1or2_dyn);
    SC_THREAD(catch_1or2_static);
    sensitive << e1 << e2; // 前一进程的静态敏感，只能"或"触发
  }
  void trigger_1() {
    wait(SC_ZERO_TIME); // 将延迟一个delta周期触发，确保捕获器已准备就绪
    while (true) {
      e1.notify(); // 通知e1
      wait(2, SC_SEC); // 动态敏感, 2秒后再次触发
    }
  }
  void trigger_2() { // 延迟一个delta周期触发
    wait(SC_ZERO_TIME);
    while (true) {
      e2.notify(); // 触发e2
      wait(3, SC_SEC); // 动态敏感, 3秒后再次触发
    }
  }
  void catch_1or2_dyn() {
    while (true) {
      wait(e1 | e2); // 动态敏感
      std::cout << "Dynamic sensitivty: e1 or e2 @ " << sc_time_stamp() << std::endl;
    }
  }
  void catch_1or2_static(void) {
    while (true) {
      wait(); // 静态敏感
      std::cout << "Static sensitivity: e1 or e2 @ " << sc_time_stamp() << std::endl;
    }
  }
};

int sc_main(int, char*[]) {
  SENSITIVITY sensitivity("sensitivity");
  sc_start(7, SC_SEC);
  return 0;
}
```

> 结果：  
> Static sensitivity: e1 or e2 @ 0 s  
> Dynamic sensitivty: e1 or e2 @ 0 s  
> Static sensitivity: e1 or e2 @ 2 s  
> Dynamic sensitivty: e1 or e2 @ 2 s  
> Static sensitivity: e1 or e2 @ 3 s  
> Dynamic sensitivty: e1 or e2 @ 3 s  
> Static sensitivity: e1 or e2 @ 4 s  
> Dynamic sensitivty: e1 or e2 @ 4 s  
> Static sensitivity: e1 or e2 @ 6 s  
> Dynamic sensitivty: e1 or e2 @ 6 s
