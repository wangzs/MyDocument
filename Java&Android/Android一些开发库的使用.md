# Android一些开发库的使用
## 一、google的Gson库
#### 1.简单使用
> * **给定JSON数据格式，转成对应的java类**
使用http://www.jsonschema2pojo.org/ ，直接将json格式的数据输入，自动生成对应的java对象
* **使用转换后的java类**
```code
// 例：生成了一个TestExp的类
final TestExp testExp = new Gson().fromJson(jsonStr, TestExp.class);
// 通过上面的语句，就将jsonStr的json格式的字符串转为对应的java类对象
```















## 二、EventBus
http://blog.csdn.net/huangyanan1989/article/details/10858695