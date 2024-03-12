# 构造函数：SC_CTOR

每个c++类都必须有一个构造函数，如果没有明确提供，则会自动生成一个默认构造函数。而每个systemC模块都必须有一个在实例化时提供的唯一的"名称"，这就需要一个至少有一个参数的构造函数。

SystemC提供了一个宏（SC_CTOR），以方便声明或定义模块的构造函数。 SC_CTOR:

1. 只能在C++规则允许声明构造函数的情况下使用，并可用作构造函数声明或构造函数定义的声明符。
2. 只有一个参数，即正在构建的模块类的名称。
3. 不能在构造函数中添加用户定义的参数。如果应用程序需要传递附加参数，则应明确提供构造函数。  

```c++
// Learn with Examples, 2020, MIT license
#include <systemc>
using namespace sc_core;

SC_MODULE(MODULE_A) {
  SC_CTOR(MODULE_A) { // constructor taking only module name
    SC_METHOD(func_a); // register member function to systemC simulation kernel, to be explained later.
  }
  void func_a() { // a member function with no input, no output
    std::cout << name() << std::endl;
  }
};

SC_MODULE(MODULE_B) {
  SC_CTOR(MODULE_B) { // constructor
    SC_METHOD(func_b); // register member function
  }
  void func_b(); // declare function
};
void MODULE_B::func_b() { // define function outside class definition
  std::cout << this->name() << std::endl;
}
SC_MODULE(MODULE_C) { // constructor taking more arguments
  const int i;
  SC_CTOR(MODULE_C); // SC_HAS_PROCESS is recommended, see next example for details
  MODULE_C(sc_module_name name, int i) : sc_module(name), i(i) { // explcit constructor
    SC_METHOD(func_c);
  }
  void func_c() {
    std::cout << name() << ", i = " << i << std::endl;
  }
};

int sc_main(int, char*[]) {
  MODULE_A module_a("module_a");
  MODULE_B module_b("module_b");
  MODULE_C module_c("module_c",1);
  sc_start();
  return 0;
}
```