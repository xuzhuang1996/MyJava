[TOC]

## 概要

[一切框架](https://www.ibm.com/developerworks/cn/java/j-lo-spring-principle/)，都是构建一个数据结构，然后根据这个数据结构设计他的生存环境，并让它在这个环境中按照一定的规律在不停的运动，在它们的不停运动中设计一系列与环境或者与其他个体完成信息交换。

## 1. ApplicationContext

1. 作用：核心方法是 `refresh()` 方法，用于从资源文件加载类定义、扩展容器的功能。

2. 例子：

   ```java
   // 早期读取xml配置的bean数据
   ApplicationContext context = new ClassPathXmlApplicationContext("classpath:demo.xml");
   // 现在通过注解配置bean
   AnnotationConfigApplicationContext applicationContext = new AnnotationConfigApplicationContext(DemoApplication.class);
   ```

3. 继承关系：

   ![1](https://javadoop.com/blogimages/spring-context/1.png)

   ### 1.1 AbstractApplicationContext的refresh流程

   1. 准备工作，记录下容器的启动时间、标记“已启动”状态、处理配置文件中的占位符。

      ```java
      prepareRefresh();
      ```

   2. 创建 Bean 容器，加载并注册 Bean。即，将配置文件解析成一个个 Bean 定义，注册到 `BeanFactory` 中。

      ```java
      ConfigurableListableBeanFactory beanFactory = obtainFreshBeanFactory();
      ```

      - BeanFactory继承关系。

        ![3](https://javadoop.com/blogimages/spring-context/3.png)
   
      - 该流程的方法实现。

        ```java
        // AbstractApplicationContext.java
        	protected ConfigurableListableBeanFactory obtainFreshBeanFactory() {
        		refreshBeanFactory();
        		return getBeanFactory();
        	}
        ```

      - 简单工厂。实质是由一个工厂类根据传入的参数，动态决定应该创建哪一个产品类。包括：

        1. 各种的Aware接口，比如 `BeanFactoryAware`，对于实现了这些Aware接口的bean，在实例化bean时Spring会帮我们注入对应的`BeanFactory`的实例。
        
        2. `BeanPostProcessor`接口，实现了`BeanPostProcessor`接口的bean，在实例化bean时Spring会帮我们调用接口中的方法。
        
        3. `InitializingBean`接口，实现了`InitializingBean`接口的bean，在实例化bean时Spring会帮我们调用接口中的方法。
        
        4. `DisposableBean`接口，实现了`BeanPostProcessor`接口的bean，在该bean死亡时Spring会帮我们调用接口中的方法。
        
           > 通过Spring接口的暴露，在实例化bean的阶段我们可以进行一些额外的处理，这些额外的处理只需要让bean实现对应的接口即可，那么spring就会在bean的生命周期调用我们实现的接口来处理该bean。
        
      - 模板方法1。`AbstractApplicationContext`的`refreshBeanFactory`方法，以`ClassPathXmlApplicationContext`为例，该类最终也是继承了`AbstractRefreshableApplicationContext`的`refreshBeanFactory`方法。
   
        ```java
        // AbstractRefreshableApplicationContext.java
            @Override
        	protected final void refreshBeanFactory() throws BeansException {
        		if (hasBeanFactory()) {
        			destroyBeans();
        			closeBeanFactory();
        		}
        		try {
                 // 初始化DefaultListableBeanFactory，该类基本包含所有BeanFactory的实现
        			DefaultListableBeanFactory beanFactory = createBeanFactory();
      	 			// 设置 BeanFactory 的两个配置属性：是否允许 Bean 覆盖、是否允许循环引用
        			customizeBeanFactory(beanFactory);
                 // 加载 Bean 到 BeanFactory 中。该方法的命名很好，结合了参数。
        			loadBeanDefinitions(beanFactory);
        			this.beanFactory = beanFactory;
        		}
        		}
        	}
        
        ```
   
      - refresh方式的目的：destroy之前的，create新的。
   
      - 模板方法2。`AbstractRefreshableApplicationContext`的`loadBeanDefinitions`方法为抽象方法。这里的可扩展性在于，**未对加载方法进行要求，也就是可以从不同来源的不同类型的资源进行加载**。
   
        ```java
        // AbstractXmlApplicationContext.java
        	@Override
        	protected void loadBeanDefinitions(DefaultListableBeanFactory beanFactory) throws BeansException, IOException {
        		// 给这个 BeanFactory 实例化一个 XmlBeanDefinitionReader.
        		XmlBeanDefinitionReader beanDefinitionReader = new XmlBeanDefinitionReader(beanFactory);
                // 用初始化的 Reader 开始来加载 xml 配置
        		loadBeanDefinitions(beanDefinitionReader);
        	}
        ```
   
      - 模板方法3。模板方法模式和回调模式的结合，是Template Method不需要继承的一种实现方式。例如，JDBC的抽象和对Hibernate的集成，都采用了一种理念或者处理方式。[JdbcTemplate](https://blog.csdn.net/weixin_40001125/article/details/88538576)
   
        ```java
        // 采用模板方法模式是为了以一种统一而集中的方式来处理资源的获取和释放
        // JdbcTemplate是抽象类，不能够独立使用，我们每次进行数据访问的时候都要给出一个相应的子类实现,这样肯定不方便，所以就引入了回调 。
        public class JdbcTemplate {  
            public final Object execute（StatementCallback callback）{  
                Connection con=null;  
                Statement stmt=null;  
                try{  
                    con=getConnection（）;  
                    stmt=con.createStatement（）;  
                    // 回调，callback类型为StatementCallback
                    Object retValue=callback.doWithStatement(stmt);  
                    return retValue;  
                }catch（SQLException e）{  
                    ...  
                }finally{  
                    closeStatement（stmt）;  
                    releaseConnection（con）;  
                }  
            }  
        
            ...//其它方法定义  
        }   
        
        // 回调接口定义
        public interface StatementCallback{  
            Object doWithStatement（Statement stmt）;  
        }
        
        // 使用方法1。采用匿名类的方式，或者如果只有一个抽象方法，可以使用lambda表示式，如Runnable
        JdbcTemplate jdbcTemplate;  
        final String sql;  
        StatementCallback callback = new StatementCallback() {  
        	public Object doWithStatement(Statement stmt){  
                return ...;  
            }  
        }    
        jdbcTemplate.execute(callback);
        
        // 使用方法2
        public void test() {
                String sql = "insert into test(name) values (?)";
                //返回的是更新的行数
                int count = jdbcTemplate.update(sql, new PreparedStatementSetter(){
                    @Override
                    public void setValues(PreparedStatement pstmt)
                            throws SQLException {
                        pstmt.setObject(1, "name4"); 
                    }
                });
        }
        ```
   
        
   
   3. 设置 `BeanFactory` 的类加载器，添加几个`BeanPostProcessor`，手动注册几个特殊的 bean。
   
      ```java
      prepareBeanFactory(beanFactory);
      ```
   
      - 该流程方法部分代码：
   
        ```java
        // 添加一个BeanPostProcessor。即添加调用者。
        // 如果Bean实现了Aware接口，初始化时processor负责回调。
        beanFactory.addBeanPostProcessor(new ApplicationContextAwareProcessor(this));
        
        // 添加一个BeanPostProcessor。注册事件监听器
        // bean实例化后，如果是ApplicationListener的子类，则将其添加到listener列表。
        beanFactory.addBeanPostProcessor(new ApplicationListenerDetector(this));
        ```
   
      - Spring中有很多继承于`aware`中的接口，这些接口是能感知到所有`Aware`前面的含义。通过实现的set方法，保存spring容器相应的引用值，给自己用。
        
        1. `BeanNameAware`。[来源](https://www.jianshu.com/p/c5c61c31080b)
   
   4. 当前步骤，是提供给子类的扩展点。与步骤5联动。
   
      > 如果配置的bean有实现`BeanFactoryPostProcessor`接口，那么在容器初始化以后，Spring 会负责调用一些bean实现的该接口里面的 `postProcessBeanFactory` 方法。
   
      ```java
      postProcessBeanFactory(beanFactory);
      ```
   
   5. 调用` BeanFactoryPostProcessor` 各个实现类的`postProcessBeanFactory(beanFactory)` 方法。
   
      ```java
      invokeBeanFactoryPostProcessors(beanFactory);
      ```
   
   6. 将所有实现`BeanPostProcessor`接口的bean注册到`BeanFactory` 维护的 `BeanPostProcessor` 列表。
   
      > `BeanPostProcessor`接口两个方法: `postProcessBeforeInitialization `和 `postProcessAfterInitialization`。两个方法分别在 Bean 初始化之前和初始化之后得到执行。
   
      ```java
      registerBeanPostProcessors(beanFactory);
      ```
   
   7. 注册事件监听器，监听器需要实现 ApplicationListener 接口。
   
      ```java
      registerListeners();
      ```
   
   8. 到这一步，才是初始化所有的 singleton beans（lazy-init 的除外）。
   
      ```java
      finishBeanFactoryInitialization(beanFactory);
      ```
   
      1. 该步骤部分代码：
   
         ```java
         	protected void finishBeanFactoryInitialization(ConfigurableListableBeanFactory beanFactory) {
         		// 实例化所有剩余的singletons(non-lazy-init) .
         		beanFactory.preInstantiateSingletons();
         	}
         ```
   
      2. `ConfigurableListableBeanFactory`接口。提供bean definition的解析,注册功能,再对单例来个预加载(解决循环依赖问题)。TODO。
   
      3. `FactoryBean`接口。当bean实现了`FactoryBean`接口，spring会在使用`getBean()`调用获得该bean时，自动调用该bean的`getObject()`方法，所以**返回的不是factory这个bean，而是这个`bean.getOjbect()`方法的返回值。**适用于 Bean 的创建过程比较复杂的场景，比如数据库连接池的创建。
   
         - 工厂方法。例子：spring与mybatis的结合。由于实现了`FactoryBean`接口，所以返回的不是 `SqlSessionFactoryBean`的实例，而是她的 `SqlSessionFactoryBean.getObject()` 的返回值。这样就相当于将自己注册到了spring容器中，sqlSessionFactory存储的是具体的bean。
   
           ```java
           <bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
           	<property name="dataSource" ref="dataSource" />
           	<property name="mapperLocations" value="classpath:mybatis-mapper/*.xml"/>
           </bean>
           ```
   
         - 单例模式。这里只是顺带提一嘴。从效率角度考虑，使用temp临时变量，是因为这里临时变量从工作内存中读取，效率更高；而volatile变量从主存中读取，效率更低。使用后，可以减少读取主存的次数。
   
           ```java
           public class SingleTon3 {
                    private SingleTon3(){};             //私有化构造方法
           
                    private static volatile SingleTon3 singleTon = null;
           
                    public static SingleTon3 getInstance() {
                            SingleTon3 temp = singleTon;
                             //第一次校验
                            if(temp==null){     
                           	synchronized(SingleTon3.class){
                                   //第二次校验
                                   if(temp==null){     
                                    	singleTon=new SingleTon3();
                                   }
                             }
                	  }
                	 return temp;
           }
           ```
   
      4. `getBean`方法。
   
   ### 1.2 ApplicationContext各个抽象类的关系梳理
   
   1.  `ApplicationContext`。抽象接口。应该提供以下功能（来自接口文档）：
      - 访问Bean的能力。继承自`ListableBeanFactory`接口。
        1. 上图可以看出，该接口继承自`beanFactory`接口。对于`AbstractApplicationContext`而言，它将在自己类里实现`ListableBeanFactory`与`beanFactory`接口的抽象方法。
        2. 该接口功能是：[根据类型获取bean](https://blog.csdn.net/u013412772/article/details/80819314)。**该接口定义了访问容器中Bean基本信息的若干方法，如查看Bean的个数、获取某一类型Bean的配置名（Autowire注解的List注入功能实现）、查看容器中是否包括某一Bean等方法。**不再需要一个个bean地查找，提供容器中bean迭代的功能。在看SpringMVC时,扫描包路径下的具体实现策略就是使用的这种方式(那边使用的是`BeanFactoryUtils`封装的api)。// TODO
      - 通用的加载资源的能力。继承自`ResourceLoader`接口。
        1. `实现接口`的隔离：对于`AbstractApplicationContext`而言，尽管它继承了`ResourceLoader`接口的抽象方法，但是它只需要在继承`ResourceLoader`接口的实现类`DefaultResourceLoader`，就能够拥有该类的能力。常规思路是自己去实现`ResourceLoader`接口的抽象方法。不过该方式对于单继承的Java有点尴尬。用法不常见。
      - 将事件发布给注册的监听器的能力。继承自`ApplicationEventPublisher`接口。
      - 解析消息，支持国际化的能力。继承自`MessageSource`接口。
   2. `ConfigurableApplicationContext`接口。此接口除了继承`ApplicaitnContext`接口的能力外. 还具有可配置上下文与生命周期管理功能。避免暴露给`ApplicaitnContext`，仅在启动与关闭时调用。其中最重要的是定义了`refresh()`方法. `refresh()`功能是加载配置.
   3. `AbstractApplicationContext`。`ApplicationContext`的抽象实现类，实现了大部分的接口方法。`refresh()`方法的实现为`ApplicationContext`提供了加载配置的能力，包括加载bean。
   4. `AbstractRefreshableApplicationContext`。继承自`AbstractApplicationContext`。实现了`refreshBeanFactory`方法，完成了对容器底层的beanFactory的刷新。
   5. `AbstractRefreshableConfigApplicationContext`。用于添加对指定配置位置的通用处理。用作基于XML的应用程序上下文实现。例如`ClassPathXmlApplicationContext`、`FileSystemXmlApplicationContext`。
   
   ## 设计模式，这节会删，并到上面的流程中
   
   1. 装饰器模式。Spring中用到的包装器模式在类名上有两种表现：一种是类名中含有Wrapper，另一种是类名中含有Decorator。动态地给一个对象添加一些额外的职责。就增加功能来说，Decorator模式相比生成子类更为灵活。
   2. 代理模式。动态代理：在内存中构建的，不需要手动编写代理类。静态代理：需要手工编写代理类，代理类引用被代理对象。
   
   ## bean的生命周期
   
   ## 参考
   
   1. https://blog.csdn.net/nuomizhende45/article/details/81158383
   2. https://www.zybuluo.com/dugu9sword/note/382745
   3. https://blog.csdn.net/caoxiaohong1005/article/details/80039656
   4. https://www.cnblogs.com/smallstudent/p/11658518.html
   5. https://www.jianshu.com/p/8aaad9cff96b