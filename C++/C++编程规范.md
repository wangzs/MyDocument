# C++编程规范（Google风格）
## 1.头文件
##### 1.1 #define使用
	所有头文件都应该使用 #define 防止头文件被多重包含, 命名格式当是: <PROJECT>_<PATH>_<FILE>_H_

#### 1.2 头文件依赖
	可以使用前置申明的地方，尽量不要引用#include
只申明`Class xx；`的情况下，可以对xx进行哪些操作：
* 不能使用xx的对象，可以使用其指针和引用
* 申明一个函数，包含xx类
* 申明xx类的静态数据成员

#### 1.3 inline函数
函数的行数较少（≤10行）的情况才定义为inline
**实用经验准则：**inline函数中包含循环和switch语句是不划算的。

#### 1.4 函数参数的顺序
	函数参数顺序为: 先输入参数, 后输出参数

#### 1.5 #include的路径与顺序
	头文件引用顺序：C库头、C++库头、其他库的头、本项目的头
例 `google-awesome-project/src/base/logging.h` 应该按如下方式包含:
```cpp
#include "base/logging.h"
```
例 `dir/foo.cc` 的主要作用是实现或测试`dir2/foo2.h`的功能, `foo.cc`中包含头文件的次序如下:
```cpp
#include "foo/public/fooserver.h" // 优先位置
#include <sys/types.h>
#include <unistd.h>
#include <hash_map>
#include <vector>
#include "base/basictypes.h"
#include "base/commandlineflags.h"
#include "foo/public/bar.h"
```






















