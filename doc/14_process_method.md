# Process: Method

方法:

  1. 可能有静态敏感
  2. 只有方法过程，可以调用函数next_trigger来创建动态敏感。
  3. 无论方法进程实例的静态敏感或动态敏感如何，进程本身触发的即时通知都不能使自身运行。

next_trigger():

  1. 是sc_module类的成员函数，sc_prim_channel类的成员函数，也是非成员函数。
  2. 可以被调用:
    a) 模块本身的成员函数,
    b) 通道的成员函数, 或
    c) 完全在方法进程中调用的函数。

注意:  
    在方法进程中声明的局部变量都将在从返回时销毁。模块的数据成员与方法进程持久状态一致。

回顾SC_METHOD和SC_THREAD

  1. SC_METHOD(func):没有自己的执行线程，不消耗模拟时间，不能暂停，不能调用调用了wait()的代码
  2. SC_THREAD(func): 有自己的执行线程，可能会消耗模拟时间，可以暂停，可以调用调用了wait()的代码

```cpp
// Learn with Examples, 2020, MIT license
#include <systemc>
using namespace sc_core;

SC_MODULE(PROCESS) {
  SC_CTOR(PROCESS) { // 构造函数
    SC_THREAD(thread); // 注册线程
    SC_METHOD(method); // 注册方法
  }
  void thread() {
    int idx = 0; // 声明一次
    while (true) { // 无限循环
      std::cout << "thread"<< idx++ << " @ " << sc_time_stamp() << std::endl;
      wait(1, SC_SEC); // 每1秒触发
    }
  }
  void method() {
    // 注意这里没有循环
    int idx = 0; // 每次方法调用时声明一次
    std::cout << "method" << idx++ << " @ " << sc_time_stamp() << std::endl;
    next_trigger(1, SC_SEC);
  }
};

int sc_main(int, char*[]) {
  PROCESS process("process");
  sc_start(4, SC_SEC);
  return 0;
}
```

> method0 @ 0 s  
> thread0 @ 0 s  
> method0 @ 1 s  
> thread1 @ 1 s  
> method0 @ 2 s  
> thread2 @ 2 s  
> method0 @ 3 s  
> thread3 @ 3 s  
