---
title: " Java反射\t\t"
tags:
  - java
  - 反射
url: 131.html
id: 131
comments: false
categories:
  - 未分类
date: 2018-10-19 09:52:39
---
参考：[反射](http://www.journaldev.com/1789/java-reflection-example-tutorial#get-set-private-field) [字段](http://blog.csdn.net/liujiahan629629/article/details/18013523)
引用：[方法](http://www.jianshu.com/p/5001b2add70b) [构造方法](http://www.cnblogs.com/LZL-student/p/5965991.html)

---


```java
    @Test
    public void checkTest(){
        AppResult a = new AppResult();
        Field[] fields = a.getClass().getDeclaredFields();//拿到私有的字段
        for (Field f : fields){
            System.out.println(f.getName());
            AccessCheck annotation = f.getAnnotation(AccessCheck.class);
            if (annotation!=null){
                if (annotation.level().equals("1")){

                    try {
                        f.setAccessible(true);  //允许访问私有字段
                        System.out.println(f.get(a)); //拿到a对象中f字段的值
                        f.set(a, "3");              //设置a对象中f字段的值
                        System.out.println(f.get(a));
                    } catch (IllegalAccessException e) {
                        e.printStackTrace();
                    }
                }
            }
        }
    }
```

对象：
```
    @AccessCheck(level = "1")
    private String levle = "2";
```
自定义注解
```
@Target({ElementType.FIELD})
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface AccessCheck {
    String value() default "";
    String level() default "1";
}
```
结果：
```
initReturnCode
initReturnMsg
initDetails
levle
2
3
```
---
## 反射详解

 拿到对象的Class
- 通过forName
  `Class<?> aClass = Class.forName("cn.ws.test.AppResult");`
- 通过.class
   `AppResult.class`
- 通过.getClass
   这个是每个对象有的属性
    通过Class文件创建对象
```java
    @Test
    public void caseTest(){
        try {
            Class<?> aClass = Class.forName("cn.ws.test.AppResult");
            System.out.println(aClass.getCanonicalName());//拿到全限定名
            Object o = aClass.newInstance();    //创建对象（无参构造方法）
            System.out.println(o);
        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        } catch (IllegalAccessException e) {
            e.printStackTrace();
        } catch (InstantiationException e) {
            e.printStackTrace();
        }
    }
```

### 私有和公有
获取私有的字段，方法的值时需要先将允许访问私有设置为true，如下：
`f.setAccessible(true);  //允许访问私有字段`

### 获取修饰符
通过反射可以获取字段的修饰符
`f.getModifiers()；`
返回的是一个int类型，对应修饰符代表数字的和
如`private static String a`;
返回的值为10
```
PUBLIC: 1
PRIVATE: 2
PROTECTED: 4
STATIC: 8
FINAL: 16
SYNCHRONIZED: 32
VOLATILE: 64
TRANSIENT: 128
NATIVE: 256
INTERFACE: 512
ABSTRACT: 1024
STRICT: 2048
```

这个方法在获取方法，类的修饰符也有效
字段的一些方法如下
## 字段（Field/s）
### 获取字段
```
Field[] fields = a.getClass().getDeclaredFields();//获取对象中所有私有字段，加Declared的都是获取私有的
    try {
            Field initReturnMsg = a.getClass().getField("initReturnMsg");
            //获取指定的公共的字段，如这里的initReturnMsg
        } catch (NoSuchFieldException e) {
            e.printStackTrace();
        }
```
### 代码
```java
for (Field field : fields){
            AccessCheck annotation = field.getAnnotation(AccessCheck.class);
            System.out.println(field.getModifiers());;//获取字段的修饰符
            System.out.println(field.getName()); //获取字段名
            if (annotation!=null){
                if (annotation.level().equals("1")){
                    try {
                        Class<?> type = field.getType();    //获取字段类型
                        System.out.println("是否相等："+type.equals(String.class)); //这个字段为String,输出为true
                        field.setAccessible(true);  //允许访问私有字段
                        System.out.println(field.get(a)); //拿到a对象中f字段的值
                        field.set(a, "zasjfdals");              //设置a对象中f字段的值
                        System.out.println(field.get(a));
                    } catch (IllegalAccessException e) {
                        e.printStackTrace();
                    }
                }
            }
        }
```
---
## 方法（Method/s）
### 获取方法

>只有通过方法签名才能找到唯一的方法，方法签名=方法名+参数列表（参数类型、参数个数、参数顺序）。
```java
    try {
            Class<?> appClass = Class.forName("cn.ws.test.AppResult");
            Method[] methods = appClass.getDeclaredMethods();   //获取所有的方法(私有和公有)
            Method[] methods1 = appClass.getMethods();  //获取所有公共方法，包括父类的
            Object o = appClass.newInstance();
            for (Method method : methods){
                method.getAnnotation(AccessCheck.class);//获取方法上的注解
                method.getModifiers();//获取修饰符
                System.out.println( method.getGenericReturnType());
                System.out.println(method.getReturnType());
                Class<?>[] parameterTypes = method.getParameterTypes(); //拿到参数的类型
                System.out.println("method name : "+method.getName() //获取方法名
                        +" returnType : "+method.getReturnType()
                        +" ParameterTypes : "+Arrays.toString(parameterTypes)
                        +""
                );//获取返回类型
            }
            System.out.println("------------------ln----------------");
            for (Method method : methods1){
                System.out.println("method name : "+method.getName()+" returnType : "+method.getReturnType());
            }
            } catch (ClassNotFoundException e) {
                e.printStackTrace();
            } catch (IllegalAccessException e) {
                e.printStackTrace();
            } catch (InstantiationException e) {
                e.printStackTrace();
        }
```
结果：
```
class java.lang.String
class java.lang.String
method name : testPrivateMethod returnType : class java.lang.String ParameterTypes : [class java.lang.String]
void
void
method name : setDetails returnType : void ParameterTypes : [class java.lang.Object]
void
void
method name : setReturnCode returnType : void ParameterTypes : [class java.lang.Object]
void
void
method name : setReturnMsg returnType : void ParameterTypes : [class java.lang.Object]
class java.lang.Object
class java.lang.Object
method name : getDetails returnType : class java.lang.Object ParameterTypes : []
class java.lang.Object
class java.lang.Object
method name : getReturnCode returnType : class java.lang.Object ParameterTypes : []
class java.lang.Object
class java.lang.Object
method name : getReturnMsg returnType : class java.lang.Object ParameterTypes : []
class java.lang.String
class java.lang.String
method name : testPublicMethod returnType : class java.lang.String ParameterTypes : []
------------------ln----------------
method name : setDetails returnType : void
method name : setReturnCode returnType : void
method name : setReturnMsg returnType : void
method name : getDetails returnType : class java.lang.Object
method name : getReturnCode returnType : class java.lang.Object
method name : getReturnMsg returnType : class java.lang.Object
method name : testPublicMethod returnType : class java.lang.String
method name : remove returnType : class java.lang.Object
method name : remove returnType : boolean
method name : get returnType : class java.lang.Object
method name : put returnType : class java.lang.Object
method name : values returnType : interface java.util.Collection
method name : clone returnType : class java.lang.Object
method name : clear returnType : void
method name : isEmpty returnType : boolean
method name : replace returnType : boolean
method name : replace returnType : class java.lang.Object
method name : replaceAll returnType : void
method name : size returnType : int
method name : entrySet returnType : interface java.util.Set
method name : putAll returnType : void
method name : putIfAbsent returnType : class java.lang.Object
method name : keySet returnType : interface java.util.Set
method name : compute returnType : class java.lang.Object
method name : computeIfAbsent returnType : class java.lang.Object
method name : computeIfPresent returnType : class java.lang.Object
method name : containsKey returnType : boolean
method name : containsValue returnType : boolean
method name : forEach returnType : void
method name : getOrDefault returnType : class java.lang.Object
method name : merge returnType : class java.lang.Object
method name : equals returnType : boolean
method name : toString returnType : class java.lang.String
method name : hashCode returnType : int
method name : wait returnType : void
method name : wait returnType : void
method name : wait returnType : void
method name : getClass returnType : class java.lang.Class
method name : notify returnType : void
method name : notifyAll returnType : void
```
### 获取指定的方法，并执行
```
    @Test
    public void methodTest1() throws ClassNotFoundException, IllegalAccessException, InstantiationException, NoSuchMethodException, InvocationTargetException {
        Class<?> appClass = Class.forName("cn.ws.test.AppResult");
        Object o = appClass.newInstance();
        Method testPrivateMethod = appClass.getDeclaredMethod("testPrivateMethod",String.class);//拿到指定名称的方法,后面可以传参数类型，来指定拿哪个方法
        testPrivateMethod.setAccessible(true);
        Object invoke = testPrivateMethod.invoke(o, "1345"); //重新传入参数，并执行方法
        System.out.println(invoke);
    }
```
结果：
```
传入的参数为1345
1345
```
### 调用静态方法和可变参数方法
> 使用反射调用静态方法: public Object invoke(Object obj,Object... args);
> 如果底层方法是静态的，那么可以忽略指定的 obj参数。将obj参数设置为null即可。
>
> 使用反射调用可变参数的方法:
> 对于数组类型的引用类型的参数,底层会自动解包,为了解决该问题,我们使用Object的一维数组把实际参数包装起来.
>
> (牢记)以后使用反射调用invoke方法,在传递实际参数的时候,无论是基本数据类型还是引用数据类型,也无论是可变参数类型,反正就是一切实际参数都包装在newObject[]{}中,就没问题。
>
> `m.invoke(方法底层所属对象,newObject[]{实际参数});通用`

---

## 构造方法（Constructor/s）
### 获取构造方法
```java
    @Test
    public void constructorTest() throws ClassNotFoundException, NoSuchMethodException {
        Class<?> appClass = Class.forName("cn.ws.test.AppResult");
        Constructor<?> constructor = appClass.getConstructor(String.class);//获取单个构造方法，非私有,不写默认是无参
        Constructor<?>[] constructors = appClass.getConstructors();//获取多个构造方法，非私有
        Constructor<?> declaredConstructor = appClass.getDeclaredConstructor(); //获取单个构造方法，可以获取私有的
        Constructor<?>[] declaredConstructors = appClass.getDeclaredConstructors();//获取所有的构造方法,包括私有
        }
```
### 使用构造器创建对象
```java
    try {
        /**
        对应上面获取构造方法
        */
            AppResult o = (AppResult)declaredConstructor.newInstance(); //使用无参构造器创建对象
            AppResult o1 = (AppResult) constructor.newInstance("使用参数为String的构成器创建的对象");
            System.out.println(o);
            System.out.println(o1);
        } catch (InstantiationException e) {
            e.printStackTrace();
        } catch (IllegalAccessException e) {
            e.printStackTrace();
        } catch (InvocationTargetException e) {
            e.printStackTrace();
        }
```
私有的构造器创建对象
```java
try {
            Constructor<?> privateConstrctor = appClass.getDeclaredConstructor(String.class, Object.class);
            privateConstrctor.setAccessible(true);
            AppResult o = (AppResult) privateConstrctor.newInstance("通过私有的构造器创建对象","12356");
            System.out.println(o);
        } catch (InstantiationException e) {
            e.printStackTrace();
        } catch (IllegalAccessException e) {
            e.printStackTrace();
        } catch (InvocationTargetException e) {
            e.printStackTrace();
        }
```
构造器也可以实现方法中的一些东西，诸如拿到参数的类型等，还可以拿到构造器所在类的Class对象

---

## 其他
```java
        Class<?> appClass = Class.forName("cn.ws.test.AppResult");
        Class<?> superclass = appClass.getSuperclass().getSuperclass();//拿到的是父类
        Type genericSuperclass = appClass.getSuperclass().getGenericSuperclass(); //得到的是泛型的父类
        System.out.println(genericSuperclass instanceof ParameterizedTypeImpl);
        System.out.println(superclass);
        System.out.println(genericSuperclass);
```
结果如下：
```
true
class java.util.AbstractMap
java.util.AbstractMap<K, V>
```

AppResult类
```java
package cn.ws.test;

import cn.ws.annotation.AccessCheck;
import org.springframework.stereotype.Component;

import java.util.HashMap;

public class AppResult<K,V> extends HashMap implements AppResultInterface{
    private static String initReturnCode = "1";
    private static String initReturnMsg = "成功！";
    private static Object initDetails = "";

    @AccessCheck(level = "1")
    private String levle ;

    public AppResult() {
        super.put("returnCode",initReturnCode);
        super.put("returnMsg",initReturnMsg);
        super.put("details",initDetails);
    }

    public AppResult(String returnCode,String returnMsg,Object details){
        super.put("returnCode",returnCode);
        super.put("returnMsg",returnMsg);
        super.put("details",details);
    }

    private AppResult(String successMsg,Object successDetails){
        super.put("returnCode",initReturnCode);
        super.put("returnMsg",successMsg);
        super.put("details",successDetails);
    }

    public AppResult(String successMsg){
        super.put("returnCode",initReturnCode);
        super.put("returnMsg",successMsg);
        super.put("details",initDetails);
    }

    public void setDetails(Object details){
        super.put("details",details);
    }

    public void setReturnCode(Object returnCode){
        super.put("returnCode",returnCode);
    }

    public void setReturnMsg(Object returnMsg){
        super.put("returnMsg",returnMsg);
    }

    public Object getDetails(){
        return super.get("details");
    }

    public Object getReturnCode(){
        return super.get("returnCode");
    }

    public Object getReturnMsg(){
        return super.get("returnMsg");
    }

    private String testPrivateMethod(String arg){
        System.out.println("传入的参数为"+arg);
        return arg;
    }

    public String testPublicMethod(){
        return "654321";
    }
}
```