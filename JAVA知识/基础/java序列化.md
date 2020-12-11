Serialization 将对象的状态信息转换为可以存储或传输的形式的过程

java中一般使用场景

RMI远程调用

数据库存储（切记是针对于对象的持久化存储（一般数据库会采用blob这种类型的字段对此可持久化对象进行保存，而不是某种属性字段）

举个例子

``` java
@Entity(name = "abc")
public class Abc {
  @Id
  private Integer a;

  private String b;

  private String c;

  private VCD vcd;
  
  .....
}
```

```java
public class VCD implements Serializable {
  private String bc;

  private String cd;

  private String dd;
```

这其中的VCD这个对象就需要进行可序列化接口继承才能进行正常的序列化操作并存入数据库的blob这个字段



transient  这个关键字是是针对这种继承可序列化对象中，不想进行序列化变量准备的前缀关键字。使用后，在进行序列化操作的时候，会忽略这个字段.从而在反序列化的时候缺少这个字段的内容

