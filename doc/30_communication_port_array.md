# Communication: port array

声明端口时:

  1. 第一个参数是接口的名称，也是端口的类型：
    一个端口只能绑定到从端口类型派生的通道，或者绑定到另一个端口，或者使用从端口类型派生的出口类型。
  2. 第二个参数是一个可选的整数值，用于指定端口实例可绑定的通道实例的最大数量：
    a) 默认值为1。
    b) 如果值为0，则端口可绑定任意数量的通道实例。
    c) 将端口绑定到超过允许数量的通道实例是错误的。
  3. 第三个参数是sc_port_policy类型的可选端口策略，用于确定绑定多端口的规则和未绑定端口的规则：
    a) [default] SC_ONE_OR_MORE_BOUND: 端口应绑定到一个或多个通道，最大数量由第二个参数的值确定。端口在阐述结束时保持未绑定是错误的。
    b) SC_ZERO_OR_MORE_BOUND: 端口应绑定到零个或多个通道，最大数目由第二个参数的值决定。在阐述结束时，端口可以保持未绑定状态。
    c) SC_ALL_BOUND: 端口应与第二个参数的值给出的通道实例数完全相同，不多也不少，前提是该值大于零。
      1) 如果第二个参数的值为零，则策略SC_ALL_BOUND与策略 SC_ONE_OR_MORE_BOUND具有相同的含义。
      2) 如果端口在阐述结束时仍未绑定，或绑定的通道数少于第二个参数要求的通道数，则属于错误。

将指定端口与指定通道绑定多次是错误的，无论是直接绑定还是通过其他端口绑定。

定义端口数组的另一种方法是使用 C/C++ 数组语法: sc_port<IF> p[10] or vector<sc_port<IF>> p(10);

Example:

  1. sc_port<IF>                         // 与1个通道实例绑定
  2. sc_port<IF,0>                       // 与1个或多个通道实例绑定，无上限
  3. sc_port<IF,3>                       // 与1或2或3个通道实例绑定
  4. sc_port<IF,0,SC_ZERO_OR_MORE_BOUND> // 与0个或多个通道实例绑定，无上限
  5. sc_port<IF,1,SC_ZERO_OR_MORE_BOUND> // 与0或1个通道实例绑定
  6. sc_port<IF,3,SC_ZERO_OR_MORE_BOUND> // 与0或1或2或3个通道实例绑定
  7. sc_port<IF,3,SC_ALL_BOUND>          // 与3个通道实例绑定
  8. sc_port<IF, 3>                      // 一个包含3个端口的数组，每个端口绑定1个通道实例
  9. vector<sc_port<IF>> p(3)            // 一个包含3个端口的数组，每个端口绑定1个通道实例

```cpp
// Learn with Examples, 2020, MIT license
#include <systemc>
#include <vector> // 用于定义端口数组
using namespace sc_core;

SC_MODULE(WRITER) {
  sc_port<sc_signal_out_if<int>> p1; // #1: 与1个通道实例绑定
  sc_port<sc_signal_out_if<int>, 0> p2; // #2: 与1个或多个通道实例绑定，无上限
  sc_port<sc_signal_out_if<int>, 3> p3; // #3: 与1或2或3个通道实例绑定
  sc_port<sc_signal_out_if<int>, 0, SC_ZERO_OR_MORE_BOUND> p4; // #4: 与0个或多个通道实例绑定，无上限
  sc_port<sc_signal_out_if<int>, 1, SC_ZERO_OR_MORE_BOUND> p5; // #5: 与0或1个通道实例绑定
  sc_port<sc_signal_out_if<int>, 3, SC_ZERO_OR_MORE_BOUND> p6; // #6: 与0或1或2或3个通道实例绑定
  sc_port<sc_signal_out_if<int>, 3, SC_ALL_BOUND> p7; // #7: 与3个通道实例绑定
  std::vector<sc_port<sc_signal_out_if<int>>> p9; // #9: 端口数组，每个端口绑定1个通道实例
  SC_CTOR(WRITER) : p9(3) { // 初始化p9大小为3
    SC_THREAD(writer);
  }
  void writer() {
    int v = 1;
    while (true) {
      p9[0]->write(v); // 写到p9[0]
      p7[1]->write(v++); // 写到p7[1]
      wait(1, SC_SEC);
    }
  }
};
SC_MODULE(READER) {
  sc_port<sc_signal_in_if<int>> p1; // #1
  sc_port<sc_signal_in_if<int>, 0> p2; // #2
  sc_port<sc_signal_in_if<int>, 3> p3; // #3
  sc_port<sc_signal_in_if<int>, 0, SC_ZERO_OR_MORE_BOUND> p4; // #4
  sc_port<sc_signal_in_if<int>, 1, SC_ZERO_OR_MORE_BOUND> p5; // #5
  sc_port<sc_signal_in_if<int>, 3, SC_ZERO_OR_MORE_BOUND> p6; // #6
  sc_port<sc_signal_in_if<int>, 3, SC_ALL_BOUND> p7; // #7
  std::vector<sc_port<sc_signal_in_if<int>>> p9; // #9
  SC_CTOR(READER) : p9(3) { 
    SC_THREAD(reader7);
    sensitive << p7; // 对p7的所有成员敏感
    dont_initialize();
    SC_THREAD(reader9);
    sensitive << p9[0] << p9[1] << p9[2]; // 对p9的所有成员敏感
    dont_initialize();
  }
  void reader7() {
    while (true) {
      std::cout << sc_time_stamp() << "; reader7, port 0/1/2 = " << p7[0]->read() << "/" << p7[1]->read() << "/" << p7[2]->read() << std::endl;
      wait();
    }
  }
  void reader9() {
    while (true) {
      std::cout << sc_time_stamp() << "; reader9, port 0/1/2 = " << p9[0]->read() << "/" << p9[1]->read() << "/" << p9[2]->read() << std::endl;
      wait();
    }
  }
};

int sc_main(int, char*[]) {
  WRITER writer("writer"); // 实例化writer
  READER reader("reader"); // 实例化reader
  // declare channels
  sc_signal<int> s1; // 1个通道
  std::vector<sc_signal<int>> s2(10); // 10个通道
  std::vector<sc_signal<int>> s3(2); // 2个通道
  // s4不绑
  sc_signal<int> s5; // 1个通道
  std::vector<sc_signal<int>> s6(2); // 2个通道
  std::vector<sc_signal<int>> s7(3); // 3个通道
  // #8 is same as #9, omitted
  std::vector<sc_signal<int>> s9(3); // 3个通道
  // 绑端口
  writer.p1(s1); // #1
  reader.p1(s1); // #1
  for (unsigned int i = 0; i < s2.size(); ++i) { // #2
    writer.p2(s2[i]);
    reader.p2(s2[i]);
  }
  for (unsigned int i = 0; i < s3.size(); ++i) { // #3
    writer.p3(s3[i]);
    reader.p3(s3[i]);
  }
  // s4 un-bound
  writer.p5(s5); // #5
  reader.p5(s5); // #5
  for (unsigned int i = 0; i < s6.size(); ++i) { // #6
    writer.p6(s6[i]);
    reader.p6(s6[i]);
  }
  for (unsigned int i = 0; i < s7.size(); ++i) { // #7
    writer.p7(s7[i]);
    reader.p7(s7[i]);
  }
  for (unsigned int i = 0; i < s9.size(); ++i) { // #9
    writer.p9[i](s9[i]);
    reader.p9[i](s9[i]);
  }
  sc_start(2, SC_SEC);
  return 0;
}
```
