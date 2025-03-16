# Delta Cycle

Delta周期可以被认为是仿真中非常小的时间步长，它不会增加用户可见的时间。由独立的评估和更新阶段组成，在特定的模拟时间内可能会出现多个delta周期。当发生信号值更新时，其他进程直到下一个delta周期才会看到新分配的值。

什么时候用:

  1. notify(SC_ZERO_TIME) 会在下一个delta周期的评估阶段通知事件，这被称为 "delta通知"。
  2. 对request_update()的（直接或间接）调用会导致update()方法在当前delta周期的更新阶段被调用。

```cpp
// Learn with Examples, 2020, MIT license
#include <systemc>
using namespace sc_core;

SC_MODULE(DELTA) {
  int x = 1, y = 1; // 定义两个成员变量
  SC_CTOR(DELTA) {
    SC_THREAD(add_x); // x += 2
    SC_THREAD(multiply_x); // x *= 3
    SC_THREAD(add_y); // y += 2
    SC_THREAD(multiply_y); // y *= 3
  }
  void add_x() { // x += 2 先发生
    std::cout << "add_x: " << x << " + 2" << " = ";
    x += 2;
    std::cout << x << std::endl;
  }
  void multiply_x() { // x *= 3 在一个delta周期后发生
    wait(SC_ZERO_TIME);
    std::cout << "multiply_x: " << x << " * 3" << " = ";
    x *= 3;
    std::cout << x << std::endl;
  }
  void add_y() { // y += 2 在一个delta周期后发生
    wait(SC_ZERO_TIME);
    std::cout << "add_y: " << y << " + 2" << " = ";
    y += 2;
    std::cout << y << std::endl;
  }
  void multiply_y() { // y *=3 先发生
    std::cout << "multiply_y: " << y << " * 3" << " = ";
    y *= 3;
    std::cout << y << std::endl;
  }
};

int sc_main(int, char*[]) {
  DELTA delta("delta");
  sc_start();
  return 0;
}
```

> 结果：  
> add_x: 1 + 2 = 3  
> multiply_y: 1 * 3 = 3  
> add_y: 3 + 2 = 5  
> multiply_x: 3 * 3 = 9  
