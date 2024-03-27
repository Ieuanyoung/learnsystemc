# simulation process

仿真过程：

  1. 是一个模块类的成员函数
  2. 没有输入参数，也不返回值
  3. 提前在仿真内核中注册

如何注册仿真：

  1. SC_METHOD(func): 没有自己的执行线程，不消耗模拟时间，不能暂停，也不能调用调用wait()的代码
  2. SC_THREAD(func):有自己的执行线程，可能会消耗模拟时间，可以被暂停，并且可以调用调用wait()的代码
  3. SC_CTHREAD(func, event): SC_THREAD的一种特殊形式，只能对时钟边沿事件敏感

何时注册：

  1. 在构造函数中
  2. 在模块的before_end_of_elaboration或end_of_elaboration的回调中
  3. 或者在前两种调用的成员函数中

注册：

  1. 注册只能在同一模块的成员函数上进行。
  2. 不能在end_of_elaboration回调中调用SC_CTHREAD。

> 注意：
>
>    1. SC_THREAD可以做SC_METHOD或SC_CHTEAD能做的一切。见示例。
>    2. 为了再次调用SC_THREAD或SC_CTHREAD进程，要有一个while循环，以确保它永远不会退出。
>    3. SC_THREAD进程不必需while循环。next_trigger()可再次调用。
>    4. systemC中的模拟时间并不是程序实际运行的时间。它是一个由模拟内核管理的计数器。稍后解释。

```c++
// Learn with Examples, 2020, MIT license
#include <systemc>
using namespace sc_core;

SC_MODULE(PROCESS) {
  sc_clock clk; // 声明时钟信号
  SC_CTOR(PROCESS) : clk("clk", 1, SC_SEC) { // 实例化周期为1秒的时钟
    SC_METHOD(method); // 注册方法
    SC_THREAD(thread); // 注册线程
    SC_CTHREAD(cthread, clk); // 注册时钟线程
  }
  void method(void) { // 定义方法成员函数
    // no while loop here
    std::cout << "method triggered @ " << sc_time_stamp() << std::endl;
    next_trigger(sc_time(1, SC_SEC)); // trigger after 1 sec
  }
  void thread() { // 定义线程成员函数
    while (true) { // 无限循环确保它永远不会退出
      std::cout << "thread triggered @ " << sc_time_stamp() << std::endl;
      wait(1, SC_SEC); // 等待1秒，然后再次执行
    }
  }
  void cthread() { // 定义时钟线程成员函数
    while (true) { // 无线循环
      std::cout << "cthread triggered @ " << sc_time_stamp() << std::endl;
      wait(); // 等待下一个CLK事件，该事件在 1 秒后出现
    }
  }
};

int sc_main(int, char*[]) {
  PROCESS process("process"); // 初始化模块
  std::cout << "execution phase begins @ " << sc_time_stamp() << std::endl;
  sc_start(2, SC_SEC); // 运行仿真2秒
  std::cout << "execution phase ends @ " << sc_time_stamp() << std::endl;
  return 0;
}
```
