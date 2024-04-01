# Time Notation

先了解一下两种时间测量值的区别：  

  1. 执行时间  
    从开始执行到完成的时间，包括等待其他系统活动和应用程序的时间  
  2. 仿真时间
    模拟仿真所用的时间，可能小于或大于执行时间。

在systemC中，sc_time是仿真内核用于跟踪仿真时间的数据类型。它定义了多个时间单位：SC_SEC, SC_MS, SC_US, SC_NS, SC_PS, SC_FS. 每个后续的时间单位是其前一个的1/1000。

sc_time对象可用作赋值、算术和比较运算的操作数。乘法允许其中一个操作数是double除法允许被除数是double

SC_ZERO_TIME:
表示时间为零的时间值的宏。最好在写入时间为零的时间值时使用此常量，例如，在创建delta通知或delta超时时。

获取当前仿真时间戳使用sc_time_stamp()。

```c++
// Learn with Examples, 2020, MIT license
#include <systemc>
using namespace sc_core;

int sc_main(int, char*[]) {
sc_core::sc_report_handler::set_actions( "/IEEE_Std_1666/  deprecated", sc_core::SC_DO_NOTHING ); // 禁止set_time_resolution导致的警告
  sc_set_time_resolution(1, SC_FS); // 已被弃用，但仍然有用，默认值为1PS
  sc_set_default_time_unit(1, SC_SEC); // 改变时间单位为1秒
  std::cout << "1 SEC =     " << sc_time(1, SC_SEC).to_default_time_units() << " SEC"<< std::endl;
  std::cout << "1  MS = " << sc_time(1, SC_MS).to_default_time_units()  << " SEC"<< std::endl;
  std::cout << "1  US = " << sc_time(1, SC_US).to_default_time_units()  << " SEC"<< std::endl;
  std::cout << "1  NS = " << sc_time(1, SC_NS).to_default_time_units()  << " SEC"<< std::endl;
  std::cout << "1  PS = " << sc_time(1, SC_PS).to_default_time_units()  << " SEC"<< std::endl;
  std::cout << "1  FS = " << sc_time(1, SC_FS).to_default_time_units()  << " SEC"<< std::endl;
  sc_start(7261, SC_SEC); // 模拟运行7261秒
  double t = sc_time_stamp().to_seconds(); // 时间转换为秒
  std::cout << int(t) / 3600 << " hours, " << (int(t) % 3600) / 60 << " minutes, " << (int(t) % 60) << "seconds" << std::endl;
  return 0;
}
```
