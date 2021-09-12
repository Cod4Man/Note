# Reflect

## 1. Enhancer(cglib)

- 依赖： cglib.jar (asm.jar)

- demo

```java
private static void enhanderDemo() {
    try {
        //                Class<?> aClass = Class.forName("com.codeman.JVMNGC.OOMDemo");
        //                Method method = aClass.getMethod("staticMethod", null);
        //                Object invoke = method.invoke(aClass, null);
        Enhancer enhancer = new Enhancer();
        enhancer.setSuperclass(OOMDemo.class); // 代理类
        enhancer.setCallback((MethodInterceptor) (o, method, objects, methodProxy) -> methodProxy.invoke(o, null));
        System.out.println("this is before method");
        OOMDemo oomDemo = (OOMDemo) enhancer.create(); // 创建代理类
        oomDemo.staticMethod(); // 调用方法 // this is static method 
        System.out.println("this is after method");
    } catch (Exception e) {
        e.printStackTrace();
    }
}
```

## 2. getModifiers 

- `public native int getModifiers()`,通过返回值可以确定该对象（方法/字段）的全部访问修饰符

  得到的就是 前面的 的修饰符 ，这个方法 字段和方法 都有。这个方法的值是 修饰符 相加的到的值 

  ```java
  public class Test1 {
  
      String c;
      public String a;
      private String b;
      protected String d;
      static String e;
      final String f="f";
  	
      public static void main(String[] args) {
          Field[] fields = Test1.class.getDeclaredFields();
          for( Field field: fields) {
              System.out.println( field.getName() +":" + field.getModifiers() );
              // c:0
              // a:1
              // b:2
              // d:4
              // e:8
              // f:16
          }
      }
  }
  ```

  所以：什么都不加 是0 ， public  是1 ，private 是 2 ，protected 是 4，static 是 8 ，final 是 16。

  如果是   public  static final  三个修饰的 就是3 个的加和 为 25 。

## 3. isAssignableFrom

- 作用： 判断类是否有继承关系，类似instanceof

- 与instanceof的区别

  - instanceof 是判断**实例**是否是继承关系 ">="

    ```java
    if (businesDetail instanceof DiscoutBizDetail)
    ```

  - isAssignableFrom则是判断**类**是否是继承关系 ">="

    ```java
    BusinesDetail.class.isAssignableFrom(DiscoutBizDetail.getClass())  //  true
    List.class.isAssignableFrom(ArrayList.class)  // true
    ```

isAssignableFrom() 

## 4. ParameterizedType

所有的泛型类 instanceof ParameterizedType.

MP就用到了这个判断BaseMapper\<Model> 用来获取\<Model>

## 5. 获取接口的所有存活子类

```java
public class ImplCountClient {

    public static void main(String[] args) {
        // BaseClass.class.getDeclaredClasses();
        getAllSubclassOfBaseClass();
    }

    @SuppressWarnings("all")
    private static List<Class<BaseClass>> getAllSubclassOfBaseClass() {
        SubClass1 subClass1 = new SubClass1();
        Field field = null;
        Vector v = null;
        List<Class<BaseClass>> allSubclass = new ArrayList<Class<BaseClass>>();
        Class<BaseClass> baseClassClass = null;
                ClassLoader classLoader = Thread.currentThread().getContextClassLoader();
        Class<?> classOfClassLoader = classLoader.getClass();
        try {
            baseClassClass = BaseClass.class;
        } catch (Exception e) {
            throw new RuntimeException(
                    "无法获取到BaseClass的Class对象!查看包名,路径是否正确");
        }
        while (classOfClassLoader != ClassLoader.class) {
            classOfClassLoader = classOfClassLoader.getSuperclass();
        }
        try {
            field = classOfClassLoader.getDeclaredField("classes");
        } catch (NoSuchFieldException e) {
            throw new RuntimeException(
                    "无法获取到当前线程的类加载器的classes域!");
        }
        field.setAccessible(true);
        try {
            v = (Vector) field.get(classLoader);
        } catch (IllegalAccessException e) {
            throw new RuntimeException(
                    "无法从类加载器中获取到类属性!");
        }
        for (int i = 0; i < v.size(); ++i) {
            Class<?> c = (Class<?>) v.get(i);
            if (baseClassClass.isAssignableFrom(c) && !baseClassClass.equals(c) /*&& !abstractBaseClassClass
                    .equals(c)*/) {
                allSubclass.add((Class<BaseClass>) c);
            }
        }
        return allSubclass;
    }
}

interface BaseClass {
}

class SubClass1 implements BaseClass{
}

class SubClass2 implements BaseClass{
}
```



