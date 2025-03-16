# Concurrency

SystemC使用模拟进程来模拟并发性。但不是真正的并发执行。
当模拟多个进程并发运行时，其实在特定时间只执行一个进程。但是，在所有进程完成任务之前，模拟时间保持不变。
因此，这些进程在相同的“模拟时间”上并发运行。这与Go语言不同，Go语言是真正的并发。

示例

```cpp
// Learn with Examples, 2020, MIT license
#include <systemc>
using namespace sc_core;

SC_MODULE(CONCURRENCY) {
  SC_CTOR(CONCURRENCY) { // 构造函数
    SC_THREAD(thread1); // 注册thread1
    SC_THREAD(thread2); // 注册thread2
  }
  void thread1() {
    while(true) { // 无限循环
      std::cout << sc_time_stamp() << ": thread1" << std::endl;
      wait(2, SC_SEC); // 2个仿真秒后再次触发
    }
  }
  void thread2() {
    while(true) {
      std::cout << "\t" << sc_time_stamp() << ": thread2" << std::endl;
      wait(3, SC_SEC);
    }
  }
};

int sc_main(int, char*[]) {
  CONCURRENCY concur("concur"); // 定义对象
  sc_start(10, SC_SEC); // 运行仿真10秒
  return 0;
}
```

> 0 s: thread1  
> 0 s: thread2  
> 2 s: thread1  
> 3 s: thread2  
> 4 s: thread1  
> 6 s: thread2  
> 6 s: thread1  
> 8 s: thread1  
> 9 s: thread2  
