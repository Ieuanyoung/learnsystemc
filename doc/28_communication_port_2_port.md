# Communication: port 2 port

到目前为止，我们已经介绍了:

  1. 通过通道连接同一模块的两个进程：process1() --> channel --> process2()
  2. 通过端口和通道连接两个不同模块的进程：
      module1::process1() --> module1::port1 --> channel --> module2::port2 --> module2::process2()
  3. 通过出口连接两个不同模块的进程：
      module1::process1() --> module1::channel --> module1::export1 --> module2::port2 --> module2::process2()

在所有这些情况下，都需要一个通道来连接端口。有一种特殊情况允许端口直接连接到子模块的端口：module::port1 --> module::submodule::port2

```cpp
// Learn with Examples, 2020, MIT license
#include <systemc>
using namespace sc_core;

SC_MODULE(SUBMODULE1) { // 写到通道的子模块
  sc_port<sc_signal_out_if<int>> p;
  SC_CTOR(SUBMODULE1) {
    SC_THREAD(writer);
  }
  void writer() {
    int val = 1; // 初始化值
    while (true) {
      p->write(val++); // 通过端口写入通道
      wait(1, SC_SEC);
    }
  }
};
SC_MODULE(SUBMODULE2) { // 读通道的子模块
  sc_port<sc_signal_in_if<int>> p;
  SC_CTOR(SUBMODULE2) {
    SC_THREAD(reader);
    sensitive << p; // 通道值改变触发
    dont_initialize();
  }
  void reader() {
    while (true) {
      std::cout << sc_time_stamp() << ": reads from channel, val=" << p->read() << std::endl;
      wait(); // 通过通道接收值
    }
  }
};
SC_MODULE(MODULE1) { // 顶层模块
  sc_port<sc_signal_out_if<int>> p; // 端口
  SUBMODULE1 sub1; // 声明子模块
  SC_CTOR(MODULE1): sub1("sub1") { // 实例化子模块
    sub1.p(p); // 把子模块的端口绑到父模块端口
  }
};
SC_MODULE(MODULE2) {
  sc_port<sc_signal_in_if<int>> p;
  SUBMODULE2 sub2;
  SC_CTOR(MODULE2): sub2("sub2") {
    sub2.p(p); // 把子模块的端口绑到父模块端口
  }
};

int sc_main(int, char*[]) {
  MODULE1 module1("module1"); // 实例化模块1
  MODULE2 module2("module2"); // 实例化模块2
  sc_signal<int> s; // 在模块1和2外面定义信号
  module1.p(s); // 把子模块1的端口绑到信号，用于写
  module2.p(s); // 把子模块2的端口绑到信号，用于读
  sc_start(2, SC_SEC);
  return 0;
}
```

> 0 s: reads from channel, val=1  
> 1 s: reads from channel, val=2
