# systemC Module

## 是什么

1. 具有状态、行为和结构（支持多层级）的最小功能容器。
2. 一个继承了systemC的sc_module的c++类。
3. SystemC的主要构成组件。
4. 表示真实系统中的一个组件。

## 怎样定义

1. `SC_MODULE(module_name) {}`: 使用systemC定义的宏"SC_MODULE"，和下面2的方法等价。
2. `struct module_name: public sc_module {}`: 定义一个继承sc_module的结构体。
3. `class module_name : public sc_module {}`: 定义一个继承sc_module的类。

## 怎样使用

1. 模块的对象只能在阐述过程中构造，不能在模拟过程中实例化。
2. 每个从sc_module派生（直接或间接）的模块类都应至少有一个构造函数。每个构造函数都应有一个且仅有一个sc_module_name类的参数，其他参数随意。
3. 每个模块的构造函数都应传递一个字符串参数。模块名最好和对象名相同。
4. 模块间通信通常应使用接口方法调用完成，也就是说，模块应通过其端口与环境通信，调试时可以用其他通信机制。

```c++
// Learn with Examples, 2020, MIT license
#include <systemc>
using namespace sc_core;

SC_MODULE(MODULE_A) { // 方法1：用SC_MODULE宏
  SC_CTOR(MODULE_A) { // 默认构造函数
    std::cout << name() << " constructor" << std::endl; // name()返回实例化时传入的对象名
  }
};
struct MODULE_B : public sc_module { // 方法2：使用c++语法，可读性高
  SC_CTOR(MODULE_B) {
    std::cout << name() << " constructor" << std::endl;
  }
};
class MODULE_C : public sc_module { // 方法3：使用类
public: // 必须把构造函数明确声明为public 
  SC_CTOR(MODULE_C) {
    std::cout << name() << " constructor" << std::endl;
  }
};

int sc_main(int, char*[]) { // 模拟入口
  MODULE_A module_a("module_a"); // 声明并实例化module_a, 一般模块名和对象名相同
  MODULE_B module_b("modb"); // 声明并实例化module_b, 模块名也可以和对象名不同
  MODULE_C module_c("module_c"); // 声明并实例化module_c
  sc_start(); // 在本例中可以跳过这一步，因为模块实例化发生在sc_start之前的阐述阶段
  return 0;
}
```
