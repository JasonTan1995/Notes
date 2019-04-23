### Observable Pattern(观察者模式)

#### :wrench:1. 概述

***

- 观察者模式是非常常用的一种设计模式.在软件系统中,当一个对象的行为依赖于另一个对象的状态时,观察者模式就相当有用.如果不使用观察者模式的结构,要实现类似的功能的话,只能开启一个线程专门不停地监听对象所依赖的状态.这将使系统的性能产生额外的负担.

- 意图:定义对象间的一种一对多的依赖关系,当一个对象的状态发生改变时,所有依赖于它的对象都得到通知并被自动更新.

- 主要解决:一个对象状态改变给其他对象通知的问题，而且要考虑到易用和低耦合，保证高度的协作。

- **何时使用：**一个对象（目标对象）的状态发生改变，所有的依赖对象（观察者对象）都将得到通知，进行广播通知。

- 优点: 

     > 1. 观察者和被观察者是抽象耦合的.
     > 2. 建立了一套触发机制.

- 缺点: 

  > 1. 时间问题:如果一个被观察者对象有很多的直接和间接的观察者的话,将通知到所有的观察者会消耗非常多的时间.
  > 2. 循环问题:观察者和观察目标之间循环依赖的话,会导致系统崩溃.
  > 3. 细节: 只知道了状态的变化,具体怎么变化的无法了解.

- 观察者模式主要角色:

     |    角色    |                             作用                             |
     | :--------: | :----------------------------------------------------------: |
     |  主题接口  | 被观察的对象.当其状态发生改变或者某事件发生时.它会将这个变化通知观察者.它维护了观察者所需要依赖的状态 |
     |  具体主题  |  具体主题实现了主题接口中的方法.其内部维护了一个观察者列表   |
     | 观察者接口 | 观察者借口定义了观察者的基本方法.当依赖状态发生改变时,主题接口就会调用观察者的update()方法. |
     | 具体观察者 |    实现了观察者接口.具体处理当被观察者状态改变的业务逻辑     |

- 观察者模式主要结构:

   UML 类图

#### :nut_and_bolt:2.使用示例

***

栗子: 拿油管(YouTube)作例子.以YouTuber的频道与其他用户之间的Subscribe来描述观察者模式.

由于JDK已经内置了一套观察者模式的实现.分别为:java.util.Observable类和java.util.Observer接口.我们可以直接复用.

- java.util.Observable(具体主题):

```java
public class Observable {
    private boolean changed = false;
    private Vector<Observer> obs;

    /** Construct an Observable with zero Observers. */

    public Observable() {
        obs = new Vector<>();
    }

    public synchronized void addObserver(Observer o) {
        if (o == null)
            throw new NullPointerException();
        if (!obs.contains(o)) {
            obs.addElement(o);
        }
    }

    public synchronized void deleteObserver(Observer o) {
        obs.removeElement(o);
    }

    public void notifyObservers() {
        notifyObservers(null);
    }

    public void notifyObservers(Object arg) {
        Object[] arrLocal;
        synchronized (this) {
            if (!changed)
                return;
            arrLocal = obs.toArray();
            clearChanged();
        }

        for (int i = arrLocal.length-1; i>=0; i--)
            ((Observer)arrLocal[i]).update(this, arg);
    }

    public synchronized void deleteObservers() {
        obs.removeAllElements();
    }

   
    protected synchronized void setChanged() {
        changed = true;
    }


    protected synchronized void clearChanged() {
        changed = false;
    }

    public synchronized boolean hasChanged() {
        return changed;
    }

    public synchronized int countObservers() {
        return obs.size();
    }
}
```

- java.util.Observer(观察者接口):

  ```java
  public interface Observer {
      /**
       * This method is called whenever the observed object is changed. An
       * application calls an <tt>Observable</tt> object's
       * <code>notifyObservers</code> method to have all the object's
       * observers notified of the change.
       *
       * @param   o     the observable object.
       * @param   arg   an argument passed to the <code>notifyObservers</code>
       *                 method.
       */
      void update(Observable o, Object arg);
  }
  ```

- YouTuber(具体观察者) :

  ```java
  /**
   * Observable Pattern
   * 观察者模式
   * 角色: 具体观察者
   * 作用: 实现了观察者接口的update().具体处理当被观察者状态改变
   * 或者某一事件发生时的业务逻辑.
   * 在这里模仿了YouTube上面.YouTuber与 订阅者之间的业务.
   */
  public class YouTuber implements Observer {
  
      private String name;
      private int followers;
  
      public String getName() {
          return name;
      }
  
      public void setName(String name) {
          this.name = name;
      }
  
      public int getFollowers() {
          return followers;
      }
  
      public void setFollowers(int followers) {
          this.followers = followers;
      }
  
      @Override
      public void update(Observable o, Object arg) {
          Followers follower = (Followers) o;
          followers++;
          System.out.println(name + "收到了来自"+follower.getName() + "的订阅");
          System.out.println("订阅人数为:" + followers);
      }
  }
  ```

- Followers(具体主题):

  ```java
  package observable.youtube;
  
  import java.util.Observable;
  
  /**
   * Observable Pattern
   * (观察者模式)
   *  角色:具体被观察者角色(具体主题)
   *  作用: 主体内部状态改变时,向所有观察者发出通知.
   *  这里内部维护了一个YouTuber的列表.每当有更新或者变动的时候
   *  就会通知列表中的YouTuber.
   */
  public class Followers extends Observable {
  
      private String name;
  
      public String getName() {
          return name;
      }
  
      public void setName(String name) {
          this.name = name;
      }
  
      public void subscribe(YouTuber youTuber) {
          setChanged();
          notifyObservers(youTuber);
      }
  }
  
  ```

  - 测试用例:

    ```java
    @Test
        public void test() {
            Followers followers = new Followers();
            followers.setName("Bob");
            Followers followers1 = new Followers();
            followers1.setName("Jason");
            YouTuber youtuber = new YouTuber();
            youtuber.setName("老高");
            followers.addObserver(youtuber);
            followers1.addObserver(youtuber);
            followers.subscribe(youtuber);
            followers1.subscribe(youtuber);
        }
    
    /**
    * 老高收到了来自Bob的订阅
    * 订阅人数为:1
    * 老高收到了来自Jason的订阅
    * 订阅人数为:2
    */
    ```

  说起发布跟订阅,还能联想到发布者模型与订阅者模型.这两者之间与观察者模式又有点相似.它们之间有什么联系呢?



#### :key:3. 观察者模式与发布/订阅模式的区别

***

- 观察者模式:定义了对象之间的一对多依赖.当一个对象改变时,它的所有依赖者都会收到通知并更新.

![](https://img2018.cnblogs.com/blog/810680/201811/810680-20181110141057999-1226534601.png)

- 发布者-订阅者模式:

  订阅者把自己想订阅的事件注册到调度中心,当该事件触发时候,发布者发布该事件到调度中心,由调度中心统一调度订阅者注册到调度中心注册.这意味着发布者和订阅者不知道彼此的存在.

​           ![](https://img2018.cnblogs.com/blog/810680/201811/810680-20181110141205995-2127160862.png)

- 主要区别:

![](https://img2018.cnblogs.com/blog/810680/201811/810680-20181110141219325-989417119.png)

1. 在Observer模式中Observer与Subject是彼此知道的,而且Subject还保留了Observer的记录.在Subject的内部维护了一个Observer的列表.而在Pub-sub模式中,发布者和订阅者不需要彼此了解.他们能通过调度中心或者注册中心的帮组下进行通信.
2. 在Pub-Sub模式中.组件是松散耦合的.每个组件对每个组件而言依赖性并不强.
3. Observer模式主要以同步方式实现.Pub-sub在大多数情况下是异步方式.(例如消息队列)

**参考资料:**

Java程序性能优化

[观察者和发布/订阅模式的区别](https://www.cnblogs.com/viaiu/p/9939301.html)

[结合JDK源码看设计模式--观察者模式](https://www.cnblogs.com/Cubemen/p/10708107.html)

