# Customized Data Type

sc_signal<T> 和 sc_fifo<T> 可用于各种数据类型。SystemC已经支持T为内置数据类型。
为了对sc_signal和sc_fifo使用自定义数据类型，应为数据类型实现以下成员函数：

  1. 赋值操作符，即operator=()：读取和写入方法都需要
  2. 等值运算符，即operator==()：sc_signal的value_changed_event()需要用
  3. 流输出, 即ostream& operator<<(): 用于打印数据结构
  4. sc_trace(): 允许数据类型与systemC跟踪工具一起使用；允许使用 wavefor查看器查看trace数据。

```cpp
// Learn with Examples, 2020, MIT license
#include <systemc>
#include <ostream>
using namespace sc_core;

struct CUSTOMIZED_TYPE {
  int x, y; // 成员变量
  CUSTOMIZED_TYPE(int x = 0, int y = 0) : x(x), y(y) {} // 构造函数
  CUSTOMIZED_TYPE& operator=(const CUSTOMIZED_TYPE& rhs) { // 赋值运算符, 用于read() write()
    x = rhs.x;
    y = rhs.y;
    return *this;
  }
  bool operator==(const CUSTOMIZED_TYPE& rhs) { // 等值运算符,用于value_changed_event()
    return x == rhs.x && y == rhs.y;
  }
};
std::ostream& operator<<(std::ostream& os, const CUSTOMIZED_TYPE& val) { // 流输出，用于打印
  os << "x = " << val.x << "; y = " << val.y << std::endl;
  return os;
}
inline void sc_trace(sc_trace_file*& f, const CUSTOMIZED_TYPE& val, std::string name) { // 用于跟踪
  sc_trace(f, val.x, name + ".x");
  sc_trace(f, val.y, name + ".y");
}

SC_MODULE(MODULE) { // 测试模块
  sc_signal<CUSTOMIZED_TYPE> s; // 定制信号
  SC_CTOR(MODULE) { // 构造函数
    SC_THREAD(writer); // 写进程
    SC_THREAD(reader); // 读进程
    sensitive << s; // 对定制信号s敏感
    dont_initialize();
  }
  void writer() {
    int x = 1; // 初始化信号
    int y = 2;
    while (true) {
      s.write(CUSTOMIZED_TYPE{x++, y++}); // 写到信号
      wait(1, SC_SEC); // 每隔1秒写
    }
  }
  void reader() {
    while (true) {
      std::cout << sc_time_stamp() << ": receives " << s.read() << std::endl; // 从信号读
      wait(); // 等待value_changed_event
    }
  }
};

int sc_main(int, char*[]) {
  MODULE module("module"); // 实例化模块
  sc_trace_file* file = sc_create_vcd_trace_file("trace"); // 打开跟踪文件
  sc_trace(file, module.s, "customized_type"); // 跟踪定制信号
  sc_start(2, SC_SEC); // 运行仿真2秒
  sc_close_vcd_trace_file(file); // 关闭跟踪文件
  return 0;
}
```
