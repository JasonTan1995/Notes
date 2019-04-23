### Decorator Pattern(装饰器模式)

#### :wrench:1. 概述

***

- 装饰器模式拥有一个设计非常巧妙的结构,它可以动态添加对象功能.根据合成/聚合复用原则,代码复用应该尽可能使用委托,而不是继承.

  > 因为继承是一种紧密的耦合,任何父类的改动都会影响其子类,不利于系统维护.
  >
  > 而委托是松散耦合,只要接口不变,委托类的改动不会影响上层.

- 装饰器模式就充分利用了该原则,通过委托机制,复用系统中的各个组件.在运行时,可以将这些功能组件进行叠加,从而构造出一个"超级对象".这个对象就会拥有这些组件的所有功能.

- 主要解决: 在不想很多子类的情况下扩展类.(可代替继承)

- 如何解决: 将具体功能划分,同时继承装饰者.

- 关键代码: 

  1. Component类充当抽象角色,不应该具体实现.

  2. 修饰类引用和继承Component,具体扩展重写父类方法.

  3. 下图为实现的重点:

     ![image](https://wx1.sinaimg.cn/mw690/6b297ce5gy1fn9o6ib7v0j20b70da3z2.jpg)

- 优点: 
  1. 与继承关系不同的是,继承关系是静态的,而装饰器模式是动态的.装饰器模式允许系统动态决定要增强的功能,或者不需要哪一个功能.继承则是在系统运行前就已经决定好了.
  2. 灵活性.通过不同的具体装饰类以及装饰类的排列组合,设计师可以创造出很多不同行为的组合.

- 缺点:
  1. 多层装饰的复杂,装饰器模式会产生比使用继承关系更多的对象,更多的对象会使得排查错误变得困难.
  2. 装饰者会导致出现很多小对象，如果过度使用，会让程序变得复杂.

##### 1.1装饰器模式基本结构

![](https://upload-images.jianshu.io/upload_images/3985563-a0d0ac0c5bdf5c93.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/628)

|             角色              |                             作用                             |
| :---------------------------: | :----------------------------------------------------------: |
|      组件接口(Component)      | 组件接口是装饰者和被装饰者的超类或者接口,定义了被装饰者的核心功能和装饰者需要加强的功能点. |
|  具体组件(ConcreteComponent)  |     具体组件实现了组件接口的核心方法,它也是被装饰的对象.     |
|       装饰者(Decorator)       |                   实现组件接口或继承超类.                    |
| 具体装饰器(ConcreteDecorator) |                   具体实现装饰的业务逻辑.                    |



#### :nut_and_bolt:2. 使用示例

***

本文将会举出两个例子,一个是抽象类的实现方式,一个是接口的.其实都是大同小异.

##### 2.1 抽象类实现装饰器模式

看了网上很多例子,看完还是理解不太深入.在这里我就举一个武器跟加强武器的例子.

- 先来说说各个角色:

|             角色              |          对应的组件          |
| :---------------------------: | :--------------------------: |
|      组件接口(Component)      |            Weapon            |
|  具体组件(ConcreteComponent)  |            Sword             |
|       装饰者(Decorator)       |     StrengthenDecorator      |
| 具体装饰器(ConcreteDecorator) | LongDecorator,SharpDecorator |

- 组件接口(Component):

  ```java
  public abstract class Weapon {
  
      public abstract double attackPower();
  }
  ```

- 具体组件(ConcreteComponent):

  ```java
  public class Sword extends Weapon {
  
      @Override
      public double attackPower() {
          return 10;
      }
  }
  ```

- 装饰器(Decorator):

  ```java
  public abstract class StrengthenDecorator extends Weapon {
  
      public abstract double attackPower();
  
  }
  
  ```

-  具体装饰器(ConcreteDecorator):

  ```java
  public class LongDecorator extends StrengthenDecorator {
  
      private Weapon weapon;
  
      public LongDecorator(Weapon weapon) {
          this.weapon = weapon;
      }
  
      @Override
      public double attackPower() {
          return weapon.attackPower() + 10;
      }
  }
  //*******************************************************************************************
  public class SharpDecorator extends StrengthenDecorator {
  
      private Weapon weapon;
  
      public SharpDecorator(Weapon weapon) {
          this.weapon = weapon;
      }
  
      @Override
      public double attackPower() {
          return weapon.attackPower() + 20;
      }
  }
  ```

  效果:

  ```java
  @Test
      public void weaponDecoratorTest() {
          Sword sword = new Sword();
          LongDecorator longDecorator = new LongDecorator(new SharpDecorator(sword));
          System.out.println(longDecorator.attackPower());
      }
  //剑的攻击力为40.0
  ```


##### 2.2 接口实现装饰器模式

齐天大圣72变的例子

|             角色              |  对应的组件  |
| :---------------------------: | :----------: |
|      组件接口(Component)      | TheGreatSage |
|  具体组件(ConcreteComponent)  |    Monkey    |
|       装饰者(Decorator)       |    Change    |
| 具体装饰器(ConcreteDecorator) |  Bird,Fish   |

- 组件接口(Component)

  ```java
  /**
   * Component
   * 组件接口.
   */
  public interface TheGreatSage {
  
      public void move();
  }
  
  ```

- 具体组件(ConcreteComponent)

  ```java
  /**
   * 具体组件
   * 具体组件实现了组件接口的核心方法.
   */
  public class Monkey implements TheGreatSage {
  
      @Override
      public void move() {
          System.out.println("Monkey Move");
      }
  }
  ```

- 装饰者(Decorator)

  ```java
  /**
   * 装饰者
   * 实现或继承组件接口.
   * 对于ConcreteComponent来说，不需要知道Decorator的存在，Decorator是一个接口或抽象类
   */
  public class Change implements TheGreatSage {
  
      private TheGreatSage theGreatSage;
  
      public Change(TheGreatSage theGreatSage) {
          this.theGreatSage = theGreatSage;
      }
  
      @Override
      public void move() {
          theGreatSage.move();
      }
  }
  ```

- 具体装饰器(ConcreteDecorator)

  ```java
  /**
   * 具体装饰者
   */
  public class Bird extends Change {
  
      public Bird(TheGreatSage theGreatSage) {
          super(theGreatSage);
      }
  
      @Override
      public void move() {
          System.out.println("Bird move");
      }
  }
  //*****************************************************
  /**
   * 具体装饰者
   */
  public class Fish extends Change {
  
      public Fish(TheGreatSage theGreatSage) {
          super(theGreatSage);
      }
  
      @Override
      public void move() {
          System.out.println("Fish move");
      }
  }
  
  ```

  看图会比较直观:

![img](https://upload-images.jianshu.io/upload_images/3985563-44abe91a268fa580.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/774)

![img](https://upload-images.jianshu.io/upload_images/3985563-44abe91a268fa580.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/774)



#### :key:3. 装饰器模式的一些变化

##### 3.1 装饰器模式的简化

一般情况下,装饰器模式的实现要比上面给出的例子简单.

- 如果只有一个ConcreteDecorator(具体装饰器)的话,就没有必要建立一个单独的Decorator类了.

![img](https://upload-images.jianshu.io/upload_images/3985563-d4d9c54c1139c455.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/641)

```java
// 就没有必要继承StrengthenDecorator,直接继承Weapon
public class LongDecorator extends Weapon {

    private Weapon weapon;

    public LongDecorator(Weapon weapon) {
        this.weapon = weapon;
    }

    @Override
    public double attackPower() {
        return weapon.attackPower() + 10;
    }
}
```

- 如果只有一个ConcreteComponent类，那么可以考虑去掉抽象的Component类（接口），把Decorator作为一个ConcreteComponent子类.

  ![img](https://upload-images.jianshu.io/upload_images/3985563-d82f42bcec447d19.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/724)



**参考资料:**

[装饰器模式](http://www.runoob.com/design-pattern/decorator-pattern.html)

[设计模式详解——装饰者模式](https://www.jianshu.com/p/d7f20ae63186)

[Java设计模式之装饰者模式(Decorator pattern)](https://www.jianshu.com/p/c26b9b4a9d9e)

[Java IO流详解（二）——IO流的框架体系](https://www.jianshu.com/p/80f8a74d4662)