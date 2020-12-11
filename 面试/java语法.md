## 泛型

1. `List<?>`、`List`、`List<Object>`等.

   ![1603096273929](C:\Users\x00574457\AppData\Local\Temp\1603096273929.png)

2. 泛型就是类型参数化, 好处是编译器会检查参数类型.多态就是多个类由继承(实现接口)得到的一致外观, 好处是简化代码.

## spring

### Autowired

1. @Autowired注解作用在方法上。

   1. 该方法如果有参数，会使用autowired的方式在spring容器中查找是否有该参数。
   2. 会执行该方法。因此，这个注解比较适合注册机制。目前看来更适合用于构造函数问题。

2. 作用在构造器上。底层注入流程就相当于是使用构造函数进行依赖注入了。

3. 作用在bean，相当于在配置文件中配置bean，并且使用setter注入。

4. Java变量的初始化顺序为：静态变量或静态语句块–>实例变量或初始化语句块–>构造方法–>@Autowired

5. ```java
   @RestController
   @RequestMapping("/user")
   public class UserController {
       // 这里不加Autowired也可以成功注入。其实是通过构造函数进行的设置注入。
       private final UserService userService;
   
       @Autowired
       public UserController(final UserService userService) {
           this.userService = userService;
       }
   
       @ResponseStatus(value = HttpStatus.NOT_FOUND, reason = "Resource not found")
       @ExceptionHandler(Exception.class)
       public void notFound() {
       }
   
       public static void main(String[] args) {
           new UserController(null);
       }
   }
   ```

## git

1. 比较当前分支，自己的修改与之前commit版本的区别

   ![1603529576323](C:\Users\x00574457\AppData\Local\Temp\1603529576323.png)

## 异常

1. NoClassDefDoundError。
   1. 文档：当 Java 虚拟机或 ClassLoader 实例试图在类的定义中加载（作为通常方法调用的一部分或者作为使用 new 表达式创建的新实例的一部分），但无法找到该类的定义时，抛出此异常。 当前执行的类被编译时，所搜索的类定义存在，但无法再找到该定义。
   2. NoClassDefFoundErr异常一般出现在我们编译环境和运行环境不一致的情况下，就是说我们有可能在编译过后更改了Classpath或者jar包所以导致在运行的过程中JVM或者ClassLoader无法找到这个类的定义。NoClassDefFoundError发生在编译时对应的类可用，而运行时在Java的classpath路径中，对应的类不可用导致的错误。

## effictive java

### 静态工厂方法

1. 适用于基于接口的框架。其中，接口为静态工厂方法提供恰当的返回类型。
2. 例如：OPCEServiceFactory产生实例，而OPCEServiceInf为接口。所有实例都实现了OPCEServiceInf接口。得益于**静态工厂方法返回对象的类可以根据输入参数的不同而不同**。使用这种静态工厂方法需要客户端通过接口而不是实现类来引用返回的对象，这通常是良好的实践。

3. 例如：服务提供者框架JDBC。
   1. 服务接口，它表示实现。即Connection
   2. 提供者注册 API，提供者用来注册实现。即DriverManager.registerDriver
   3. 服务访问 API，客户端使用该 API 获取服务的实例。DriverManager.getConnection
   4. 服务提供者接口，它描述了一个生成服务接口实例的工厂对象。在没有服务提供者接口的情况下，必须对实现进行反射实例化。即Driver接口。即`Class.forName("com.mysql.jdbc.Driver")`
4. 缺点：很难找到对应的实例。需要通过编码规范命名：
   - from —— 类型转换方法，它接受单个参数并返回此类型的相应实例，例如：`Date d = Date.from(instant);`。
   - create 或 newInstance —— 该方法保证每次调用返回一个新的实例，例如：`Object newArray = Array.newInstance(classObject, arrayLen)`;
   - 除非有令人信服的理由，否则不要提供独立于构造方法或静态工厂的公共初始化方法。

### builder模式

1. 优势：builder 的参数可以在构建方法的调用之间进行调整，以改变创建的对象。 builder 可以在创建对象时自动填充一些属性，例如每次创建对象时增加的序列号。
2. 劣势：builder 模式比伸缩构造方法模式更冗长，因此只有在有足够的参数时才值得使用它，比如四个或更多。

### 私有构造方法

1. 工具类（utility classes）不是设计用来被实例化的。因此提供一个私有构造方法。

### 依赖注入优于硬连接资源（hardwiring resources）

1. 许多类依赖于一个或多个底层资源。例如，拼写检查器依赖于字典。不要用单例和静态工具类来实现依赖一个或多个底层资源的类，且该资源的行为会影响到该类的行为；也不要直接用这个类来创建这些资源。而应该将这些资源或者工厂传给构造器（或者静态工厂，或者构建器），通过它们来创建类。这个实践就被称作依赖注人，它极大地提升了类的灵活性、可重用性和可测试性。

### 避免创建不必要的对象

1. 工厂方法 `Boolean.valueOf(String)` 比构造方法 `Boolean(String)` 更可取，后者在 Java 9 中被弃用。构造方法每次调用时都必须创建一个新对象，而工厂方法永远不需要这样做，在实践中也不需要。

### 使用try

1. try-with-resources。要使用这个构造，资源必须实现 `AutoCloseable` 接口。

2. 例如：

   ```java
   // try-with-resources with a catch clause
   static String firstLineOfFile(String path, String defaultVal) {
       try (BufferedReader br = new BufferedReader(
              new FileReader(path))) {
           return br.readLine();
       } catch (IOException e) {
           return defaultVal;
       }
   }
   ```

3. 由于底层物理设备发生故障，对 `readLine` 方法的调用可能会引发异常，并且由于相同的原因，调用 `close` 方法可能会失败。 在这种情况下，第二个异常完全冲掉了第一个异常。 在异常堆栈跟踪中没有第一个异常的记录，这可能使实际系统中的调试非常复杂——通常这是你想要诊断问题的第一个异常。

### 组合优于继承

1. 父类的实现可能会从发布版本不断变化，如果是这样，子类可能会被破坏，即使它的代码没有任何改变。 因此，一个子类必须与其超类一起更新而变化.
2. 存在继承的情况下，方法向超类方向集中，数据向子类方向集中

### 接口优于抽象类

1. 你可以通过提供一个抽象的骨架实现类（abstract skeletal implementation class）来与接口一起使用，将接口和抽象类的优点结合起来。 接口定义了类型，可能提供了一些默认的方法，而骨架实现类在原始接口方法的顶层实现了剩余的非原始接口方法。 继承骨架实现需要大部分的工作来实现一个接口。 这就是模板方法设计模式。

### 接口仅用来定义类型

1. 当类实现接口时，该接口作为一种类型（type），可以用来引用类的实例。因此，一个类实现了一个接口，因此表明客户端可以如何处理类的实例。**为其他目的定义接口是不合适的**。

## 架构

### 包结构

1. model层，分层领域模型。
2. helper层，用于提供底层的业务计算逻辑。Helper 依赖于特定的领域，即特定的业务。service里的每个方法只完成一个业务逻辑，如果业务逻辑比较复杂，可以考虑拆分成一个helper或者manager来专门处理这个复杂的逻辑，service就清清爽爽了，而不用一大堆的private方法在service里了。
3. 通用工具层与业务无关，任何系统，只要有需要都可以引用。
   1. 常量：分为属性名常量类、状态值常量类、普通字符串常量。
4. 

## 小工具

1. 界面输入密码繁琐。然后将其粘贴到书签栏，形成一个书签。

   ```javascript
   javascript:$('#name').val('admin');
   $('#pwd').val('123456');
   $('#submitDataverify').click();
   ```

2. 

## 参考

1. https://www.bookstack.cn/read/effective-java-3rd-chinese/docs-README.md
2. [郑教练博客](http://arganzheng.life/)
3. [分布式锁AOP](https://developer.ibm.com/zh/languages/spring/articles/j-spring-boot-aop-web-log-processing-and-distributed-locking/)