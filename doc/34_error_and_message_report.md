# Error and Message Report

sc_report:

  1. 表示由函数sc_report_handler::report生成的报告实例。
  2. 如果为给定的严重性级别和消息类型设置了操作SC_CACHE_REPORT，则应用程序可访问。
  3. 当报告处理程序抛出时，可能会被应用程序捕获。

sc_report_handler:
  提供用于编写有关发生异常情况的文本报告的功能，以及用于定义在生成这些报告时要执行的特定于应用程序的行为的功能。

sc_severity表示报告的严重性级别:

  1. enum sc_severity {SC_INFO = 0, SC_WARNING, SC_ERROR, SC_FATAL, SC_MAX_SEVERITY};
  2. 有四个严重性级别。SC_MAX_SEVERITY不是严重性级别。将 SC_MAX_SEVERITY传递给需要sc_severity类型参数的函数是错误的。

sc_verbosity提供了指示性详细级别的值，这些值可作为参数传递给类 sc_report_handler的成员函数set_verbosity_level和类 sc_report_handler的成员函数report：
  enum sc_verbosity {SC_NONE = 0, SC_LOW = 100, SC_MEDIUM = 200, SC_HIGH = 300, SC_FULL = 400, SC_DEBUG = 500};

sc_actions表示一个字，其中每个bit代表一个不同的动作。可以设置多个bit，在这种情况下，应执行所有相应的操作：

  1. enum {
     SC_UNSPECIFIED  = 0x0000, // 不是操作，作为默认值，表示未设置任何操作。
     SC_DO_NOTHING   = 0x0001, // 是一个指定动作
     SC_THROW        = 0x0002,
     SC_LOG          = 0x0004,
     SC_DISPLAY      = 0x0008,
     SC_CACHE_REPORT = 0x0010,
     SC_INTERRUPT    = 0x0020,
     SC_STOP         = 0x0040,
     SC_ABORT        = 0x0080
    }
  2. 每个严重性级别都与一组默认操作相关联，可以通过调用成员函数set_actions来覆盖这些操作。
  3, 默认动作:
    a) #define SC_DEFAULT_INFO_ACTIONS ( SC_LOG | SC_DISPLAY )
    b) #define SC_DEFAULT_WARNING_ACTIONS ( SC_LOG | SC_DISPLAY )
    c) #define SC_DEFAULT_ERROR_ACTIONS ( SC_LOG | SC_CACHE_REPORT | SC_THROW )
    d) #define SC_DEFAULT_FATAL_ACTIONS ( SC_LOG | SC_DISPLAY | SC_CACHE_REPORT | SC_ABORT )

void report(sc_severity, const char* msg_type, const char* msg, [int verbosity], const char* file, int line) 生成报告并采取适当的操作：

  1. 使用第一个参数传递的严重性和第二个参数传递的消息类型来确定先前调用函数set_actions、stop_after、suppress和force要执行的操作集。
  2. 使用所有五个参数值创建一个sc_report类的对象，并将此对象传递给成员函数set_handler设置的处理程序。
  3. 除非设置了SC_CACHE_REPORT操作，否则在调用成员函数report之后，该对象不会继续存在，在这种情况下，可以通过调用函数get_cached_reports来检索该对象。
  4. 负责确定要执行的一组操作。由函数set_handler设置的处理程序函数负责执行这些操作。
  5. 对生成的报告数量进行计数。无论是否执行了操作，这些计数都应递增，除非报告因其冗长程度而被忽略，在这种情况下，计数不应递增。

set_actions():

  1. 设置使用给定的严重性级别和/或消息类型调用report时要执行的操作。
  2. 替换上一个为给定的严重性、消息类型或严重性-消息类型设定的操作。

stop_after(): 当针对给定严重性级别、消息类型或严重性-消息类型对生成的报告数量正好达到函数stop_after的参数limit所给定的数量时，report应调用 sc_stop。

get_count(): 返回成员函数 report 维护的每个严重性级别、每个消息类型和每个严重性-消息类型对生成的报告数。

详细程度:

  1. int set_verbosity_level(int): 将最大详细级别作为参数传递，返回最大详细级别的上一个值。
  2. int get_verbosity_level(): 返回最大详细级别值。

```cpp
// Learn with Examples, 2020, MIT license
#include <systemc>
using namespace sc_core;

SC_MODULE(MODULE) { // 测试模块
  sc_port<sc_signal<int>> p;
  SC_CTOR(MODULE) {
    SC_REPORT_WARNING("ctor", "register function"); // 给"ctor"生成报告
    SC_THREAD(writer); // 写进程
    SC_THREAD(reader); // 读进程
    sensitive << p; // 对p敏感
    dont_initialize();
  }
  void writer() {
    int v = 1;
    while (true) {
      SC_REPORT_INFO("writer", ("write " + std::to_string(v)).c_str()); // 给"writer"生成报告
      p->write(v++); // 通过端口写到通道
      wait(1, SC_SEC); // 每隔1秒写
    }
  }
  void reader() {
    while (true) {
      SC_REPORT_INFO("reader", ("read " + std::to_string(p->read())).c_str()); // 给"reader"生成报告
      wait();
    }
  }
};
int sc_main(int, char*[]) {
  sc_report_handler::set_log_file_name("report.log"); // 初始化报告
  sc_report_handler::set_actions("writer", SC_INFO, SC_LOG); // "writer"的INFO日志存在文件中不展示

  MODULE module("module");
  sc_signal<int> s;
  module.p(s);

  SC_REPORT_INFO("main", "simulation starts"); // 给"main"生成报告
  sc_start(2, SC_SEC);
  SC_REPORT_INFO("main", "simulation ends"); // 给"main"生成报告
  return 0;
}
```

> Warning: ctor: register function  
> Info: main: simulation starts  
> Info: reader: read 1  
> Info: reader: read 2  
> Info: main: simulation ends
