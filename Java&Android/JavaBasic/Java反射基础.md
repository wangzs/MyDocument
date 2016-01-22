# Java反射
#### 一、Classes
通过classes可以获取的信息：
* **Class Name**
获取类的名字
```java
package reflecttest;
public class CC {
    public static void main(String[] args) {
         // output: Class Name: reflecttest.CC
         //         Class Simple Name: CC
         System.out.println("Class Name: " + CC.class.getName());
         System.out.println("Class Simple Name: " + CC.class.getSimpleName());
    }
}
```
* **Class Modifiers**
获取类本身modifier属性（public/private/synchronized等)
```java
// Modifier类中常量：PUBLIC-0x00000001 PRIVATE-0x00000002等
public class CC {
    public class A {}
    public static class B {}
    interface C {}
    public static void main(String[] args) {
         // output:A modifiers: 1 | B modifiers: 9
         //        C modifiers: 1544 true
         System.out.println("A modifiers: " + A.class.getModifiers() + " | B modifiers: " + B.class.getModifiers());
         System.out.println("C modifiers: " + C.class.getModifiers() + " " + (new Boolean(Modifier.isInterface(C.class.getModifiers()))).toString());
    }
}
```
* **Package info**
获取类所在的package的信息
```java
package reflecttest;
public class CC {
    public static void main(String[] args) {
         // output: Package info: package reflecttest name: reflecttest
         Package p = C.class.getPackage();
         System.out.println("Package info: " + p.toString() + " name: " + p.getName());
    }
}
```
* **Superclass**
获取父类信息
```java
public class CC {
    public class A {}
    public static void main(String[] args) {
         // output: superclass info: class java.lang.Object name: java.lang.Object
         Class superclass = A.class.getSuperclass();
         System.out.println("superclass info: " + superclass.toString() + " name: " + superclass.getName());
    }
}
```
* **实现的interfaces**
获取类implement的接口列表
```java
package reflecttest;
public class CC { 
    interface C {}
    interface D {}
    class CD implements C,D {}
    public static void main(String[] args) {
         // output: implemented interface:interface reflecttest.CC$C
         //         implemented interface:interface reflecttest.CC$D
         Class[] interfaces  = CD.class.getInterfaces();
         for(Class i : interfaces) {
              System.out.println("implemented interface:" + i.toString());
          }
    }
}
```
* **Constructors**
获取类的所有public构造函数
```java
// output: constructor:public reflecttest.CC$CD(reflecttest.CC)
//         constructor:public reflecttest.CC$CD(reflecttest.CC, int, java.lang.String)
package reflecttest;
public class CC { 
    interface C {}
    interface D {}
      class CD implements C,D{
        public CD(){}
        public CD(int a, String b){}
      }
    public static void main(String[] args) {
        Constructor[] con = CD.class.getConstructors();
        for (Constructor c : con) {
            System.out.println("constructor:" + c.toString());
        }
    }
}
```
* **Methods**
获取类以及其父类的public函数方法
```java
// output: method:public void reflect.CC$CD.f1()
//         method:public static void reflect.CC$CD.f3(java.lang.String,int) 还有父类的public方法
package reflecttest;
public class CC { 
    interface C {}
    interface D {}
    static class CD implements C,D{
        public CD(){}
        public CD(int a, String b){}
        public void f1(){}
        private String f2(){return"";}
        public static void f2(String a, int b){}
      }
    public static void main(String[] args) {
        Method[] ms = CD.class.getMethods();
        for(Method m : ms) {
            System.out.println("method:" + m.toString());
        }
    }
}
```
* **Fields**
获取public的成员变量
```java
// output: field:public long reflect.CC$CD.m2
package reflecttest;
public class CC { 
    interface C {}
    interface D {}
    static class CD implements C,D{
        private int m0;
        private String m1;
        public long m2;
        public CD(){}
        public void f1(){}
      }
    public static void main(String[] args) {
       Field[] fs = CD.class.getFields();
       for(Field f : fs) {
           System.out.println("field:" + f.toString());
       }
    }
}
```
* **Annotations**
获取类的注解列表
```java
// output: annotation:@java.lang.annotation.Retention(value=RUNTIME)
//         annotation:@java.lang.annotation.Target(value=[TYPE])
package reflecttest;
public class CC { 
    @Retention(RetentionPolicy.RUNTIME)
    @Target(ElementType.TYPE)
    public @interface EA {
    }
    public static void main(String[] args) {
        Annotation[] as = EA.class.getAnnotations();
        for(Annotation a : as) {
            System.out.println("annotation:" + a.toString());
        }
    }
}
```
#### 二、反射之Constructors
















## 参考文档
* [Java反射Api教程][1]
* [Java Reflection Tutorial][2]

[1]:http://www.javacodegeeks.com/2014/11/java-reflection-api-tutorial.html
[2]:http://tutorials.jenkov.com/java-reflection/
