# Signal: read and write

sc_signal:

  1. 是一个预定义的原始通道，用于模拟单根导线传输数字电子信号的行为。
  2. 使用"评估-更新"方案，以确保在同时读和写时的确定性行为。我们维护当前值和新值。
  3. 如果新值与当前值不同，其write()方法将提交更新请求。
  4. 实现sc_signal_inout_if<T>接口.

构造函数:

  1. sc_signal(): 在初始化列表中调用基类构造函数：sc_prim_channel(sc_gen_unique_name("signal"))
  2. sc_signal(const char* name_): 在初始化列表中调用基类构造函数：sc_prim_channel(name_)

成员函数:

  1. T& read() or operator const T& (): 返回信号当前值的引用，不修改信号的状态。
  2. void write(const T&): 会修改信号值，在下一个delta周期更新值（由成员函数read返回），但在此之前不会更新。
  3. operator=: 和write()等效
  4. sc_event& default_event(), sc_event& value_changed_event(): 返回值改变事件的引用。
  5. bool event(): 当信号值在前一个delta周期的更新阶段和当前模拟时间发生变化时，才返回true。

和FIFO相比:

  1. sc_signal只有一个用于读写的槽
  2. sc_signal只有新值和当前值不同时触发更新
  3. sc_signal的读不会删除值

除执行阶段外，sc_signal:

  1. 可以在阐述过程中写入，以初始化信号值。
  2. 可在阐述过程中或模拟暂停时，即调用函数sc_start之前或之后，从函数sc_main写入。

```cpp
// Learn with Examples, 2020, MIT license
#include <systemc>
using namespace sc_core;

SC_MODULE(SIGNAL) {
  sc_signal<int> s;
  SC_CTOR(SIGNAL) {
    SC_THREAD(readwrite);
  }
  void readwrite() {
    s.write(3);
    std::cout << "s = " << s << "; " << s.read() << std::endl;
    wait(SC_ZERO_TIME);
    std::cout << "after delta_cycle, s = " << s << std::endl;
    
    s = 4;
    s = 5;
    int tmp = s;
    std::cout << "s = " << tmp << std::endl;
    wait(SC_ZERO_TIME);
    std::cout << "after delta_cycle, s = " << s.read() << std::endl;
  }
};

int sc_main(int, char*[]) {
  SIGNAL signal("signal");
  signal.s = -1;
  sc_start();
  return 0;
}
```
  
> s = -1; -1  
> after delta_cycle, s = 3  
> s = 3  
> after delta_cycle, s = 5
