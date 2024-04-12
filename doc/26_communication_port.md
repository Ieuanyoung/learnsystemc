# Communication: port

通信的三个关键概念:

  1. Interface(接口):
    a) 一个从sc_interface派生但不从sc_object派生的抽象类。
    b) 包含一组纯虚拟函数，这些函数应在该接口派生的一个或多个通道中定义。
  2. Port(端口):
    a) 提供了编写模块的方法，使其独立于实例化模块的上下文。
    b) 将接口方法调用转发给端口绑定的通道。
    c) 定义包含端口的模块所需的一组服务（由端口类型标识）。
  3. Channel(通道):
    a) sc_prim_channel是所有原始通道的基类。
    b) 通道可以提供可以使用接口方法调用范例调用的公共成员函数。
    c) 原始通道应实现一个或多个接口。

简而言之:
  端口需要服务,接口定义服务,通道实现服务。
  如果通道实现了端口所需的接口，端口就可以连接（绑定）到通道。
  端口相当于指向通道的指针。

什么时候使用端口:

  1. 如果模块要调用模块本身之外的通道的成员函数，则应通过模块的端口使用接口方法调用。否则将被视为不良的编码风格。
  2. 不过，也可以直接调用属于当前模块内实例化通道的成员函数。这就是所谓的无端口通道访问。
  3. 如果模块要调用属于子模块内通道实例的成员函数，则该调用应通过子模块的导出进行。

```cpp
// Learn with Examples, 2020, MIT license
#include <systemc>
using namespace sc_core;

SC_MODULE(MODULE1) { // 定义1个模块
  sc_signal<int> s; // 模块内的1个信号(channel通道)
  sc_port<sc_signal_out_if<int> > p; // 用来向外面的通道写值的端口
  SC_CTOR(MODULE1) {
    SC_THREAD(selfWrite); // 写到模块内通道的进程
    SC_THREAD(selfRead); // 从模块内通道读的进程
    sensitive << s; // s值改变触发
    dont_initialize();
    SC_THREAD(outsideWrite); // 写到模块外面通道的进程
  }
  void selfWrite() {
    int val = 1; // 初始化值
    while (true) {
      s.write(val++); // 写到模块内通道
      wait(1, SC_SEC); // 1秒后重复
    }
  }
  void selfRead() {
    while (true) {
      std::cout << sc_time_stamp() << ": reads from own channel, val=" << s.read() << std::endl; // 从模块内通道读
      wait(); // 从s读取
    }
  }
  void outsideWrite() {
    int val = 1; // 初始化值
    while (true) {
      p->write(val++); // 写到模块外通道, 调用模块外通道的写函数. p是个指针
      wait(1, SC_SEC);
    }
  }
};
SC_MODULE(MODULE2) { // 一个从外面通道读值的模块
  sc_port<sc_signal_in_if<int> > p; // 从模块外通道读值的端口
  SC_CTOR(MODULE2) {
    SC_THREAD(outsideRead); // 从模块外通道读的进程
    sensitive << p; // p值改变触发
    dont_initialize();
  }
  void outsideRead() {
    while (true) {
      std::cout << sc_time_stamp() << ": reads from outside channel, val=" << p->read() << std::endl; // 用端口去读通道值，像指针一样
      wait(); // 从端口接收
    }
  }
};

int sc_main(int, char*[]) {
  MODULE1 module1("module1"); // 实例化模块1
  MODULE2 module2("module2"); // 实例化模块2
  sc_signal<int> s; // 在模块1和2外面声明1个信号(通道)
  module1.p(s); // 把模块1的端口p绑到通道s上
  module2.p(s); // 把模块2的端口p绑到通道s上
  sc_start(2, SC_SEC);
  return 0;
}
```
