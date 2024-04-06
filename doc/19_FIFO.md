# FIFO

sc_fifo:

  1. 是一个预定义的原始通道，用于模拟fifo（即先进先出缓冲区）的行为。
  2. 有一定数量的槽来存储值。在创建对象时确定槽的数量后不可更改。
  3. 实现了sc_fifo_in_if<T>和sc_fifo_out_if<T>两个接口类。

构造函数:

  1. explicit sc_fifo(int size_ = 16): 在初始化列表中调用基类构造函数： sc_prim_channel(sc_gen_unique_name( "fifo" ))
  2. explicit sc_fifo(const char* name_, int size_ = 16): 在初始化列表中调用基类构造函数: sc_prim_channel(name_)
  这两个构造函数都会将 fifo 中的插槽数初始化为参数 size_ 给出的值。插槽数应大于零。

读的成员函数:

  1. void read(T&), T read():
    a) 返回最近写入FIFO的值，并从FIFO中删除该值，使其无法再次读取。
    b) 从FIFO中读取数值的顺序应与向FIFO中写入数值的顺序完全一致。
    c) 在当前delta周期中写入FIFO的值在该delta周期中无法读取，但在紧随其后的delta周期中可以读取。
    d) 如果FIFO为空，则暂停运行，直到收到数据写入。
  2. bool nb_read(T&):
    a), b), c) 和read()一样
    d) 如果FIFO为空，成员函数nb_read立即返回，不修改FIFO的状态，不调用 request_update，返回false。否则，如果有值可供读取，返回true。
  3. operator T(): 等效于"operator T() {return read();}""

写的成员函数:

  1. write(const T&):
    a) 将作为参数传入的值写入FIFO。
    b) 在一个delta周期内可写入多个值。
    c) 如果在当前delta周期内从FIFO中读取了数值，则在紧接着的delta周期之前，FIFO中因此而产生的空槽不会空出来供写入之用。
    d) 如果FIFO已满，暂停运行，直到收到数据读取事件通知。
  2. bool nb_write(const T&):
    a), b), c) 和write()一样。
    d) 如果FIFO已满，立即返回false，不修改FIFO的状态，也不调用request_update。否则返回true。
  3. operator=: 等效于"sc_fifo<T>& operator= (const T& a) {write(a); return *this;}"

事件的成员函数:

  1. sc_event& data_written_event(): 返回对数据写入事件的引用，该事件在向FIFO写入值的delta周期结束时的delta通知阶段发出通知。
  2. sc_event& data_read_event(): 返回对数据读取事件的引用，该事件会在从FIFO读取数值的delta周期结束时的delta通知阶段发出通知。

可用值和可用插槽的成员函数:

  1. int num_available(): 返回当前delta周期中可读取的数值个数。当前delta周期内的读取会导致减小，写不会导致增加。
  2. int num_free(): 返回当前delta周期中可写入的空闲槽位数。计算时应扣除在当前 delta 周期中写入的空闲槽位，但不增加在当前 delta 周期中通过读取获得的空闲槽位。

```cpp
// Learn with Examples, 2020, MIT license
#include <systemc>
using namespace sc_core;

SC_MODULE(FIFO) {
  sc_fifo<int> f1, f2, f3;
  SC_CTOR(FIFO) : f1(2), f2(2), f3(2) { // 容量2的fifo
    SC_THREAD(generator1);
    SC_THREAD(consumer1);

    SC_THREAD(generator2);
    SC_THREAD(consumer2);

    SC_THREAD(generator3);
    SC_THREAD(consumer3);
  }
  void generator1() { // 阻塞写入
    int v = 0;
    while (true) {
      f1.write(v); // 相当于f = v(不推荐)
      std::cout << sc_time_stamp() << ": generator1 writes " << v++ << std::endl;
      wait(1, SC_SEC); // 每隔1秒写
    }
  }
  void consumer1() { // 阻塞读取
    int v = -1;
    while (true) {
      f1.read(v); // same as v = int(f), which is not recommended; or, v = f1.read();
      std::cout << sc_time_stamp() << ": consumer1 reads " << v << std::endl;
      wait(3, SC_SEC); // 每隔3秒读，fifo会被填满
    }
  }
  void generator2() { // 非阻塞写入
    int v = 0;
    while (true) {
      while (f2.nb_write(v) == false ) { // nb write直到成功
        wait(f2.data_read_event()); // 如果不成功就等读发生，这样就有了空槽
      }
      std::cout << sc_time_stamp() << ": generator2 writes " << v++ << std::endl;
      wait(1, SC_SEC); // 每隔1秒写
    }
  }
  void consumer2() { // 非阻塞读取
    int v = -1;
    while (true) {
      while (f2.nb_read(v) == false) {
        wait(f2.data_written_event());
      }
      std::cout << sc_time_stamp() << ": consumer2 reads " << v << std::endl;
      wait(3, SC_SEC); // 每隔3秒读，fifo会被填满
    }
  }
  void generator3() { // 写前后的空槽数
    int v = 0;
    while (true) {
      std::cout << sc_time_stamp() << ": generator3, before write, #free/#available=" << f3.num_free() << "/" << f3.num_available() << std::endl;
      f3.write(v++);
      std::cout << sc_time_stamp() << ": generator3, after write, #free/#available=" << f3.num_free() << "/" << f3.num_available() << std::endl;
      wait(1, SC_SEC);
    }
  }
  void consumer3() { // 读前后的空槽数
    int v = -1;
    while (true) {
      std::cout << sc_time_stamp() << ": consumer3, before read, #free/#available=" << f3.num_free() << "/" << f3.num_available() << std::endl;
      f3.read(v);
      std::cout << sc_time_stamp() << ": consumer3, after read, #free/#available=" << f3.num_free() << "/" << f3.num_available() << std::endl;
      wait(3, SC_SEC); // 每隔3秒读，fifo会被填满
    }
  }
};

int sc_main(int, char*[]) {
  FIFO fifo("fifo");
  sc_start(10, SC_SEC);
  return 0;
}
```
