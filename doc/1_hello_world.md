# hello world

```c++
#include <systemc>
using namespace sc_core; 

void hello1() { // 普通c++函数
  std::cout << "Hello world using approach 1" << std::endl;
}

struct HelloWorld : sc_module { // 定义一个systemC模块
  SC_CTOR(HelloWorld) { // 构造函数
    SC_METHOD(hello2); // 注册一个成员函数
  }
  void hello2(void) { // systemC仿真函数
    std::cout << "Hello world using approach 2" << std::endl;
  }
};

int sc_main(int, char*[]) { // 模拟入口
  hello1(); // 方法1: 手动调用方法
  HelloWorld helloworld("helloworld"); // 方法2：实例化一个systemC模块
  sc_start(); // 让systemC调用方法
  return 0;
}
```
