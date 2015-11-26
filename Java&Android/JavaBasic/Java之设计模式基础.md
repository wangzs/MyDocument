# Java之设计模式基础

## 1. 单例模式
  单例即程序运行时，只有一个实例对象。
**一个单例模式的实例**
```Java
public class JavaSingleton {
  static {
    // 可以添加一些复杂的初始化操作
  }
  private static JavaSingleton ourInstance = new JavaSingleton();
  public static JavaSingleton getInstance() {
      return ourInstance;
  }
  private JavaSingleton() {
  }
}
```
上面的单例，即使未调用`getInstance`，也会创建出实例对象。下面的方法可以实现真正调用`getInstance`时才创建实例：
```Java
public class JavaSingleton {
  private static class SingletonHolder {
    private static final JavaSingleton ourInstance = new JavaSingleton();
  }
  // 只有在真正调用getInstance时才会初始化JavaSingleton对象
  public static JavaSingleton getInstance() {
    return SingletonHolder.ourInstance;
  }
  private JavaSingleton() {
  }
}
```
**序列化时出现的问题**
单例实现了`Serializable`的接口时，则默认情况下，反序列化会产生新的对象，这时需要修改`readResolve`的接口：
```Java
public class JavaSingleton implements Serializable {
  private static final long serialVersionUID = -3976924865522920913L;
  private static JavaSingleton ourInstance = new JavaSingleton();
  public static JavaSingleton getInstance() {
      return ourInstance;
  }
  private JavaSingleton() {
  }
  private Object readResolve() {
      return ourInstance;
  }
}
```

## 2. 工厂方法
**Define an interface for creating an object, but let subclasses decide which class to instantiate. Factory Method lets a class defer instantiation to subclasses.**
工厂方法可以让接口使用者不用过多关注同系列的有些许差异的类的创建。
如一般考虑同系列但有差异的类的创建：
```Java
// 创建两把枪AK47和M4
public static class GunAk47 {
    public GunAk47() {
        System.out.println("====> Gun AK47");
    }
    void doSomeThing() {
      // xxx
    }
}
public static class GunM4 {
    public GunM4() {
        System.out.println("====> Gun M4");
    }
    void doSomeThing() {
      // yyy
    }
}

// 创建了类接口后，使用者在使用时需要关注这两个类的对象创建
GunAk47 ak47 = new GunAk47();
GunM4   m4 = new GunM4();
// 做各自的事
ak47.doSomeThing();
m4.doSomeThing();
```
使用工厂方法来创建这两个对象：
```Java


```
