# Communication: specialized ports

除了使用基本的sc_port类来声明端口外，还有各种专门的端口类可用于不同的通道类型或提供附加功能：

   1. sc_in用于信号的专用端口类。
   2. sc_fifo_in是从FIFO读取数据的专用端口类。
   3. sc_fifo_out是写入FIFO时使用的专用端口类。
   4. sc_in<bool>和sc_in<sc_dt::sc_logic>: value_changed(), pos(), neg()
   5. sc_inout是用于信号的专用端口类: value_changed(), initialize()
   6. sc_inout<bool> 和 sc_inout<sc_dt::sc_logic> 为二值信号提供额外成员函数的专用端口类: value_changed(), initialize(), pos(), neg()
   7. sc_out派生自类sc_inout，与类sc_inout相同，只是它作为派生类所固有的差异，例如构造函数和赋值运算符。
   8. sc_in_resolved是用于解析信号的专用端口类。它类似于派生它的sc_in<sc_dt::sc_logic>。唯一的区别是，sc_in_resolved类的端口应绑定到sc_signal_resolved类的信道，而sc_in<sc_dt::sc_logic>类的端口可以绑定到sc_signal<sc_dt::sc_logic，WRITER_POLICY>或类sc_signal_resolved。
   9. sc_inout_resolved是用于解析信号的专用端口类。它的行为类似于派生它的端口类sc_inout<sc_dt::sc_logic>。唯一的区别是，sc_inout_resolved类的端口应绑定到sc_signal_resolved类的信道，而sc_inout<sc_dt::sc_logic>类的端口可以绑定到sc_signal<sc_dt::sc_logic，WRITER_POLICY>类或sc_signal_resolved类的信道。
   10. sc_out_resolved派生自类sc_inout_resolved，它与类 sc_inout_resolved相同，只是它作为派生类所固有的差异，例如构造函数和赋值运算符。
   11. sc_in_rv是用于解析信号的专用端口类。它的行为类似于派生它的端口类 sc_in<sc_dt::<W>sc_lv>。唯一的区别是，sc_in_rv类的端口应绑定到sc_signal_rv类的信道，而sc_in<sc_dt::sc_lv>类的端口<W>可以绑定到sc_signal<sc_dt::<W>sc_lv，WRITER_POLICY>类的信道或sc_signal_rv类。
   12. sc_inout_rv是用于解析信号的专用端口类。它的行为类似于派生它的端口类sc_inout<sc_dt::<W>sc_lv>。唯一的区别是，sc_inout_rv类的端口应绑定到sc_signal_rv类的信道，而sc_inout<sc_dt::sc_lv>类的端口<W>可以绑定到sc_signal<sc_dt::<W>sc_lv，WRITER_POLICY>或sc_signal_rv类。
   13. sc_out_rv派生自类sc_inout_rv，它与类sc_inout_rv相同，只是它作为派生类所固有的差异，例如构造函数和赋值运算符。

一个基础的sc_port<sc_signal_inout_if<int>>只能访问信号通道提供的成员功能：

   1. read()
   2. write()
   3. default_event() // 当端口用于定义静态敏感时通过sc_sensitive的操作符<<来调用
   4. event() // 检查事件是否发生，返回true/false
   5. value_changed_event() // 值改变的事件

sc_port<sc_signal_inout_if<bool>>可以访问signal<bool>通道提供的这些附加成员函数:

   6. posedge() // 如果值从false更改为true则返回true
   7. posedge_event() // 值从false更改为true的事件
   8. negedge() // 如果值从true变为false则返回true
   9. negedge_event() // 值从true更改为false的事件

sc_inout<>端口专门提供额外的成员功能：

  10. initialize() // 在端口绑定到通道之前初始化端口的值
  11. value_changed() // 用于在端口绑定到通道之前创建敏感（指针未初始化）

当信号通道类型为bool或sc_logic时，sc_inout<bool>提供另外两个成员函数：

  12. pos() // 端口绑定前创建敏感
  13. neg() // 端口绑定前创建敏感

上述成员函数：
  1~9由信号通道提供，可通过port->method()访问;
  10~13由专用端口提供，可通过port.method()访问。

```cpp
// Learn with Examples, 2020, MIT license
#include <systemc>
using namespace sc_core;

SC_MODULE(WRITER) {
  sc_out<bool> p1, p2; // 专用端口1 2
  SC_CTOR(WRITER) {
    SC_THREAD(writer);
    p1.initialize(true); // #10, 初始化默认值为true
  }
  void writer() {
    bool v = true;
    while (true) {
      p1->write(v); // #2 写到端口
      v = !v; // 改变值
      wait(1, SC_SEC); // 每1秒重复
    }
  }
};
SC_MODULE(READER) {
  sc_in<bool> p1, p2; // 专用端口
  SC_CTOR(READER) {
    SC_THREAD(reader1);
    sensitive << p1 << p2; // #3 default_event()等效于p->default_event()或p.default_event()
    dont_initialize();
    SC_THREAD(reader2);
    sensitive << p1.value_changed(); // #11 对未绑定端口的值改变事件敏感
    dont_initialize();
    SC_THREAD(reader3);
    sensitive << p1.neg(); // #13 对未绑定端口的neg事件敏感
    dont_initialize();
    SC_THREAD(reader4);
    sensitive << p1.pos(); // #12 对未绑定端口的pos事件敏感
    dont_initialize();
  }
  void reader1() {
    while (true) {
      std::cout << sc_time_stamp() << ": default_event. p1 = " << p1->read() << "; p1 triggered? " << p1->event() << "; p2 triggered? " << p2->event() << std::endl; // #1 read(), #4 event()
      wait();
    }
  }
  void reader2() {
    while (true) {
      std::cout << sc_time_stamp() << ": value_changed_event. p1 = " << p1->read() <<  std::endl; // #1 read()
      wait();
    }
  }
  void reader3() {
    while (true) {
      std::cout << sc_time_stamp() << ": negedge_event. p1 = " << p1->read() << "; negedge = " << p1->negedge() << std::endl; // #8 如果下降沿出现
      wait();
    }
  }
  void reader4() {
    while (true) {
      std::cout << sc_time_stamp() << ": posedge_event. p1 = " << p1->read() <<  "; posedge = " << p1->posedge() << std::endl; // #6 如果上升沿出现
      wait();
    }
  }
};

int sc_main(int, char*[]) {
  WRITER writer("writer"); // 实例化writer
  READER reader("reader"); // 实例化reader
  sc_signal<bool> b1, b2; // 声明bool值信号通道
  writer.p1(b1); // 绑定端口
  writer.p2(b2);
  reader.p1(b1);
  reader.p2(b2);
  sc_start(4, SC_SEC);
  return 0;
}
```
