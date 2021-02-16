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

