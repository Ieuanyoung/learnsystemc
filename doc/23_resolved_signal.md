# Resolved Signal

resolved signal是类sc_signal_resolved或类sc_signal_rv的对象。它与sc_signal的不同之处在于，resolved signal可由多个进程写入，冲突值在通道内解析。

  1. sc_signal_resolved是从sc_signal类派生出来的预定义原始通道.
  2. sc_signal_rv是从sc_signal类派生出来的预定义原始通道。
    a) sc_signal_rv与sc_signal_resolved相似。
    b) 区别是给模板基类sc_signal传入的类型是sc_dt::sc_lv<W> 而不是sc_dt::sc_logic.

类定义:

  1. class sc_signal_resolved: public sc_signal<sc_dt::sc_logic,SC_MANY_WRITERS>
  2. template <int W> class sc_signal_rv: public sc_signal<sc_dt::sc_lv<W>,SC_MANY_WRITERS>

sc_signal_resolved的解析表:
  | 0 | 1 | Z | X |
0 | 0 | X | 0 | X |
1 | X | 1 | 1 | X |
Z | 0 | 1 | Z | X |
X | X | X | X | X |

简而言之，一个resolved signal通道可以同时被多个进程写入。这与 sc_signal不同，后者在每个delta周期只能由一个进程写入。

```cpp
// Learn with Examples, 2020, MIT license
#include <systemc>
#include <vector> // use c++  vector lib
using namespace sc_core;
using namespace sc_dt; // 定义了sc_logic
using std::vector;

SC_MODULE(RESOLVED_SIGNAL) {
  sc_signal_resolved rv; // 声明一个resolved signal
  vector<sc_logic> levels; // 声明一个可能4个逻辑值的vector
  SC_CTOR(RESOLVED_SIGNAL) : levels(vector<sc_logic>{sc_logic_0, sc_logic_1, sc_logic_Z, sc_logic_X}){ // 初始化vector为4个逻辑值
    SC_THREAD(writer1);
    SC_THREAD(writer2);
    SC_THREAD(consumer);
  }
  void writer1() {
    int idx = 0;
    while (true) {
      rv.write(levels[idx++%4]); // 0,1,Z,X, 0,1,Z,X, 0,1,Z,X, 0,1,Z,X
      wait(1, SC_SEC); // 每隔1秒写
    }
  }
  void writer2() {
    int idx = 0;
    while (true) {
      rv.write(levels[(idx++/4)%4]); // 0,0,0,0, 1,1,1,1, Z,Z,Z,Z, X,X,X,X
      wait(1, SC_SEC); // 每隔1秒写
    }
  }
  void consumer() {
    wait(1, SC_SEC); // 延迟1秒读
    int idx = 0;
    while (true) {
      std::cout << " " << rv.read() << " |"; // 打印读到的值
      if (++idx % 4 == 0) { std::cout << std::endl; } // 每4个值换行
      wait(1, SC_SEC); // 每隔1秒读
    }
  }
};

int sc_main(int, char*[]) {
  RESOLVED_SIGNAL resolved("resolved");
  sc_start(17, SC_SEC); // 运行足够时间测试16种不同的解析组合
  return 0;
}
```

> 0 | X | 0 | X |  // consumer2 == 0, consumer1 = 0, 1, Z, X  
> X | 1 | 1 | X |  // consumer2 == 1, consumer1 = 0, 1, Z, X  
> 0 | 1 | Z | X |  // consumer2 == Z, consumer1 = 0, 1, Z, X  
> X | X | X | X |  // consumer2 == X, consumer1 = 0, 1, Z, X
