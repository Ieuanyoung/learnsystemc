# Signal: detect event

1. sc_event& default_event(), sc_event& value_changed_event(): 返回对改变值的事件的引用。
2. bool event(): 当且仅当信号值在前一个delta周期的更新阶段和当前仿真时间发生变化时，返回true。

```cpp
// Learn with Examples, 2020, MIT license
#include <systemc>
using namespace sc_core;

SC_MODULE(SIGNAL_EVENT) {
  sc_signal<int> s1, s2; // 定义两个信号
  SC_CTOR(SIGNAL_EVENT) {
    SC_THREAD(producer1);
    SC_THREAD(producer2);
    SC_THREAD(consumer); // consumer对s1/s2敏感
    sensitive << s1 << s2; // 等效于：sensitive << s1.default_event() << s2.value_changed_event();
    dont_initialize();
  }
  void producer1() {
    int v = 1;
    while (true) {
      s1.write(v++); // 写到s1
      wait(2, SC_SEC);
    }
  }
  void producer2() {
    int v = 1;
    while (true) {
      s2 = v++; // 写到s2
      wait(3, SC_SEC);
    }
  }
  void consumer() {
    while (true) {
      if ( s1.event() == true && s2.event() == true) { // 都触发
        std::cout << sc_time_stamp() << ": s1 & s2 triggered" << std::endl; 
      } else if (s1.event() == true) { // 只有s1触发
        std::cout << sc_time_stamp() << ": s1 triggered" << std::endl; 
      } else { // 只有s2触发
        std::cout << sc_time_stamp() << ": s2 triggered" << std::endl; 
      }
      wait();
    }
  }
};

int sc_main(int, char*[]) {
  SIGNAL_EVENT signal_event("signal_event");
  sc_start(7, SC_SEC);
  return 0;
}
```

> 0 s: s1 & s2 triggered  
> 2 s: s1 triggered  
> 3 s: s2 triggered  
> 4 s: s1 triggered  
> 6 s: s1 & s2 triggered
