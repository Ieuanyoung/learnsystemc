# Trace File

跟踪文件:

  1. 在仿真过程中按时间顺序记录值变化。
  2. 用VCD(Value change dump)文件格式.
  3. 只能用sc_create_vcd_trace_file创建和打开.
  4. 可随时在阐述过程中或模拟过程中打开。
  5. 包含只能通过sc_trace跟踪的值。
  6. 如果在打开文件后已经过了一个或多个delta周期，就不能将数值追踪到给定的跟踪文件。
  7. 应通过 sc_close_vcd_trace_file 关闭。在模拟的最后一个delta周期之前，不得关闭跟踪文件。

```cpp
// Learn with Examples, 2020, MIT license
#include <systemc>
using namespace sc_core;

SC_MODULE(MODULE) { // 写到通道的模块
  sc_port<sc_signal<int>> p; 
  SC_CTOR(MODULE) {
    SC_THREAD(writer); // 写的进程
  }
  void writer() {
    int v = 1;
    while (true) {
      p->write(v++); // 通过端口写到通道
      wait(1, SC_SEC); // 每隔1秒写
    }
  }
};
int sc_main(int, char*[]) {
  MODULE module("module"); // 实例化模块
  sc_signal<int> s; // 声明信号通道
  module.p(s); // 绑定端口和通道

  sc_trace_file* file = sc_create_vcd_trace_file("trace"); // 打开跟踪文件
  sc_trace(file, s, "signal"); // 用“signal”的名字跟踪“s”
  sc_start(5, SC_SEC); // 运行仿真5秒
  sc_close_vcd_trace_file(file); // 关闭文件
  return 0;
}
```
