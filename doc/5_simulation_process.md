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
  sc_clock clk; // declares a clock
  SC_CTOR(PROCESS) : clk("clk", 1, SC_SEC) { // instantiate a clock with 1sec periodicity
    SC_METHOD(method); // register a method
    SC_THREAD(thread); // register a thread
    SC_CTHREAD(cthread, clk); // register a clocked thread
  }
  void method(void) { // define the method member function
    // no while loop here
    std::cout << "method triggered @ " << sc_time_stamp() << std::endl;
    next_trigger(sc_time(1, SC_SEC)); // trigger after 1 sec
  }
  void thread() { // define the thread member function
    while (true) { // infinite loop make sure it never exits 
      std::cout << "thread triggered @ " << sc_time_stamp() << std::endl;
      wait(1, SC_SEC); // wait 1 sec before execute again
    }
  }
  void cthread() { // define the cthread member function
    while (true) { // infinite loop
      std::cout << "cthread triggered @ " << sc_time_stamp() << std::endl;
      wait(); // wait for next clk event, which comes after 1 sec
    }
  }
};

int sc_main(int, char*[]) {
  PROCESS process("process"); // init module
  std::cout << "execution phase begins @ " << sc_time_stamp() << std::endl;
  sc_start(2, SC_SEC); // run simulation for 2 second
  std::cout << "execution phase ends @ " << sc_time_stamp() << std::endl;
  return 0;
}
```
