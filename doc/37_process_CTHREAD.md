# Process: CTHREAD

SC_CTHREAD:

  1. 在 SystemC 2.0中已弃用。仍支持第二个参数是事件查找器的情况。
  2. 注册进程时需要时钟。
  3. 无单独敏感列表
  4. 每当出现指定的时钟边沿就会激活。

```cpp
// Learn with Examples, 2020, MIT license
#include <systemc>
using namespace sc_core;

SC_MODULE(MODULE) {
  sc_in<bool> clk; // 需要event_finder方法,不能用基础的sc_port
  SC_CTOR(MODULE) {
    SC_CTHREAD(cthread1, clk); // 对时钟上升沿敏感
    // 没有静态敏感，所以不能用dont_initialize()
    SC_CTHREAD(cthread2, clk.pos()); // 对时钟上升沿敏感
    SC_CTHREAD(cthread3, clk.neg()); // 对时钟下降沿敏感
  }
  void cthread1() {
    while (true) {
      wait(); // 等待时钟上升沿; wait()紧接while循环以避免初始化
      std::cout << sc_time_stamp() << ", cthread1, value = " << clk->read() << std::endl;
    }
  }
  void cthread2() {
    while (true) {
      wait(); // 等待时钟上升沿
      std::cout << sc_time_stamp() << ", cthread2, value = " << clk->read() << std::endl;
    }
  }
  void cthread3() {
    while (true) {
      wait(); // 等待时钟下降沿
      std::cout << sc_time_stamp() << ", cthread3, value = " << clk->read() << std::endl;
    }
  }
};

int sc_main(int, char*[]) {
  sc_clock clk("clk", 10, SC_SEC, 0.2, 10, SC_SEC, false); // 10秒周期，2秒true，8秒false，从10秒开始，从false开始。
  MODULE module("module");
  module.clk(clk);
  sc_start(31, SC_SEC);
  return 0;
}
```
