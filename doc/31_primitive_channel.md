# Primitive Channel

sc_prim_channel:

  1. 是所有原始通道的基类
  2. 为原始通道提供对调度程序更新阶段的唯一访问权限。
  3. 不包含层次结构、端口或模拟进程。
  4. 与分层通道一样，原始通道可以提供使用接口方法调用范式调用的公共成员函数。
  提供以下成员函数:
      1. request_update(): 用于对通道的更新请求进行排队的调度程序
      2. async_request_update():  
          1) 调度程序以线程安全的方式对通道的更新请求进行排队。可以从systemC内核以外的操作系统线程可靠地调用。
          2) 不建议从systemC内核上下文中执行的函数调用
      3. update():
          1) 在更新阶段由调度程序回调，以响应对request_update或 async_request_update的调用。
          2) 应用程序可以重载此成员函数。sc_prim_channel中此函数的定义本身没有任何作用。
          3) 通常只读取和修改当前对象的数据成员，并创建delta通知。
          4) 不能:  
            a) 调用类sc_prim_channel的任何成员函数，但成员函数update本身（如果在当前对象的基类中被重写）除外  
            b) 调用无参数的sc_event类成员函数notify()创建即时通知  
            c) 调用sc_process_handle类的任何成员函数进行进程控制（例如挂起或杀死进程）  
            d) 更改除当前对象的数据成员之外的任何存储的状态。  
            e) 读取当前对象以外的任何原始通道实例的状态。  
            f) 调用其他通道实例的接口方法。特别是，成员函数update不应写入任何信号。
      4. next_trigger()
      5. wait()

通道应实现一个或多个接口，因此需要从接口类（基类类型为 sc_interface）继承。接口为通道提供所需的方法。

```cpp
// Learn with Examples, 2020, MIT license
#include <systemc>
#include <string>
using namespace sc_core;

class GENERATOR_IF : public sc_interface { // 产生中断的接口
public:
  virtual void notify() = 0;
};
class RECEIVER_IF : public sc_interface { // 接收中断的接口
public:
  virtual const sc_event& default_event() const = 0; // 用于敏感
};
class INTERRUPT : public sc_prim_channel, public GENERATOR_IF, public RECEIVER_IF { // 中断类
public:
  INTERRUPT(sc_module_name name) : sc_prim_channel(name) {} // 构造函数，构造sc_prim_channel
  void notify() { // 实现GENERATOR_IF
    e.notify();
  }
  const sc_event& default_event() const { // 实现RECEIVER_IF
    return e;
  }
private:
  sc_event e; // 用于同步的私有事件
};
SC_MODULE(GENERATOR) { // 中断生成类
  sc_port<GENERATOR_IF> p; // 生成中断的端口
  SC_CTOR(GENERATOR) { // 构造函数
    SC_THREAD(gen_interrupt);
  }
  void gen_interrupt() {
    while (true) {
      p->notify(); // 调用中断通道的通知函数
      wait(1, SC_SEC);
    }
  }
};
SC_MODULE(RECEIVER) { // 中断接收类
  sc_port<RECEIVER_IF> p; // 接收中断的端口
  SC_CTOR(RECEIVER) { // 构造函数
    SC_THREAD(rcv_interrupt);
    sensitive << p; // 监视p端口的中断
    dont_initialize();
  }
  void rcv_interrupt() { // 中断触发
    while (true) {
      std::cout << sc_time_stamp() << ": interrupt received" << std::endl;
      wait();
    }
  }
};

int sc_main(int, char*[]) {
  GENERATOR generator("generator"); // 实例化generator
  RECEIVER receiver("receiver"); // 实例化receiver
  INTERRUPT interrupt("interrupt"); // 实例化interrupt
  generator.p(interrupt); // 绑端口
  receiver.p(interrupt);
  sc_start(2, SC_SEC);
  return 0;
}
```

> 0 s: interrupt received  
> 1 s: interrupt received
