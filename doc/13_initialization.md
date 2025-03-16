# Initialization

初始化是执行阶段的一部分，发生在sc_start()之后。在初始化过程中，它会按指定顺序执行以下三个步骤：

  1) 运行更新阶段，但不执行到delta通知阶段。
  2) 将对象层次结构中的每个方法和进程实例添加到可运行进程集合中，不包括调用了dont_initialize函数的进程实例，以及时钟线程进程。
  3) 运行delta通知阶段。delta通知阶段结束后，进入评估阶段。

注意:

  1. 更新和delta通知阶段是必要的，因为在阐述过程中可以为原始通道设置初始值，例如从sc_inout类的初始化函数中设置初始值。
  2. 在SystemC 1.0中,  
    a) 在模拟的初始化阶段不执行线程进程。  
    b) 如果方法进程对输入信号/端口敏感，则会在模拟的初始化阶段执行。
  3. SystemC 2.0 调度程序将在仿真的初始化阶段执行所有线程进程和所有方法进程。
    如果线程进程的行为在 SystemC 1.0 和 SystemC 2.0 之间有所不同，请在线程进程的无限循环之前插入一条 wait() 语句。
  4. 在初始化阶段，进程（SystemC 1.0的SC_METHODs;SystemC2.0的SC_METHODs和 SC_THREADs）以未指定的顺序执行。
  5. dont_initialize(): 告诉调度程序不要初始化最后声明的进程方法。

```cpp
// Learn with Examples, 2020, MIT license
#include <systemc>
using namespace sc_core;

SC_MODULE(INITIALIZATION) {
  sc_event e; // 用于进程内部触发的事件
  SC_CTOR(INITIALIZATION) {
    SC_THREAD(trigger); // 无静态敏感
    SC_THREAD(catcher_1); // 无静态敏感
    SC_THREAD(catcher_2); // 无静态敏感
    SC_THREAD(catcher_3);
    sensitive << e; // 对事件e静态敏感
    dont_initialize(); // 不要初始化
  }
  void trigger() {
    while (true) { // 在1, 3, 5, 7 ...秒触发事件e
      e.notify(1, SC_SEC); // 1秒后通知
      wait(2, SC_SEC); // 每两秒触发1次
    }
  }
  void catcher_1() {
    while (true) {
      std::cout << sc_time_stamp() << ": catcher_1 triggered" << std::endl;
      wait(e); // 动态敏感
    }
  }
  void catcher_2() {
    wait(e); // 不初始化 --- 模仿systemC 1.0行为
    while (true) {
      std::cout << sc_time_stamp() << ": catcher_2 triggered" << std::endl;
      wait(e); // 动态敏感
    }
  }
  void catcher_3() { // 用dont_initialize()避免初始化
    while (true) {
      std::cout << sc_time_stamp() << ": catcher_3 triggered" << std::endl;
      wait(e); // 动态敏感
    }
  }
};

int sc_main(int, char*[]) {
  INITIALIZATION init("init");
  sc_start(4, SC_SEC);
  return 0;
}
```

> 结果：  
> 0 s: catcher_1 triggered  
> 1 s: catcher_3 triggered  
> 1 s: catcher_1 triggered  
> 1 s: catcher_2 triggered  
> 3 s: catcher_3 triggered  
> 3 s: catcher_2 triggered  
> 3 s: catcher_1 triggered
