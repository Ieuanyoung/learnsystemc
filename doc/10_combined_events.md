# Combined Events

支持以下格式的wait()

  1. wait(): 等待敏感列表中的事件 （SystemC 1.0）。
  2. wait(e1): 等待事件e1.
  3. wait(e1 | e2 | e3): 等待事件e1或e2或e3。
  4. wait(e1 & e2 & e3): 等待事件e1,e2,和e3。
  5. wait(200, SC_NS): 等待200 ns。
  6. wait(200, SC_NS, e1): 等待事件e1, 200ns后超时。
  7. wait(200, SC_NS, e1 | e2 | e3): 等待事件e1,e2或e3, 200ns后超时。
  8. wait(200, SC_NS, e1 & e2 & e3): 等待事件e1,e2和e3, 200ns后超时。
  9. wait(sc_time(200, SC_NS)): 同5。
  10. wait(sc_time(200, SC_NS), e1):同6。
  11. wait(sc_time(200, SC_NS), e1 | e2 | e3): 同7。
  12. wait(sc_time(200, SC_NS), e1 & e2 & e3): 同8。
  13. wait(200): 等200个时钟, 只能用于SC_CTHREAD(SystemC 1.0)。
  14. wait(0, SC_NS): 等一个delta周期。
  15. wait(SC_ZERO_TIME): 等一个delta周期。

Note:
  SystemC 2.0 不支持混合使用“|”运算符和“&”运算符。

```cpp
// Learn with Examples, 2020, MIT license
#include <systemc>
using namespace sc_core;

SC_MODULE(COMBINED) {
  sc_event e1, e2, e3, e4, e5, e6, e7, e8, e9, e10; // 声明多个事件
  SC_CTOR(COMBINED) {
    SC_THREAD(trigger); // 注册触发函数
    SC_THREAD(catcher_0); // 注册捕获函数
    SC_THREAD(catcher_1);
    SC_THREAD(catcher_2and3);
    SC_THREAD(catcher_4or5);
    SC_THREAD(catcher_timeout_or_6);
    SC_THREAD(catcher_timeout_or_7or8);
    SC_THREAD(catcher_timeout_or_9and10);
  }
  void trigger(void) {
    e1.notify(1, SC_SEC);  // 事件e1在1秒时通知
    e2.notify(2, SC_SEC);  // ...
    e3.notify(3, SC_SEC);
    e4.notify(4, SC_SEC);
    e5.notify(5, SC_SEC);
    e6.notify(6, SC_SEC);
    e7.notify(7, SC_SEC);
    e8.notify(8, SC_SEC);
    e9.notify(9, SC_SEC);
    e10.notify(10, SC_SEC); // 事件e10在10秒时通知
  }
  void catcher_0(void) {
    wait(2, SC_SEC); // 时间触发
    std::cout << sc_time_stamp() << ": 2sec timeout" << std::endl;
  }
  void catcher_1(void) {
    wait(e1); // 事件e1触发
    std::cout << sc_time_stamp() << ": catch e1" << std::endl;
  }
  void catcher_2and3(void) {
    wait(e2 & e3); // 事件e2和e3触发
    std::cout << sc_time_stamp() << ": catch e2 and e3" << std::endl;
  }
  void catcher_4or5(void) {
    wait(e4 | e5); // 事件e4或e5触发
    std::cout << sc_time_stamp() << ": catch e4 or e5" << std::endl;
  }
  void catcher_timeout_or_6(void) {
    wait(sc_time(5, SC_SEC), e6); // 时间或事件e6触发
    std::cout << sc_time_stamp() << ": 5sec timeout or catch e6"<< std::endl;
  }
  void catcher_timeout_or_7or8(void) {
    wait(sc_time(20, SC_SEC), e7 | e8); // 时间或事件e7/e8触发
    std::cout << sc_time_stamp() << ": 20sec timeout or catch e7 or e8" << std::endl;
  }
  void catcher_timeout_or_9and10(void) {
    wait(sc_time(20, SC_SEC), e9 & e10); // 时间或事件e9和e10触发
    std::cout << sc_time_stamp() << ": 20sec timeout or catch (e9 and e10)" << std::endl;
  }
};

int sc_main(int, char*[]) {
  COMBINED combined("combined");
  sc_start();
  return 0;
}

```
