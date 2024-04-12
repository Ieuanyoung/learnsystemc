# Buffer

sc_buffer是从类sc_signal派生的预定义原始通道。
它与sc_signal类的不同之处在于，只要写入buffer，就会通知值变化事件，而不是仅在信号值发生变化时才通知。
例如：
如果当前的"signal"值为1: 写入1不会触发值改变事件
如果当前的"buffer"值为1: 写入1会触发值改变事件

```cpp
// Learn with Examples, 2020, MIT license
#include <systemc>
using namespace sc_core;

SC_MODULE(BUFFER) {
  sc_signal<int> s; // 声明一个signal
  sc_buffer<int> b; // 声明一个buffer
  SC_CTOR(BUFFER) {
    SC_THREAD(writer); // 给signal和buffer一起写入值
    SC_THREAD(consumer1);
    sensitive << s; // signal触发
    dont_initialize();
    SC_THREAD(consumer2);
    sensitive << b; // buffer触发
    dont_initialize();
  }
  void writer() {
    int val = 1; // 初始化值
    while (true) {
      for (int i = 0; i < 2; ++i) { // 同样的值写入两次
        s.write(val); // 写入signal
        b.write(val); // 写入buffer
        wait(1, SC_SEC); // 等1秒
      }
      val++; // 值改变
    }
  }
  void consumer1() {
    while (true) {
      std::cout << sc_time_stamp() << ": consumer1 receives " << s.read() << std::endl;
      wait(); // 从signal接收
    }
  }
  void consumer2() {
    while (true) {
      std::cout << sc_time_stamp() << ": consumer2 receives " << b.read() << std::endl;
      wait(); // 从bugger接收
    }
  }
};

int sc_main(int, char*[]) {
  BUFFER buffer("buffer");
  sc_start(4, SC_SEC);
  return 0;
}
```
