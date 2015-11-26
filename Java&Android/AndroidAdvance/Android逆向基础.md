# Android逆向
## 1. Substrate基础
#### 1.1 Cydia的一个例子
[官方示例][1]
* 首先需要安装Cydia的[Substrate][2]的apk到机器上（机器需要root，才可以使用其功能）；如果已经root过了，点击`Link Substrate Files`时，可能出现`[Unix.cpp:99]Permission denied`，则需要修改`\system\vendor\lib`的权限，添加`w`权限。修改权限后，点击就会弹出四个按钮的页面。
* 在安装了Cydia的[Substrate][2]后，添加hook的实例
  ```java
  // 创建一个名为Main的类
  public class Main {
    static void initialize() {
        MS.hookClassLoad("android.content.res.Resources", new MS.ClassLoadHook() {
            public void classLoaded(Class<?> resources) {
                Method getColor;
                try {
                    getColor = resources.getDeclaredMethod("getColor", Integer.TYPE);
                } catch (NoSuchMethodException e) {
                    getColor = null;
                }
                if (getColor != null) {
                    MS.hookMethod(resources, getColor, new MS.MethodAlteration<Resources, Integer>() {
                        public Integer invoked(Resources resources, Object... args) throws Throwable {
                            return invoke(resources, args) & ~0x0000ff00 | 0x00ff0000;
                        }
                    });
                }
            }
        });
      }
  }

  // 向AndroidManifest.xm中添加如下
  <uses-permission android:name="cydia.permission.SUBSTRATE"/>
  <application ...>
    <meta-data android:name="com.saurik.substrate.main"
            android:value=".Main"/>
  </application>
  ```
  安装后，选择Cydia的[Substrate][2]的`Restart System(Soft)`重启系统，就会影响系统的很多字体的显示颜色。

#### 1.2 Substrate的java接口
* [MS.hookClassLoad][3]: 指定的class载入时通知触发第二个参数hook
  ```java
  /**
 * 指定的类被加载的时候发出通知
 * @param name Class的包名+类名,如android.content.res.Resources
 * @param hook MS.ClassLoadHook的一个实例，当这个类被加载的时候，它的classLoaded方法会被执行
 */
void hookClassLoad(String name, MS.ClassLoadHook hook);
```

* [MS.hookMethod][4]: 允许开发者提供一个回调函数替换原来的方法
  ```java
  /**
 * Hook一个指定的方法，并替换方法中的代码
 * @param _class 加载的目标类，为classLoaded传下来的类参数
 * @param member 通过反射得到的需要hook的方法(或构造函数). 注意：不能HOOK字段 (在编译的时候会进行检测).
 * @param hook MS.MethodHook的一个实例，其包含的invoked方法会被调用，用以代替member中的代码
 * @param old Hook前方法，类似C中的方法指针
 */
  public static <T, R> void hookMethod(
            Class clazz,
            Method member,  // 或者Constructor对象参数
            MS.MethodHook<T, R> hook,
            MS.MethodPointer<T, R> old) {
        hookMethod_(clazz, member, hook, old);
  }
  public static <T, R> void hookMethod_(
          Class clazz,
          Member member,
          final MS.MethodHook<T, R> hook,
          MS.MethodPointer<T, R> old) {
        _MS.hookMethod(clazz, member, hook != null?new com.saurik.substrate._MS.MethodHook() {
            public R invoked(Object thiz, Object... args) throws Throwable {
                return hook.invoked(thiz, args);
            }
        }:null, old != null?old.method_:null);
  }
  /**
   * Hook一个指定的方法，并替换方法中的代码
   * @param _class 加载的目标类，为classLoaded传下来的类参数
   * @param member 通过反射得到的需要hook的方法(或构造函数). 注意：不能HOOK字段 (在编译的时候会进行检测).
   * @param alteration
   */
  public static <T, R> void hookMethod(
            Class clazz,
            Method member,  // 或者Constructor对象参数
            MS.MethodAlteration<T, R> alteration) {
        hookMethod_(clazz, member, alteration);
  }
  private static <T, R> void hookMethod_(
            Class clazz,
            Member member,
            MS.MethodAlteration<T, R> alteration) {
          hookMethod_(clazz, member, alteration, alteration.method_);
  }
  ```
* [MS.moveUnderClassLoader][5]: 使用一个ClassLoader重载一个对象
  ```java
  /**
   * 使用一个ClassLoader重载一个对象
   * @param loader 使用的ClassLoader
   * @param object 带重载的对象
   * @return 重载后的对象
   */
  <T> T moveUnderClassLoader(ClassLoader loader, T object);
  ```

<strong style="background:#456743">一个例子</strong>
```java
// AndroidManifest.xml中的配置同官方示例一样;xxx.xxx.xxx.MainActivity就是例子应用中的一个Activity类
public class Main {
  static void initialize() {
    MS.hookClassLoad("xxx.xxx.xxx.MainActivity", new MS.ClassLoadHook() {
            public void classLoaded(Class<?> resources) {
                Log.e("TAG", "====> class MainActivity");
                Method testSubstrate;
                try {
                    // MainActivity中定义了一个testSubstrate()函数，并在onCreate中调用了
                    testSubstrate = resources.getDeclaredMethod("testSubstrate", null);
                } catch (NoSuchMethodException e) {
                    testSubstrate = null;
                }
                if (onCreate != null) {
                    final MS.MethodPointer old = new MS.MethodPointer();
                    MS.hookMethod(resources, testSubstrate, new MS.MethodHook() {
                        public Object invoked(Object object, Object... args) throws Throwable {
                            Log.e("TAG", "====> inject testSubstrate");
                            // 继续执行原testSubstrate的方法
                            Object result = old.invoke(object, args);
                            return result;
                        }
                    }, old);
                }
            }
      });
  }
}
```
在写好上面代码，并设置了AndroidManife.xml，同样需要打开`Substrate`的应用，并点击`Restart System(Soft)`重启系统（直接硬件重启没起作用），才能真正的起到效果；

#### 1.3 Substrate的c/c++接口



























## 第三方注入相关库
* [Cydia Substrate][7]
* [frida][8]


[1]:http://www.cydiasubstrate.com/id/20cf4700-6379-4a14-9bc2-853fde8cc9d1/
[2]:http://www.cydiasubstrate.com/download/com.saurik.substrate.apk
[3]:http://www.cydiasubstrate.com/api/java/MS.hookClassLoad/
[4]:http://www.cydiasubstrate.com/api/java/MS.hookMethod/
[5]:http://www.cydiasubstrate.com/api/java/MS.moveUnderClassLoader/

[7]:http://www.cydiasubstrate.com
[8]:http://www.frida.re/
