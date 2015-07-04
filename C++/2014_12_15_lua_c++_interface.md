#Lua与C/C++间的交互
### 两种绑定c++函数到Lua中使用的方式
在Lua中使用的C/C++函数必须遵循下面的格式：
```cpp
typedef int (*lua_CFunction) (lua_State *L);
```
**1. 单个函数的注册**
```cpp
// C/C++中定义遵循上面格式的函数CForLuaFunc
// 在Lua文件中调用C/C++的CForLuaFunc函数需要一个对应的函数名
// 	此处命名为LuaUsedFunc
lau_register(L, "LuaUsedFunc", CForLuaFunc);
```

