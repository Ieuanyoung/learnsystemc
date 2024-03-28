# Simulation Stages

## 三个阶段

  1. 阐述: 在sc_start()之前执行的语句  
    主要目的是创建内部数据结构，以支持模拟语义。创建模块层次结构的各个部分（模块、端口、通道和进程），并将端口和通道绑定。
  2. 执行: 细分为2个阶段  
    1. 初始化  
      仿真内核会识别所有仿真进程，并将它们放入可运行进程集或等待进程集合。除了那些要求 "不初始化"的进程外，所有模拟进程都在可运行进程集中。  
    2. 模拟  
      通常被描述为一种状态机，用于调度进程运行并推进模拟时间。它有两个内部阶段:  
      1. 评估:逐个运行所有可运行的进程。每个进程运行至wait()或return。如果没有可运行进程，则停止。  
      2. 推进时间: 一旦可运行进程集清空，模拟就会进入推进时间阶段。a)将模拟时间移至与预定事件最接近的时间;b)将等待该特定时间的进程移入可运行集;c)回到评估阶段。  
      以上循环一直持续到发生以下三种情况之一:所有进程均已返回结果/一个进程执行了sc_stop()/达到最大运行时间。然后进入清理阶段。
  3. 清理或后处理：销毁对象、释放内存、关闭打开的文件等。

内核在阐述和模拟的不同阶段会调用四个回调函数

  1. virtual void before_end_of_elaboration():在构建模块层次结构后调用。
  2. virtual void end_of_elaboration():在对before_end_of_elaboration的所有回调完成后，以及这些回调执行的任何实例化或端口绑定完成后，在开始模拟之前调用。
  3. virtual void start_of_simulation():  
    a) 在程序首次调用sc_start时立即调用，如果仿真是在内核的直接控制下启动的，则在仿真开始时调用。  
    b) 如果程序多次调用sc_start，则在第一次调用时调用。  
    c) 在对end_of_elaboration的回调之后和调用调度程序的初始化阶段之前调用。
  4. virtual void end_of_simulation():  
    a) 在调度程序因sc_stop而停止时调用，如果仿真是在内核的直接控制下启动的，则在仿真结束时调用。  
    b) 只调用一次，即使sc_stop被多次调用。

```c++
// Learn with Examples, 2020, MIT license
#include <systemc>
using namespace sc_core;

SC_MODULE(STAGE) {
  SC_CTOR(STAGE) { // 阐述
    std::cout << sc_time_stamp() << ": Elaboration: constructor" << std::endl;
    SC_THREAD(thread); // 初始化+仿真
  }
  ~STAGE() { // 清理
    std::cout << sc_time_stamp() << ": Cleanup: desctructor" << std::endl;
  }
  void thread() {
    std::cout << sc_time_stamp() << ": Execution.initialization" << std::endl;
    int i = 0;
    while(true) {
      wait(1, SC_SEC); // 推进时间
      std::cout << sc_time_stamp() << ": Execution.simulation" << std::endl; // 评估
      if (++i >= 2) {
        sc_stop(); // 两次迭代后停止仿真
      }
    }
  }
  void before_end_of_elaboration() {
    std::cout << "before end of elaboration" << std::endl;
  }
  void end_of_elaboration() {
    std::cout << "end of elaboration" << std::endl;
  }
  void start_of_simulation() {
    std::cout << "start of simulation" << std::endl;
  }
  void end_of_simulation() {
    std::cout << "end of simulation" << std::endl;
  }
};

int sc_main(int, char*[]) {
  STAGE stage("stage"); // 阐述
  sc_start(); // 执行到sc_stop
  return 0; // 清理
}
```
