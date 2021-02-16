# Enum

## 1. 小型数据库

```java
public enum EnumDemo {

    ZS(1, "bizType", "zhangsan"),
    LS(2, "bizType", "lisi"),
    WW(3, "bizType", "wangwu");


    private Integer id;
    private String type;
    private String value;



    EnumDemo(Integer id, String type, String value) {
        this.id = id;
        this.type = type;
        this.value = value;
    }


    public static EnumDemo getEnumDemoById(Integer id) {
        return (EnumDemo) Arrays.stream(EnumDemo.values())
                .filter(en -> en.getId() == 1).toArray()[0];
    }

    public static String getValueById(Integer id) {
        return (String) Arrays.stream(EnumDemo.values())
                .filter(en -> en.getId() == 1)
                .map(EnumDemo::getValue).toArray()[0];
    }

    public Integer getId() {
        return id;
    }

    public String getType() {
        return type;
    }

    public String getValue() {
        return value;
    }
}


class EnumTest {

    public static void main(String[] args) {
        System.out.println(EnumDemo.ZS.getId());  // 1
        System.out.println(EnumDemo.ZS.getValue()); // zhangsan
        System.out.println();
        Arrays.stream(EnumDemo.values())
                .filter(en -> en.getId() == 1)
                .map(EnumDemo::getValue)
                .forEach(System.out::println);  // zhangsan

        System.out.println();
        System.out.println();
        System.out.println(EnumDemo.getEnumDemoById(1));  // ZS
        System.out.println(EnumDemo.getValueById(1));     // zhangsan
    }
}

```

