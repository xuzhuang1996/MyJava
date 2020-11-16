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

      - 工厂模式。实质是由一个工厂类根据传入的参数，动态决定应该创建哪一个产品类。包括：

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
   
      3. `FactoryBean`。适用于 Bean 的创建过程比较复杂的场景，比如数据库连接池的创建。
   
      4. `getBean`方法。
   
   
   
   ## 参考
   
   1. https://blog.csdn.net/nuomizhende45/article/details/81158383
   2. https://www.zybuluo.com/dugu9sword/note/382745
   3. https://blog.csdn.net/caoxiaohong1005/article/details/80039656