### FlyWeight (享元模式)

#### :wrench:1. 概述

***

- 意图：运用共享技术有效地支持大量细粒度的对象。
- 主要解决：在有大量对象时，有可能会造成内存溢出，把其中共同的部分抽出来，如果有相同的业务请求，直接返回在内存中已有的对象。
- 何时使用：
  1. 系统中有大量对象。
  2. 这些对象消耗大量内存。
  3. 需要使用到池的场景。

- 应用实例：池技术，String常量池，数据库连接池，缓冲池等等，享元模式是池技术的重要实现方式。
- 优点：
  1. 可以节省重复创建对象的开销，因为被享元模式维护的相同对象只创建一次，当创建对象比较耗时的时候，便可以节省大量时间。
  2. 由于创建对象的数量减少，所以对系统内存的需求也减少，这会使得GC的压力也相应地降低，进而使得系统拥有一个更健康的内存结构和更快的反应速度。
- 缺点：提高了系统的复杂度，需要分离出外部状态和内部状态，而且外部状态具有固有化的性质，不应该随着内部状态的变化而变化，否则会造成系统的混乱。

|    角色    |                         作用                         |
| :--------: | :--------------------------------------------------: |
|  享元工厂  | 用来创建具体享元类，保证相同的享元对象可以被系统共享 |
|  抽象享元  |            定义需要共享的对象的业务接口。            |
| 具体享元类 |        实现抽象享元类的接口，完成某一具体逻辑        |
|    Main    |               通过享元工厂获得享元对象               |

![](https://ws1.sinaimg.cn/large/6b297ce5ly1g276p32237j20hm08s3yw.jpg)

##### 1.1 总结：

- 享元模式是设计模式中少数几个以提高系统性能为目的的模式之一。

- 它的核心思想是:如果在一个系统中存在多个相同的对象,那么只需共享一份对象的拷贝,而不必为每一次使用都创建新的对象。
- 在享元模式中，由于需要构造和维护这些可以共享的对象，因此，常常会出现一个工厂类，用于维护和创建对象。



#### :nut_and_bolt:2. 使用示例

***

举一个需要大量图形形状的例子.颜色为内部状态,形状的属性为外部状态.

- 享元工厂

  ```java
  /**
   * 享元工厂
   * 用以创建具体享元类,维护相同的享元对象.
   * Color 是内部状态.
   * X Y Radius 是 外部状态.
   */
  public class ShapeFactory {
  
      private static final ConcurrentHashMap<String, Shape> pool = new ConcurrentHashMap<>();
  
      public static Shape getCircle(String color) {
          if (!pool.containsKey(color)) {
              pool.put(color, new Circle(color));
              System.out.println("Creating circle of color :" + color);
              System.out.println("池中对象的数量"+pool.size());
          }
          return pool.get(color);
      }
  }
  ```

- 抽象享元

  ```java
  /**
   * 抽象享元
   * 定义需共享的对象的业务接口.
   */
  public abstract class Shape {
  
      public abstract void draw ();
  }
  
  ```

-  具体享元:

  ```java
  /**
   * 具体享元类
   * 实现抽象享元类的接口,完成具体的某一逻辑.
   */
  public class Circle extends Shape{
  
      private String color;
      private int x;
      private int y;
      private double radius;
  
      public Circle (String color) {
          this.color = color;
      }
  
      @Override
      public void draw() {
          System.out.println("Circle: Draw() [Color : " + color
                  +", x : " + x +", y :" + y +", radius :" + radius +" ]");
      }
  
      public String getColor() {
          return color;
      }
  
      public void setColor(String color) {
          this.color = color;
      }
  
      public int getX() {
          return x;
      }
  
      public void setX(int x) {
          this.x = x;
      }
  
      public int getY() {
          return y;
      }
  
      public void setY(int y) {
          this.y = y;
      }
  
      public double getRadius() {
          return radius;
      }
  
      public void setRadius(double radius) {
          this.radius = radius;
      }
  }
  ```

- Main测试:

  ```java
  @Test
      public void fun2() {
          for (int i = 0; i < 20; ++i) {
              Circle circle =
                      (Circle) ShapeFactory.getCircle(getRandomColor());
              circle.setX(getRandomX());
              circle.setY(getRandomY());
              circle.setRadius(100);
              circle.draw();
          }
      }
  ```

-  结果

  ```java
  Creating circle of color :Black
  池中对象的数量1
  Circle: Draw() [Color : Black, x : 12, y :33, radius :100.0 ]
  Circle: Draw() [Color : Black, x : 17, y :31, radius :100.0 ]
  Creating circle of color :Red
  池中对象的数量2
  Circle: Draw() [Color : Red, x : 95, y :67, radius :100.0 ]
  Creating circle of color :Green
  池中对象的数量3
  Circle: Draw() [Color : Green, x : 53, y :57, radius :100.0 ]
  Circle: Draw() [Color : Green, x : 24, y :10, radius :100.0 ]
  Circle: Draw() [Color : Red, x : 41, y :20, radius :100.0 ]
  Creating circle of color :White
  池中对象的数量4
  Circle: Draw() [Color : White, x : 5, y :17, radius :100.0 ]
  Circle: Draw() [Color : Green, x : 92, y :50, radius :100.0 ]
  Creating circle of color :Blue
  池中对象的数量5
  Circle: Draw() [Color : Blue, x : 74, y :93, radius :100.0 ]
  Circle: Draw() [Color : White, x : 86, y :18, radius :100.0 ]
  Circle: Draw() [Color : Blue, x : 7, y :21, radius :100.0 ]
  Circle: Draw() [Color : Black, x : 86, y :36, radius :100.0 ]
  Circle: Draw() [Color : White, x : 29, y :25, radius :100.0 ]
  Circle: Draw() [Color : White, x : 67, y :13, radius :100.0 ]
  Circle: Draw() [Color : Blue, x : 12, y :65, radius :100.0 ]
  Circle: Draw() [Color : Green, x : 31, y :74, radius :100.0 ]
  Circle: Draw() [Color : Black, x : 10, y :97, radius :100.0 ]
  Circle: Draw() [Color : Green, x : 31, y :23, radius :100.0 ]
  Circle: Draw() [Color : Green, x : 82, y :52, radius :100.0 ]
  Circle: Draw() [Color : White, x : 87, y :83, radius :100.0 ]
  ```

- 从结果中可以得知,即便需要20个对象,但如果用到的享元模式的话,最终创建的对象就只有5个而已.减少了大量的创建对象.在这里颜色是内部状态,而形状的属性是外部状态,外部状态并不会随着内部状态的改变而改变.



#### :key: 3. 结合JDK源码看设计模式——享元模式

***

想必这道面试题会经常出现:

```java
Integer a = Integer.valueOf(127);
Integer b = new Integer(127);
int c = 127;

System.out.println(a == b); 
System.out.println(b == c); 
System.out.println(a == c);
```

答案在分析完之后解答.

##### 3.1.1 Integer的内部状态和外部状态

- 内部状态

```java
/**
     * A constant holding the minimum value an {@code int} can
     * have, -2<sup>31</sup>.
     */
    @Native public static final int   MIN_VALUE = 0x80000000;

    /**
     * A constant holding the maximum value an {@code int} can
     * have, 2<sup>31</sup>-1.
     */
    @Native public static final int   MAX_VALUE = 0x7fffffff;
```

MIN_VALUE和MAX_VALUE无论外部传什么值进来,都不影响到这两个值.

- 外部状态

```java
/**
     * The value of the {@code Integer}.
     *
     * @serial
     */
    private final int value;

    /**
     * Constructs a newly allocated {@code Integer} object that
     * represents the specified {@code int} value.
     *
     * @param   value   the value to be represented by the
     *                  {@code Integer} object.
     */
    public Integer(int value) {
        this.value = value;
    }
```

##### 3.1.2 Integer中的享元模式

```java
public final class Integer extends Number implements Comparable<Integer> {
    
    private final int value;
    
    public Integer(int value) {
        this.value = value;
    }
    public static Integer valueOf(int i) {
        if (i >= IntegerCache.low && i <= IntegerCache.high)
            return IntegerCache.cache[i + (-IntegerCache.low)];
        return new Integer(i);
    }
}
```

上面是Integer中比较核心的代码.在日常工作中你是否有思考过使用valueOf还是直接new 创建Integer对象?.看完源码valueOf(int i)这个方法后会发现,它是先去找IntegerCache中是否有缓存一份,如果有就直接返回,没有就创建一个新的对象并放到一个数组中保存.

```java
static {
            // high value may be configured by property
            int h = 127;
            String integerCacheHighPropValue =
                sun.misc.VM.getSavedProperty("java.lang.Integer.IntegerCache.high");
            if (integerCacheHighPropValue != null) {
                try {
                    int i = parseInt(integerCacheHighPropValue);
                    i = Math.max(i, 127);
                    // Maximum array size is Integer.MAX_VALUE
                    h = Math.min(i, Integer.MAX_VALUE - (-low) -1);
                } catch( NumberFormatException nfe) {
                    // If the property cannot be parsed into an int, ignore it.
                }
            }
            high = h;

            cache = new Integer[(high - low) + 1];
            int j = low;
            for(int k = 0; k < cache.length; k++)
                cache[k] = new Integer(j++);

            // range [-128, 127] must be interned (JLS7 5.1.7)
            assert IntegerCache.high >= 127;
        }
```

- 看下测试情况	

```java
Integer a = Integer.valueOf(127);
Integer b = Integer.valueOf(127);
System.out.println(a == b);
```

- 结论:在需要创建大量对象的情况下,用到享元模式(缓冲池技术).即便定义了两个不相同的对象,但实际上指向的是不同的内存地址.在实际场景中,我们应该更多的是创建缓冲池,来达到缓冲池里对象的复用.这样就可以减少内存的开销,加快系统的响应速度.	

- 所以上面面试题的答案是:

  ```java
  Integer a = Integer.valueOf(127);
  Integer b = new Integer(127);
  int c = 127;
  
  System.out.println(a == b); //false 
  System.out.println(b == c); //true
  System.out.println(a == c); // true
  ```

当int 类型和Integer比较的时候会自动拆箱,比较的是里面值的大小是否相等.所以答案就是false,true,true.



**参考资料:**

[享元模式](http://www.runoob.com/design-pattern/flyweight-pattern.html)

Java程序性能优化----葛一鸣

[简说设计模式--享元模式](https://www.cnblogs.com/adamjwh/p/9070107.html)

[设计模式读书笔记----享元模式](https://www.cnblogs.com/chenssy/p/3330555.html)

[结合JDK源码看设计模式——享元模式 ](https://www.cnblogs.com/Cubemen/p/10666164.html)