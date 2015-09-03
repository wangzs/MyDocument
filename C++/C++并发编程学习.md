# C++并发编程学习
## 一、线程的基本概念
#### 1.1 简单使用的两种形式
```code
// 函数对象的形式
class ClassThreadSample {
public:
	void operator()() const {
		std::cout << "This is class operator for thread" << std::endl;
	}
};
ClassThreadSample cts;
std::thread t(cts);
t.join();

// 函数形式
std::thread tt([]() {std::cout << "Lambda function" << std::endl; });
tt.join();
```

#### 1.2 参数传递问题
```c++
void update_reference(std::string &ref) {
	ref += " - changed";
}

std::string param("test");
std::thread a(update_reference, param);
a.join();
std::cout << param << std::endl;
```
上面的结果是前后param的值没有变，因为thread并不知道函数期待的参数类型，只是拷贝提供的变量到线程的内存空间，所以新线程中的ref实际是线程的内存空间中复制后的参数的引用。
* 若要以引用的形式传递参数给thread中的函数，需要使用`std::ref()`
```c++
std::string param("test");
// 此时update_reference函数中使用的参数才是param的引用
std::thread a(update_reference, std::ref(param));
a.join();
std::cout << param << std::endl;
```
* 如果线程使用的是成员函数指针，则第一个参数需要是相应的对象指针
```c++
Class TT {
public:
	void do_some_thing();
};
TT ta;
std::thread(&TT::do_some_thing, &ta);
// 如果成员函数有参数，则在第一个对象指针的参数后面添加对应参数即可
```

* 可以并发执行的线程数目
函数`std::thread::hardware_concurrency()`返回机器可以支持并发线程的数目。该值有可能是0，一般作为一个参考。


* 线程的标识
线程的id标识着线程，可以通过thread的对象的成员函数`get_id()`来获取；如果thread的对象未绑定任何的操作，获取的id是0（VS2013中结果）；
通过函数`std::this_thread::get_id()`获取当前线程的id
```c++
std::thread a(test_func, param);
std::cout << "thread id: " << a.get_id() << std::endl;	// thread id: 11476
std::thread b;
std::cout << "thread id: : " << b.get_id() << std::endl;	// thread id: 0
std::thread::id curr_thread_id = std::this_thread::get_id();
```

## 二、线程间的数据共享
#### 2.1 使用互斥量保护数据共享
* 一个使用Mutex的基本例子
```c++
	static std::list<int> s_list;
	static std::mutex s_mutex;
	void add_to_list(int new_val) {
		std::lock_guard<std::mutex> m(s_mutex);
		s_list.push_back(new_val);
	}
	bool list_contain(int con_val) {
		std::lock_guard<std::mutex> m(s_mutex);
		return std::find(s_list.begin(), s_list.end(), con_val) != s_list.end();
	}
	void random_add_list() {
		for (int i = 0; i < 20; ++i) {
			int val = std::rand() % 1000;
			add_to_list(val);
			std::cout << "+" << val << std::endl;
		}
	}
	void random_list_find() {
		for (int i = 0; i < 20; ++i) {
			int val = std::rand() % 560;
			std::cout << (list_contain(val) ? "has " : "not has ") << val << std::endl;
		}
	}

	// 开始测试
	std::thread t0(random_add_list);
	std::thread t1(random_list_find);
	t0.join();
	t1.join();
```

#### 2.2 死锁处理
死锁一般出现原因：不同线程掌握了一个互斥量，都在等待对方的互斥量。避免方法:一般建议两个互斥量总以相同顺序上锁
C++中可以使用`std::lock`函数一次锁住多个互斥量。
```c++
class some_class;
swap(some_class& lhs, some_class& rhs);

class X {
private:
	std::mutex m;
	some_class mem;
public:
	X(const some_class& val): mem(val) {}
	friend void swap(X& lhs, X& rhs) {
		if (&lhs == &rhs) return;
		std::lock(lhs.m, rhs.m);
		std::lock_guard lock_a(lhs.m, std::adopt_lock);
		std::lock_guard lock_b(rhs.m, std::adopt_lock);
		swap(lhs.mem, rhs.mem);
	}
}
```
==避免死锁==
> * 避免嵌套锁，即使需要多个锁，使用`std::lock`
* 避免在持有锁时调用用户代码
* 使用固定的顺序获取锁
* 使用锁的层次结构

一个层次锁的实现：
```c++
class   hierarchical_mutex
{
	std::mutex  internal_mutex;
	unsigned long const   hierarchy_value;
	unsigned long previous_hierarchy_value;
	static  thread_local unsigned long this_thread_hierarchy_value;  //  1
	void check_for_hierarchy_violation()
	{
		if(this_thread_hierarchy_value  <=  hierarchy_value)  //  2
		{
		  throw   std::logic_error(“mutex hierarchy   violated”);
		}
	}
	void update_hierarchy_value()
	{
		previous_hierarchy_value=this_thread_hierarchy_value;    //  3
		this_thread_hierarchy_value=hierarchy_value;
	}
public:
  explicit hierarchical_mutex(unsigned long value):
      hierarchy_value(value),
      previous_hierarchy_value(0) {}
  void lock()
  {
	check_for_hierarchy_violation();
	internal_mutex.lock();   //  4
	update_hierarchy_value();    //  5
  }
  void unlock()
  {
    this_thread_hierarchy_value=previous_hierarchy_value;    //  6
    internal_mutex.unlock();
  }
  bool try_lock()
  {
    check_for_hierarchy_violation();
    if(!internal_mutex.try_lock())   //  7
       return  false;
    update_hierarchy_value();
    return  true;
  }
};
thread_local unsigned long hierarchical_mutex::this_thread_hierarchy_value(ULONG_MAX);  //  7

// thread_local是的static变量不再是全局的，而是线程中的全局
```

==std::unique_lock锁==
```c++
// std::defer_lock表明了互斥量在结构上应该保持解锁状态，直到使用std::lock上锁
class some_big_object;
void  swap(some_big_object& lhs,some_big_object&  rhs);
class X
{
private:
    some_big_object some_detail;
    std::mutex  m;
public:
    X(some_big_object const&  sd):some_detail(sd){}
    friend  void  swap(X& lhs,  X&  rhs)
    {
      if(&lhs==&rhs)
        return;
      std::unique_lock<std::mutex>  lock_a(lhs.m,std::defer_lock);  //  1
      std::unique_lock<std::mutex>  lock_b(rhs.m,std::defer_lock);  //  1 std::def_lock 留下未上锁的互斥量
      std::lock(lock_a,lock_b); //  2 互斥量在这里上锁
      swap(lhs.some_detail,rhs.some_detail);
    }
};
```

一个线程安全的类成员延迟初始化示例：
```c++
// 使用std::once_flag和std::call_once实现
class   X
{
private:
	connection_info connection_details;
	connection_handle    connection;
	std::once_flag  connection_init_flag;
	void open_connection()
	{
		connection = connection_manager.open(connection_details);
	}
public:
	X(connection_info    const&  connection_details_) :
		connection_details(connection_details_)
	{}
	void send_data(data_packet   const&  data)     //   1
	{
		std::call_once(connection_init_flag, &X::open_connection, this);    //   2
		connection.send_data(data);
	}
	data_packet  receive_data()    //   3
	{
		std::call_once(connection_init_flag, &X::open_connection, this);    //   2
		return  connection.receive_data();
	}
};
```
C++11提供了static变量的获取无需考虑数据竞争的问题，因为其保证static的变量的初始化和定义完全在一个线程中发生，在未完成前，其它线程不可以使用该变量。故现在对于写一个单例，只要使用static来申明唯一实例即可，不用考虑初始化过程中的数据竞争。


目前C++11没有采纳读写锁，可以使用boost的`boost::shared_mutex`来实现。
==一个使用boost的读写锁的例子==
```c++
#include <map>
#include <string>
#include <mutex>
#include <boost/thread/shared_mutex.hpp>
class   dns_entry;
class   dns_cache
{
	std::map<std::string, dns_entry> entries;
	mutable boost::shared_mutex  entry_mutex;
public:
	dns_entry  find_entry(std::string const& domain) const
	{
		boost::shared_lock<boost::shared_mutex> lk(entry_mutex);  //   1
		std::map<std::string, dns_entry>::const_iterator const   it =
			entries.find(domain);
		return  (it == entries.end()) ? dns_entry() : it->second;
	}
	void update_or_add_entry(std::string const&  domain, dns_entry const& dns_details)
	{
		std::lock_guard<boost::shared_mutex> lk(entry_mutex);  //   2
		entries[domain] = dns_details;
	}
};

// 在未调用update_or_add_entry时，不同线程都可以调用find_entry且不出错；
// 当update_or_add_entry被调用时，find_entry也处于被锁状态不可调用
```

#### 2.3 嵌套锁
如果一个线程已经获取一个std::mutex的实例（上锁），此时再对其上锁会出错。C++标准库中提供了`std::recursive_mutex`，为线程提供了可以获取同一个互斥量多次。
一般不推荐使用嵌套锁，如果存在这种情况，可以考虑重构代码，去掉这种调用需求。


## 三、同步并发操作






































































