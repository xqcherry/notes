# JAVA基础



## 特点

- 平台无关
- 面向对象
- 内存管理



## 数据类型

![img](https://cdn.xiaolincoding.com//picgo/1715930632378-7f03a5ae-3364-41d4-88a8-428997d543dd.png)

**装箱与拆箱**：基本数据类型和对应包装类之间进行转换的过程

```Java
Interger i = 10; // 装箱
int i = 10; // 拆箱
```



**Java为什么要有Interger**

1. 泛型只能使用引用类型，不能使用基本类型，要在泛型中使用`int`类型，必须使用`Interger`包装类

   ```java
   List<Integer> list = new ArrayList<>;
   list.add(3);
   list.add(1);
   list.add(2);
   Collection.sort(list);
   System.out.println(list);
   ```

2. java中基本类型和引用类型不能直接转换，必须使用包装类
   ```java
   int i = 10;
   Integer integer = new Integer(i);
   String str = integer.toString();
   System.out.println(str);
   ```

3. java集合只能存储对象，不能存储基本数据类型

   ```java
   List<Integer> list = new ArrayList<>();
   list.add(3);
   list.add(1);
   list.add(2);
   int sum = list.stream().mapToInt(Integer::intValue).sum();
   System.out.println(sum);
   ```



## 面向对象

- 封装，继承，多态

1. 封装：属性私有化`private`，通过公共方法`public`访问

   ```java
   public class Animal() {
       private String name;
       public void setName(String name) {
           this.name = name;
   	}
       public String getName() {
           return name;
       }
   }
   ```

2. 继承：子类通过extends关键字获取父类功能

   ```java
   // Dog 类继承了 Animal，自动拥有了 name 属性和相关方法
   public class Dog extends Animal {
       public void bark() {
           System.out.println(getName() + "在狗叫")
       }
   }
   public class Dog extends Animal {
       public void meow() {
           System.out.println(getName() + "在猫叫")
       }
   }
   ```

3. 多态：**类继承多态**和**接口多态**

   多态体现：

   - 方法重载：对应`add`方法，可定义`add(int a, int b)`和`add(double a, double b)`

   - 方法重写：定义`sound`方法，子类`Dog`可重写`bark`，`cat`可以实现`meow`

   - 接口与实现：多个类可以是实现同一个接口，并且用接口类型的引用来调用这些类的方法

     ```java
     // 方法重写（类继承多态）
     class Animal() {
         public void sound() {
             System.out.println("动物发出声音...");
         }
     }
     class Dog extends Animal() {
         @Override
         public void sound() {
             System.out.println("汪汪");
         }
     }
     class Cat extends Animal() {
         @Override
         public void sound() {
             System.out.println("喵喵");
     	}
     }
     // 运行时多态
     public class OverrideDemo {
         public static void main(String[] args) {
             Animal myDog = new Dog(); // 父类引用指向子类对象
             Animal myCat = new Cat(); // 父类引用指向子类对象
             
             myDog.sound();
             myCat.sound();
         }
     }
     ```

     ```java
     // 接口多态
     interface Animal() {
         void makeSound();
     }
     class Dog implements Animal() {
         public void makeSound() {
             System.out.println("汪汪叫...");
         }
     }
     class Cat implements Animal {
         public void makeSound() {
             System.out.println("喵喵叫...");
         }
     }
     public class InterfaceDemo {
         public static void main(String[] args) {
             // 使用接口类型的引用来调用这些类的方法
             Animal dogInstance = new Dog();
             Animal catInstance = new Cat();
             
             dogInstance.makeSound();
             catInstance.makeSound();
         }
     }
     ```



**面向对象设计原则**

- 单一职责：一个类只负责一项职责

- 开放封闭：对拓展开放，对修改封闭

- 里氏替换：子类对象应能够替换所有父类对象

- 接口隔离：通过接口抽象层实现底层和高层模块的高度解耦，比如依赖注入

- 依赖倒置：高层模块不依赖于低层模块，应依赖于抽象(接口，抽象类)

- 最少知识：一个对象应对与其它对象有最少的了解。**不要暴露太多内部细节，只提供必要的高层接口**

  ```java
  // 抽象接口
  interface Isender {
      void send(String recipient, String content);
  }
  
  // 实现类
  class EmailSender implements ISender {
      @Override
      public void send(String recipient, String content) {
          System.out.println("✅ 使用 Email 发送通知到 " + recipient);
      }
  }
  
  // 实现类
  class SmsSender implements ISender {
      @Override
      public void send(String recipient, String content) {
          System.out.println("✅ 使用 SMS 短信发送通知到 " + recipient);
      }
  }
  // 高层模块
  class OrderService {
      private Isender sender; // 依赖于抽象
      // 依赖注入，构造方法
      public OrderService(Isender sender) {
          this.sender = sender;
      }
  
      public void placeOrder(String orderId) {
          System.out.println("订单 " + orderId + " 已创建。");
          // 调用接口方法
          sender.send("admin@example.com", "有新订单了！");
      }
  }
  
  // 运行示例
  public class Main {
      public static void main(String[] args) {
          // 方案A
          Isender email = new EmailSender();
  
          OrderService service1 = new OrderService(email);
          service1.placeOrder("Order_001");
  
          System.out.println("-------------------------");
  
          // 方案B
          Isender sms = new SmsSender();
          OrderService service2 = new OrderService(sms);
          service2.placeOrder("Order_002");
      }
  }
  ```



**普通类和抽象类区别**

- 实例化：普通类可以实例化，抽象类只能被继承

- 方法实现：普通类只能包含具体实现的方法，抽象类可以包含抽象方法

- 继承：普通类和抽象类都具有单继承多接口实现的功能

  ```java
  interface Driveable {
      void drive();
  }
  interface GPSCapable {
      void showLocaltion();
  }
  class Vehicle {
  	void startEngine() {
          System.out.println("引擎启动");
      }
  }
  class Car extends Vehicle implements Driveable, GPSCapable {
      @Override
      public void drive() {
          System.out.println("正在驾驶汽车");
      }
  
      @Override
      public void showLocation() {
          System.out.println("显示GPS位置");
      }
  }
  ```

- 实现限制：普通类可以被其它类继承使用，抽象类一般作为基类，被其它类继承和拓展（不能加final）



**接口里面方法**

- 方法：抽象方法，默认方法，静态方法，私有方法

  ```java
  public interface Animal {
      void makesound(); // 抽象方法
      
      default void sleep() {
          System.out.println("sleeping...");
  	} // 默认方法
      
      static void staticMethod() {
          System.out.println("Static method in interface");
      } // 静态方法
      
      private void logsleep() {
          System.out.println("Logging sleep");
      } // 私有方法
  }
  ```

- **抽象方法：定义核心业务契约**

  ```java
  public interface UserService {
      // 核心业务：根据ID获取用户信息，每个实现类
      User getUserById(int id);
      // 核心业务：保存用户
      void saveUser(User user);
  }
  ```

- **默认方法：平滑扩展功能**

  ```java
  // 需要给接口增加新功能，但又不希望破坏所有已经实现了该接口的旧代码
  public interface UserService {
    	// 抽象方法...
      // 默认实现了一个软删除逻辑，所有现有实现类自动获得这个能力
      default void deactivateUser(int userId) {
          System.out.println("执行默认的用户注销逻辑...");
          User user = getUserById(userId);
          if (user != null) {
              user.setActive(false);
              saveUser(user); // 内部调用了抽象方法
          }
      }
  }
  ```

- **静态方法：提供工具或辅助功能**

  ```java
  public interface UserService {
      // ... (抽象方法和默认方法略) ...
  
      // 提供一个通用的电子邮件格式验证工具，不依赖任何 UserService 实例
      static boolean isValidEmail(String email) {
          // 实际的正则表达式验证逻辑
          return email != null && email.contains("@");
      }
  }
  ```

- **私有方法：接口内部代码复用**

  ```java
  public interface UserService {
      // ... (抽象方法、默认方法、静态方法略) ...
  
      default void setupNewUser(User user) {
          validateUserData(user); // 调用私有方法进行通用验证
          // ... 其他设置逻辑
      }
  
      default void updateUserProfile(User user) {
          validateUserData(user); // 再次调用私有方法进行通用验证
          // ... 其他更新逻辑
      }
  
      // 私有方法：将通用的验证逻辑封装起来，不对外部暴露
      private void validateUserData(User user) {
          if (user == null || user.getName() == null || !isValidEmail(user.getEmail())) {
              throw new IllegalArgumentException("用户数据无效");
          }
      }
  }
  ```

  

**静态成员（变量和方法）与内部类**

- **静态成员（`static`）**：
  - 属于**类**本身，而非某个对象。
  - **静态变量**用于所有对象**共享**的数据（如计数器、常量）。
  - **静态方法**用于不依赖对象状态的**工具函数**。
- **内部类**：
  - **非静态内部类**：依赖外部类的实例，可以直接访问外部类的**所有成员**。
  - **静态内部类**：独立于外部类实例存在，**只能**直接访问外部类的**静态成员**



**final作用**

1. 修饰变量：值不能被改变
2. 修饰方法：禁止子类重写
3. 修饰类：禁止被继承



**泛型**

允许在定义类、接口和方法时，不预先指定具体的数据类型，而是使用一个类型参数（就像一个占位符 `T`）来代表类型

```java
public class Box<T> {
    private T item;

    public void setItem(T item) {
        this.item = item;
    }

    public T getItem() {
        return item;
    }
}

```



**反射**

- 对于任意一个类，都能够知道这个类中的所有属性和方法
- 对于任意一个对象，都能够调用它的任意一个方法和属性
- 这种动态获取的信息以及动态调用对象的方法的功能称为**反射**



**注解**

**注解**是一种特殊**标记**（标签），用来给代码添加**额外信息**，告诉**编译器**或**框架**（利用反射技术）在特定情况下（比如编译时或运行时）要执行某些**特殊操作**。



**异常**

![img](https://cdn.xiaolincoding.com//picgo/1720683900898-1d0ce69d-4b5d-41a6-a5df-022e42f8f4c5.webp)

**要点**

- **`Error` (错误)：** 是严重且程序无法处理的问题（如内存不足），通常不用管它。
- **`Exception` (异常)：** 是程序可以预见并处理的情况。
- **运行时异常 (非受检)：** 逻辑错误导致（如空指针），**不需要强制**处理。
- **非运行时异常 (受检)：** 外部因素导致（如文件丢失），**必须强制**处理（要么 `try-catch`，要么 `throws`）

**异常处理**

1. `try-catch`

   ```java
   try {
   	// 可能抛出异常的代码
   } catch (ExceptionType1 e1) {
   	// 处理异常类型1的逻辑
   } catch (ExceptionType2 e2) {
   	// 处理异常类型2的逻辑
   } finally {
   	// 用于定义无论是否发生异常都会执行的代码块
   }
   ```

2. `throw和throws`

   - **`throw`** 是一个**动作**，用于**主动抛出**一个具体的异常**对象**
   - **`throws`** 是一个**声明**，用于在方法签名上**警告**调用者该方法可能会抛出某些异常**类型**

   
