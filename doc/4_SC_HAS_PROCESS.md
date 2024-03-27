# SC_HAS_PROCESS

SC_HAS_PROCESS从systemC v2.0开始引入。它只有一个参数，即模块类的名称。它通常与SC_CTOR相提并论。让我们看看这两个宏是如何定义的：

1. SC_SCOR: `#define SC_CTOR(user_module_name) typedef user_module_name SC_CURRENT_USER_MODULE; user_module_name( ::sc_core::sc_module_name )`
2. SC_HAS_PROCESS: `#define SC_HAS_PROCESS(user_module_name) typedef user_module_name SC_CURRENT_USER_MODULE`

当提供比如“module”作为传入参数给SC_CTOR和SC_HAS_PROCESS, 他们扩展为：

1. SC_CTOR(module): `typedef module SC_CURRENT_USER_MODULE; module( ::sc_core::sc_module_name )`
2. SC_HAS_PROCESS(module): `typedef module SC_CURRENT_USER_MODULE;`

可以看出：

1. 两者都将“module”定义为“SC_CURRENT_USER_MODULE”，用于通过 SC_METHOD/SC_THREAD/SC_CTHREAD将成员函数注册到仿真内核。
2. SC_CTOR还声明了一个默认构造函数，模块名是唯一的输入参数。其影响如下：

    1. SC_CTOR用一行代码实现构造函数，而如果使用SC_HAS_PROCESS，则必须声明构造函数：`module_class_name(sc_module_name name, additional argument ...);`
    2. 由于SC_CTOR有一个构造函数声明，所以只能放在类的头文件中。

我的建议：

1. 如果模块没有仿真进程，则不要使用SC_CTOR或SC_HAS_PROCESS（通过 SC_METHOD/SC_THREAD/SC_CTHREAD向仿真内核注册成员函数）
2. 如果模块的实例化不需要额外参数（模块名除外），则使用SC_CTOR
3. 在实例化过程中需要额外参数时使用SC_HAS_PROCESS

  ```c++
  // Learn with Examples, 2020, MIT license
#include <systemc>
using namespace sc_core;

SC_MODULE(MODULE_A) { // 没有仿真过程的模块不需要SC_CTOR或SC_HAS_PROCESS
  MODULE_A(sc_module_name name) { // c++风格构造函数, 基类使用模块名称隐式实例化
    std::cout << this->name() << ", no SC_CTOR or SC_HAS_PROCESS" << std::endl;
  }
};

SC_MODULE(MODULE_B1) { // 将模块名称作为唯一输入参数的构造函数
  SC_CTOR(MODULE_B1) { // 隐式声明MODULE_B1(sc_module_name)的构造函数
    SC_METHOD(func_b); // 注册成员函数到仿真内核
  }
  void func_b() { // define function
    std::cout << name() << ", SC_CTOR" << std::endl;
  }
};

SC_MODULE(MODULE_B2) { // 将模块名称作为唯一输入参数的构造函数
  SC_HAS_PROCESS(MODULE_B2); // 没有隐式构造函数声明
  MODULE_B2(sc_module_name name) { // 显式构造函数声明，也默认通过sc_module(name)实例化基类
    SC_METHOD(func_b); // 注册成员函数
  }
  void func_b() { // define function
    std::cout << name() << ", SC_HAS_PROCESS" << std::endl;
  }
};

SC_MODULE(MODULE_C) { // 可传额外的输入参数
  const int i;
  SC_HAS_PROCESS(MODULE_C); // 可以使用SC_CTOR，但还将定义一个不用的构造函数：MODULE_A(sc_module_name)
  MODULE_C(sc_module_name name, int i) : i(i) { // 定义构造函数
    SC_METHOD(func_c); // 注册成员函数
  }
  void func_c() { // 定义成员函数
    std::cout << name() << ", additional input argument" << std::endl;
  }
};

SC_MODULE(MODULE_D1) { // 可以在头文件内使用SC_CTOR，构造函数在头文件外定义
  SC_CTOR(MODULE_D1);
  void func_d() {
    std::cout << this->name() << ", SC_CTOR inside header, constructor defined outside header" << std::endl;
  }
};
MODULE_D1::MODULE_D1(sc_module_name name) : sc_module(name) { // 定义构造函数，有没有"sc_module(name)"都可
  SC_METHOD(func_d);
}

SC_MODULE(MODULE_D2) { // 可以在头文件内使用SC_HAS_PROCESS，构造函数在头文件外定义
  SC_HAS_PROCESS(MODULE_D2);
  MODULE_D2(sc_module_name); // 声明构造函数
  void func_d() {
    std::cout << this->name() << ", SC_HAS_PROCESS inside header, constructor defined outside header" << std::endl;
  }
};
MODULE_D2::MODULE_D2(sc_module_name name) : sc_module(name) { // 定义构造函数，有没有"sc_module(name)"都可
  SC_METHOD(func_d);
}

SC_MODULE(MODULE_E) { // 构造函数在头文件外定义
  MODULE_E(sc_module_name name); // c++风格声明构造函数
  void func_e() {
    std::cout << this->name() << ", SC_HAS_PROCESS outside header, CANNOT use SC_CTOR" << std::endl;
  }
};
MODULE_E::MODULE_E(sc_module_name name) { // 构造函数定义
  SC_HAS_PROCESS(MODULE_E); // 用SC_CTOR不合适
  SC_METHOD(func_e);
}

int sc_main(int, char*[]) {
  MODULE_A module_a("module_a");
  MODULE_B1 module_b1("module_b1");
  MODULE_B2 module_b2("module_b2");
  MODULE_C module_c("module_c", 1);
  MODULE_D1 module_d1("module_d1");
  MODULE_D2 module_d2("module_d2");
  MODULE_E module_e("module_e");
  sc_start();
  return 0;
}
```  
