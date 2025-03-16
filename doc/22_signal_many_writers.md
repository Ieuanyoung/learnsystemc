# Signal: many writers

sc_signal的类定义:
  template <class T, sc_writer_policy WRITER_POLICY = SC_ONE_WRITER> class sc_signal: public sc_signal_inout_if<T>, public sc_prim_channel {}
  
1. 如果WRITER_POLICY == SC_ONE_WRITER，则在仿真过程中的任何时候都不允许多个过程实例写入该信号。
2. 如果WRITER_POLICY == SC_MANY_WRITERS:
  a) 任何进程实例都不能在评估阶段写入信号，
  b) 但不同的进程实例可以在不同的delta周期内写入信号。

因此，默认情况下，一个sc_signal只有一个写入器；当声明为MANY_WRITERS时，写入器可以在不同时间写入信号。

至于消费者，一个sc_signal可能有多个消费者。它们可以在同一时间或不同时间从信号通道读取数据。

```cpp
// Learn with Examples, 2020, MIT license
#include <systemc>
using namespace sc_core;

SC_MODULE(MULTI) {
  sc_signal<int> s1; // 单个写入的信号
  sc_signal<int, SC_MANY_WRITERS> s2; // 多个写入的信号
  SC_CTOR(MULTI) {
    SC_THREAD(writer1); // 写入s1和s2
    SC_THREAD(writer2); // 写入s2
    SC_THREAD(consumer1);
    sensitive << s1; // 对s1敏感
    dont_initialize();
    SC_THREAD(consumer2);
    sensitive << s1 << s2; // 对s1和s2敏感
    dont_initialize();
  }
  void writer1() {
    int v = 1; // 初始化值
    while (true) {
      s1.write(v); // 写入s1
      s2.write(v); // 写入s2
      std::cout << sc_time_stamp() << ": writer1 writes " << v++ << std::endl;
      wait(1, SC_SEC); // 每隔1秒写
    }
  }
  void writer2() {
    int v = -1; // 初始化值
    while (true) {
      // s1.write(v); /* 不可以, 不然会报错runtime error: (E115) sc_signal<T> cannot have more than one driver(不能多个写入)*/
      wait(SC_ZERO_TIME); // 需要避开写入时间，不然会报错runtime error: conflicting write in delta cycle 0(在delta周期同时写冲突)
      s2.write(v); // 写入s2
      std::cout << sc_time_stamp() << ": writer2 writes " << v-- << std::endl;
      wait(1, SC_SEC); // 每隔1秒写
    }
  }
  void consumer1() {
    while (true) {
      std::cout << sc_time_stamp() << ": consumer1 reads s1=" << s1.read() << "; s2=" << s2.read() << std::endl; // 读s1和s2
      wait(); // wait for s1
    }
  }
  void consumer2() {
    while (true) {
      std::cout << sc_time_stamp() << ": consumer2 reads s1=" << s1.read() << "; s2=" << s2.read() << std::endl; // 读s1和s2
      wait(); // 等s1或s2
    }
  }
};

int sc_main(int, char*[]) {
  MULTI consumers("consumers");
  sc_start(2, SC_SEC); // 运行仿真2秒
  return 0;
}
```

> 0 s: writer1 writes 1  
> 0 s: consumer2 reads s1=1; s2=1  
> 0 s: consumer1 reads s1=1; s2=1  
> 0 s: writer2 writes -1  
> 0 s: consumer2 reads s1=1; s2=-1  
> 1 s: writer1 writes 2  
> 1 s: consumer2 reads s1=2; s2=2  
> 1 s: consumer1 reads s1=2; s2=2  
> 1 s: writer2 writes -2  
> 1 s: consumer2 reads s1=2; s2=-2
