# sc_signal<bool>

sc_signal_in_if<bool> and sc_signal_in_if<sc_dt::sc_logic>是提供了适用于两值信号的成员函数的接口。sc_signal实现了这些函数:

  1. posedge_event() 返回对每当通道的值发生更改且通道的新值为true或“1”时通知的事件的引用。
  2. negedge_event() 返回对每当通道的值发生更改且通道的新值为false或“0”时通知的事件的引用。
  3. posedge() 当且仅当通道值在前一个delta周期的更新阶段和当前模拟时间发生变化，且通道的新值为true或"1"时，才返回true。
  4. negedge() 当且仅当通道值在前一个delta周期的更新阶段和当前模拟时间发生变化，且通道的新值为false或"0"时，才返回true。

```cpp
// Learn with Examples, 2020, MIT license
#include <systemc>
using namespace sc_core;

SC_MODULE(SIGNAL_BOOL) {
  sc_signal<bool> b;
  SC_CTOR(SIGNAL_BOOL) {
    SC_THREAD(writer);
    SC_THREAD(consumer);
    sensitive << b; // 每次值改变都触发
    dont_initialize();
    SC_THREAD(consumer_pos);
    sensitive << b.posedge_event(); // 值改为true时触发
    dont_initialize();
    SC_THREAD(consumer_neg);
    sensitive << b.negedge_event(); // 值改为false时触发
    dont_initialize();
  }
  void writer() {
    bool v = true;
    while (true) {
      b.write(v); // 写true到信号b
      v = !v; // 翻转值
      wait(1, SC_SEC); // 每隔1秒写一次
    }
  }
  void consumer() {
    while (true) {
      if (b.posedge()) { // 如果新值是true
        std::cout << sc_time_stamp() << ": consumer receives posedge, b = " << b << std::endl;
      } else { // 如果新值是false
        std::cout << sc_time_stamp() << ": consumer receives negedge, b = " << b << std::endl;
      }
      wait(); // 等到任何值改变事件
    }
  }
  void consumer_pos() {
    while (true) {
      std::cout << sc_time_stamp() << ": consumer_pos receives posedge, b = " << b << std::endl;
      wait(); // 等到值改变到true
    }
  }
  void consumer_neg() {
    while (true) {
      std::cout << sc_time_stamp() << ": consumer_neg receives negedge, b = " << b << std::endl;
      wait(); // 等到值改变到false
    }
  }
};

int sc_main(int, char*[]) {
  SIGNAL_BOOL signal_bool("signal_bool");
  sc_start(4, SC_SEC);
  return 0;
}
```

> 0 s: consumer_pos receives posedge, b = 1  
> 0 s: consumer receives posedge, b = 1  
> 1 s: consumer_neg receives negedge, b = 0  
> 1 s: consumer receives negedge, b = 0  
> 2 s: consumer_pos receives posedge, b = 1  
> 2 s: consumer receives posedge, b = 1  
> 3 s: consumer_neg receives negedge, b = 0  
> 3 s: consumer receives negedge, b = 0
