# Combined Event Queue

1. 多个事件队列可以用"OR"连接，组成进程的静态敏感性。不能在静态敏感中使用"AND"。
2. 事件队列不能用作wait()的输入，因此不能用于动态敏感。

```cpp
// Learn with Examples, 2020, MIT license
#include <systemc>
using namespace sc_core;

SC_MODULE(QUEUE_COMBINED) {
  sc_event_queue eq1, eq2;
  SC_CTOR(QUEUE_COMBINED) {
    SC_THREAD(trigger);
    SC_THREAD(catcher);
    sensitive << eq1 << eq2; // eq1或eq2，不能“和”
    dont_initialize();
  }
  void trigger() {
    eq1.notify(1, SC_SEC); // 1秒时触发事件eq1
    eq1.notify(2, SC_SEC); // 2秒时触发事件eq1
    eq2.notify(2, SC_SEC); // 2秒时触发事件eq2
    eq2.notify(3, SC_SEC); // 3秒时触发事件eq2
  }
  void catcher() {
    while (true) {
      std::cout << sc_time_stamp() << ": catches trigger" << std::endl;
      wait(); // 不能在动态敏感中使用事件队列
    }
  }
};

int sc_main(int, char*[]) {
  QUEUE_COMBINED combined("combined");
  sc_start();
  return 0;
}
```

> 1 s: catches trigger  
> 2 s: catches trigger  
> 3 s: catches trigger
