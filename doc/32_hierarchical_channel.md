# Hierarchical Channel

分层通道:

  1. 应继承sc_channel基类，它与sc_module相同。因此，分层通道就是一个 systemC模块。
  2. 应继承于一个接口，以便连接到一个端口。

与普通的systemC模块一样，分层通道可能有模拟进程、端口等。

本例展示了一个实现sc_signal_inout_if<int>的自定义分层通道。根据 sc_signal_inout_if的定义，我们必须实现以下函数：

  1. void write(const int&)
  2. const int& read() const
  3. const sc_event& value_changed_event() const
  4. const sc_event& default_event() const
  5. const int& get_data_ref() const
  6. bool event() const

```cpp
// Learn with Examples, 2020, MIT license
#include <systemc>
using namespace sc_core;

// 与 sc_signal 相比，这是一个简单的实现，只是为了说明分层通道的概念
class SIGNAL : public sc_channel, public sc_signal_inout_if<int> { // 声明SIGNAL通道, 继承自sc_chanel和signal_inout_if<int>
public:
  SC_HAS_PROCESS(SIGNAL);
  SIGNAL(sc_module_name name = sc_gen_unique_name("SIG")) : sc_channel(name) {} // 构造函数
  void write(const int& v) { // 实现写方法
    if (v != m_val) { // 只有值是新的时候更新
      m_val = v; // 更新值
      e.notify(); // 触发事件
    }
  }
  const int& read() const {
    return m_val;
  }
  const sc_event& value_changed_event() const {
    return e; // 返回事件的引用
  }
  const sc_event& default_event() const {
    return value_changed_event(); // 允许用于静态敏感列表
  }
  const int& get_data_ref() const {
    return m_val;
  }
  bool event() const {
    return true; // 虚拟实现，始终返回 true
  }
private:
  int m_val = 0;
  sc_event e;
};

SC_MODULE(TEST) { // 测试类
  SIGNAL s; // 声明SIGNAL通道
  SC_CTOR(TEST) { // s没有名字，用默认值
    SC_THREAD(writer); // 注册一个写进程
    SC_THREAD(reader); // 注册一个读进程
    sensitive << s; // SIGNAL用于静态敏感列表
    dont_initialize();
  }
  void writer() {
    int v = 1;
    while (true) {
      s.write(v++); // 写到通道
      wait(1, SC_SEC);
    }
  }
  void reader() {
    while (true) {
      std::cout << sc_time_stamp() << ": val = " << s.read() << std::endl; // 从通道读
      wait();
    }
  }
};
int sc_main(int, char*[]) {
  TEST test("test"); // 实例化
  sc_start(2, SC_SEC);
  return 0;
}
```

> 0 s: val = 1  
> 1 s: val = 2
