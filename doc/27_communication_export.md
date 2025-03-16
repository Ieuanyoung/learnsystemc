# Communication: export

export出口:

  1. 允许模块为其父模块提供接口。
  2. 将接口方法调用转发给出口绑定的通道。
  3. 定义了包含该出口的模块所提供的一系列服务。

什么时候使用出口:

  1. 通过出口提供接口是模块简单实现接口的替代方法。
  2. 使用显式出口允许单个模块实例以结构化方式提供多个接口。
  3. 如果模块要调用子模块中属于通道实例的成员函数，则应通过子模块的出口进行调用。

```cpp
// Learn with Examples, 2020, MIT license
#include <systemc>
using namespace sc_core;

SC_MODULE(MODULE1) { // 定义1个模块
  sc_export<sc_signal<int>> p; // 连接其他模块的出口
  sc_signal<int> s; // 模块内部的信号（通道）。如果不使用export，则需要在module1之外定义通道。
  SC_CTOR(MODULE1) {
    p(s); // 把出口绑到内部通道
    SC_THREAD(writer); // 给内部通道写值的进程
  }
  void writer() {
    int val = 1; // 初始化值
    while (true) {
      s.write(val++); // 写入内部通道
      wait(1, SC_SEC);
    }
  }
};
SC_MODULE(MODULE2) { // 从出口读值的模块
  sc_port<sc_signal_in_if<int>> p; // 用来从其他模块的出口读值的端口
  SC_CTOR(MODULE2) {
    SC_THREAD(reader); // 从外面通道读值的进程
    sensitive << p; // 通道值改变触发
    dont_initialize();
  }
  void reader() {
    while (true) {
      std::cout << sc_time_stamp() << ": reads from outside channel, val=" << p->read() << std::endl; // 用端口去读值，像指针
      wait(); // 从端口接收
    }
  }
};

int sc_main(int, char*[]) {
  MODULE1 module1("module1"); // 实例化模块1
  MODULE2 module2("module2"); // 实例化模块2
  module2.p(module1.p); // 连接模块2的端口到模块1的出口，不需要再在模块外声明一个通道
  sc_start(2, SC_SEC);
  return 0;
}
```

> 0 s: reads from outside channel, val=1  
> 1 s: reads from outside channel, val=2
