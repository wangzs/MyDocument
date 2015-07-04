# SGI STL学习
### 1、std::accumulate
```
// 两个原型
template <class InputIterator, class T>
T accumulate (InputIterator first, InputIterator last, T init);
template <class InputIterator, class T, class BinaryOperation>
T accumulate (InputIterator first, InputIterator last, T init,
                 BinaryOperation binary_op);
```
**具体定义：**
```c++
template <class InputIterator, class T>
   T accumulate (InputIterator first, InputIterator last, T init)
{
  while (first!=last) {
    init = init + *first;  // or: init=binary_op(init,*first) for the binary_op version
    ++first;
  }
  return init;
}
```

**自定义累加函数的使用样例**
```c++
template <typename T>
class accumuFuncObj {
public:
	T operator()(const T& a, const T& b) {
		return a + "-" + b;
	}
};

vector<string> ppString{"1","2","3"};
// a-1-2-3
accumulate(ppString.begin(), ppString.end(), string("a"), accumuFuncObj<string>());
```

### 2.Binary Function
**原型**
```
template <class Arg1, class Arg2, class Result> struct binary_function;
```
**具体定义**
```c++
template <class Arg1, class Arg2, class Result>
  struct binary_function {
    typedef Arg1 first_argument_type;
    typedef Arg2 second_argument_type;
    typedef Result result_type;
  };
```
































