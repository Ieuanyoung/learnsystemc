# Clock

sc_clock是一个从sc_signal类派生出来的预定义原始通道，用于模拟数字时钟信号的行为。
与时钟相关的值和事件可通过sc_signal_in_if<bool>接口访问。

Constructor:
sc_clock(
  const char* name_, // 独特模块名
  double period_v_, // 从false到true的两次连续转换之间的时间间隔，也等于从true到false的两次连续转换之间的时间间隔。大于零，默认值为1纳秒。
  sc_time_unit period_tu_, // 周期时间单位
  double duty_cycle_, // 时钟值为true的周期比例。介于0.0和1.0之间，不重复。默认值为0.5。
  double start_time_v_, // 时钟值第一次转换（false到true或true到false）的绝对时间。默认值为0。
  sc_time_unit start_time_tu_,
  bool posedge_first_ = true ); // 如果为true，则时钟初始化为false，并在开始时间更改为true。反之亦然。默认值为true。

```cpp
// Learn with Examples, 2020, MIT license
#include <systemc>
using namespace sc_core;

SC_MODULE(CLOCK) {
  sc_port<sc_signal_in_if<bool>> clk; // 访问时钟的端口
  SC_CTOR(CLOCK) {
    SC_THREAD(thread); // 注册进程
    sensitive << clk; // 对时钟敏感
    dont_initialize();
  }
  void thread() {
    while (true) {
      std::cout << sc_time_stamp() << ", value = " << clk->read() << std::endl; // 打印当前时钟
      wait(); // 等待下一个时钟值改变
    }
  }
};

int sc_main(int, char*[]) {
  sc_clock clk("clk", 10, SC_SEC, 0.2, 10, SC_SEC, false); // 10秒周期，2秒true，8秒false，从10秒开始，从false开始。
  CLOCK clock("clock"); // 实例化模块
  clock.clk(clk); // 绑定端口
  sc_start(31, SC_SEC); // 运行仿真31秒
  return 0;
}
```
