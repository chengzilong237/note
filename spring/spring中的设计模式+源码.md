

[TOC]



####  设计模式

- 工厂模式（factory）

  - 简单工厂模式（基本不用）

    ````java
    /**
     * 简单工厂模式
     */
    public class SimpleFactory {
    
        public Milk getMilk(String name) {
    
            if ("蒙牛".equals(name)) {
                return  new Meniu();
            } else if ("伊利".equals(name)) {
                return new Yili();
            }
            return null;
        }
    ````

    缺点：不易扩展，需要修改源码（不符合开闭原则），且使用起来比较麻烦

  - 方法工厂模式

    ```java
    /**
     * 方法工厂
     */
    public interface FunctionFactory {
    
    
        Milk getMilk();
    }
    /**
    *方法工厂的实现类
    */
    public class MengniuFactoryImpl implements FunctionFactory {
        @Override
        public Milk getMilk() {
            return new Meniu();
        }
    }
    ```

    

  - 抽象工厂模式（最常用）

    ````java
    
    /**
     * 抽象工厂
     */
    public abstract class AbstractFactory {
    
        abstract Milk getMengNiu();
    
        abstract Milk getYili();
    
    
    }
    
    public class Factory extends AbstractFactory {
        @Override
        Milk getMengNiu() {
            return new Meniu();
        }
    
        @Override
        Milk getYili() {
            return new Yili();
        }
    }
    
    
    ````

    

- 单例模式（singleton）

  - 恶汉式

    ````java
    /**
     * 饿汉式
     */
    public class Hungry {
    	/*私有化构造器*/	
        private Hungry(){}
        private final static Hungry HUNGRY = new Hungry();
    
        public static Hungry getHungry(){
            return HUNGRY;
        }
    }
    
    ````

    优点：绝对线程安全。

    缺点：预先创建出对象，浪费资源。

  - 懒汉式

    在调用时才创建对象，且只创建一个

    ````java
    public class LazyOne {
    
        private LazyOne(){}
    
        private static LazyOne lazyOne=null;
    
        public static  LazyOne getInstance(){
            if(null== lazyOne){
                lazyOne=new LazyOne();
            }
            return lazyOne;
        }
    
    }
    ````

    以上的创建方式存在线程不安全的情况

    test代码如下：

    ````java
        public static void main(String[] args) {
            int count = 2000;
            CountDownLatch latch = new CountDownLatch(count);
    
            for (int i = 0; i < count; i++) {
                Thread thread = new Thread(() -> {
                    latch.countDown();
                    try {
                        latch.await();
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                    LazyOne lazyOne = LazyOne.getInstance();
                    System.out.println(System.currentTimeMillis()+":"+lazyOne);
    
    
                });
                thread.start();
            }
        } 
    ````

    test输出如下:高并发情况下会返回多个实例  :

      `1532875114253:com.czl.factory.lazy.LazyOne@ee2a9e`

    ````
    1532875114253:com.czl.factory.lazy.LazyOne@ee2a9e
    1532875114253:com.czl.factory.lazy.LazyOne@ee6f75
    1532875114253:com.czl.factory.lazy.LazyOne@ee6f75
    1532875114253:com.czl.factory.lazy.LazyOne@ee6f75
    1532875114253:com.czl.factory.lazy.LazyOne@190fd7a
    1532875114253:com.czl.factory.lazy.LazyOne@ee6f75
    1532875114253:com.czl.factory.lazy.LazyOne@ee6f75
    ````

    优化代码如下

    - 优化方式一 ：方法体加 `synchronized`修饰

      ```java
      public class LazyOneSync {
      
          private LazyOneSync(){}
      
          private static LazyOneSync lazyOne=null;
      
          public static synchronized  LazyOneSync getInstance(){
              if(null== lazyOne){
                  lazyOne=new LazyOneSync();
              }
              return lazyOne;
          }
      }
      ```

      解决了高并发下返回多个实例的问题，但是性能有很大的下降

      比较一下加`synchronized`和不加`synchronized`的性能，比较代码如下:

      ```java
      
      /**
       * 懒汉模式的性能测试
       */
      public class LazyOneSyncTest {
      
          public static void main(String[] args) {
             Long begin  = System.currentTimeMillis();
              lazyOneTest();
              Long end = System.currentTimeMillis();
              System.out.println("lazyOne:"+(end-begin));
             
              Long begin1  = System.currentTimeMillis();
              lazyOneSyncTest();
             	Long end1  = System.currentTimeMillis();
              System.out.println("lazyOneSync:"+(end1-begin1));
      
          }
      
          static void lazyOneTest(){
              for (int i = 0; i < 3000000; i++) {
                      LazyOne lazyOne = LazyOne.getInstance();
              }
      
          }
      
          static void lazyOneSyncTest(){
              for (int i = 0; i < 3000000; i++) {
                  LazyOneSync lazyOne = LazyOneSync.getInstance();
              }
          }
      }
      ```

      输出：

      ```java
      lazyOne:8
      lazyOneSync:89
      ```

      - 优化方式二：内部类方式 （利用内部类的初始化特性，当父类被调用时初始化一次）

        ```java
        //特点：在外部类被调用的时候内部类才会被加载
            //内部类一定是要在方法调用之前初始化
            //巧妙地避免了线程安全问题
        
            //这种形式兼顾饿汉式的内存浪费，也兼顾synchronized性能问题
            //完美地屏蔽了这两个缺点
            //史上最牛B的单例模式的实现方式
        public class LazyTwo {
        	//私有化构造器
            private LazyTwo(){}
            //static final 修饰 子类无法重写
            public static final LazyTwo getInstance(){
                return Init.LAZYTWO;
            }
            //在父类实例化的时候加载一次
            private static class Init {
              private  static final  LazyTwo LAZYTWO = new LazyTwo();
            }
        }
        
        ```

  - 注册登记式

    spring bean的单例实现模式

    ```java
    //Spring中的做法，就是用这种注册式单例
    public class BeanFactory {
    
        private BeanFactory(){}
    
        //线程安全
        private static Map<String,Object> ioc = new ConcurrentHashMap<String,Object>();
    
        public static Object getBean(String className){
    
            if(!ioc.containsKey(className)){
                Object obj = null;
                try {
                    obj = Class.forName(className).newInstance();
                    ioc.put(className,obj);
                } catch (Exception e) {
                    e.printStackTrace();
                }
                return obj;
            }else{
                return ioc.get(className);
            }
    
        }
    
    
    }
    ```

    

  - 反序列化导致单例失效的处理办法

- 原型模式 (prototype)

- 代理模式（proxy）

  代理只参与某一部分功能

  实现方式，jdk proxy 想要实现接口

  ​		   Cglib 、AspectJ ，asm

  ​		

- 策略模式

  固定算法封装，例如支付方式的封装

- 模板方法（通常又叫做模板方法模式 Template Method）

  饮料的加工流程：添加原料，加水，加热，混合

  `JdbcTemplate`手写

  ```java
  /*抽象类*/
  public abstract class Template{
      public  List<?> executeQuery(String sql,Object[]values){
          
          try{
              //1.获取连接
              Connection conn = dataSource.getConnection();
              //2.创建语句集
              PreparedStatement pstmt = conn.prepareStatement();
              //3.执行语句集，并且获得结果集
              ResultSet rs = pstmt.excuteQuery();
              //4.解析语句集
              List<?> result = processResult(rs);
              //5.关闭结果集
              rs.close();
              //6.关闭语句集
              pstmt.close;
              //7.关闭连接
              conn.close;
          }catch(Exception e){
              e.printStackTrace();
              return null;
          }
          
          return null;
      }
      
      public abstract Object processResult(ResultSet rs);
  }
  
  ```

  - 委派模式（Delegate） 代理模式的特殊情况 全权代理

    应用场景：项目经理，Dispatcher

    spring 中 以delegate 和dispatcher 结尾的类都是委派模式

    - `DispatcherServlet`

- 适配器模式 adapter

  ​	兼容、转换

   - 应用场景 : 一拖三充电头 hdmi转vga 编码和解码
   - 代码场景：

 - 装饰器模式

    在Spring 中 只要是Derocator wrapper 结尾的都是

 - 观察者模式

   ​	针对目标对象的一举一动，要得到一个反馈

   - 应用场景：日志监听，数据源，日志收集、短信通知、邮件通知 等等

   	​			   

   

#### Spring源码 

- IOC容器的初始化过程（ClassPathXmlApplicationContext）xml加载的配置文件(定位->加载->注册)

  

  ```java
  
  	public static void main(String[] args) {
  		//Spring框架的启动入口 （xml方式）
  		ClassPathXmlApplicationContext applicationContext = 
  				new ClassPathXmlApplicationContext("application-mvc.xml","applicatoin-spring.xml");
  		//启动完成之后通过 applicatonContext 上下文 获得Spring注册的Bean
  		applicationContext.getBean("member");
  	}
  ```

  xml方式加载情况下 Spring的IOC容器的初始化过程

  - 1定位过程

  ​	1、ClassPathXmlApplicationContext 构造器

  ```java
   /**
   构造器调用了refresh方法 refresh方法封装了SpringIOC容器的整个构造过程
   */
  	public ClassPathXmlApplicationContext(
  			String[] configLocations, boolean refresh, @Nullable ApplicationContext parent)
  			throws BeansException {
  
  		super(parent);
  		setConfigLocations(configLocations);
  		if (refresh) {
  			refresh();//refresh方法封装了SpringIOC容器的整个构造过程
  		}
  	}
  ```

  2、`org.springframework.context.support.AbstractApplicationContext`的refresh方法构成  调用obtainFreshBeanFactory()方法，此方法的作用是得到刷新（载入完成？）的bean

  ```java
  public void refresh() throws BeansException, IllegalStateException {
  		synchronized (this.startupShutdownMonitor) {
  			// Prepare this context for refreshing.
  			//调用容器准备刷新的方法，获取容器的当时时间，同时给容器设置同步标识
  			prepareRefresh();
  
  			// Tell the subclass to refresh the internal bean factory.
  			//告诉子类启动refreshBeanFactory()方法，Bean定义资源文件的载入从
  			//子类的refreshBeanFactory()方法启动
  			ConfigurableListableBeanFactory beanFactory = obtainFreshBeanFactory();
  
  			// Prepare the bean factory for use in this context.
  			//为BeanFactory配置容器特性，例如类加载器、事件处理器等
  			prepareBeanFactory(beanFactory);
  
  			try {
  				// Allows post-processing of the bean factory in context subclasses.
  				//为容器的某些子类指定特殊的BeanPost事件处理器
  				postProcessBeanFactory(beanFactory);
  
  				// Invoke factory processors registered as beans in the context.
  				//调用所有注册的BeanFactoryPostProcessor的Bean
  				invokeBeanFactoryPostProcessors(beanFactory);
  
  				// Register bean processors that intercept bean creation.
  				//为BeanFactory注册BeanPost事件处理器.
  				//BeanPostProcessor是Bean后置处理器，用于监听容器触发的事件
  				registerBeanPostProcessors(beanFactory);
  
  				// Initialize message source for this context.
  				//初始化信息源，和国际化相关.
  				initMessageSource();
  
  				// Initialize event multicaster for this context.
  				//初始化容器事件传播器.
  				initApplicationEventMulticaster();
  
  				// Initialize other special beans in specific context subclasses.
  				//调用子类的某些特殊Bean初始化方法
  				onRefresh();
  
  				// Check for listener beans and register them.
  				//为事件传播器注册事件监听器.
  				registerListeners();
  
  				// Instantiate all remaining (non-lazy-init) singletons.
  				//初始化所有剩余的单例Bean
  				finishBeanFactoryInitialization(beanFactory);
  
  				// Last step: publish corresponding event.
  				//初始化容器的生命周期事件处理器，并发布容器的生命周期事件
  				finishRefresh();
  			}
  
  			catch (BeansException ex) {
  				if (logger.isWarnEnabled()) {
  					logger.warn("Exception encountered during context initialization - " +
  							"cancelling refresh attempt: " + ex);
  				}
  
  				// Destroy already created singletons to avoid dangling resources.
  				//销毁已创建的Bean
  				destroyBeans();
  
  				// Reset 'active' flag.
  				//取消refresh操作，重置容器的同步标识.
  				cancelRefresh(ex);
  
  				// Propagate exception to caller.
  				throw ex;
  			}
  
  			finally {
  				// Reset common introspection caches in Spring's core, since we
  				// might not ever need metadata for singleton beans anymore...
  				resetCommonCaches();
  			}
  		}
  	}
  ```

  3、`org.springframework.context.support.AbstractApplicationContext`obtainFreshBeanFactory()方法中 refreshBeanFactory()Bean定义资源文件的载入refreshBeanFactory()方法启动

  ```java
  /**
  	 * Tell the subclass to refresh the internal bean factory.
  	 * @return the fresh BeanFactory instance
  	 * @see #refreshBeanFactory()
  	 * @see #getBeanFactory()
  	 */
  	protected ConfigurableListableBeanFactory obtainFreshBeanFactory() {
  		//这里使用了委派设计模式，父类定义了抽象的refreshBeanFactory()方法，具体实现调用子类容器的refreshBeanFactory()方法
  		refreshBeanFactory();//此方法启动资源文件的加载
  		ConfigurableListableBeanFactory beanFactory = getBeanFactory();
  		if (logger.isDebugEnabled()) {
  			logger.debug("Bean factory for " + getDisplayName() + ": " + beanFactory);
  		}
  		return beanFactory;
  	}
  ```

  4、`refreshBeanFactory();//此方法启动资源文件的加载`加载逻辑如下

  ```java
  @Override
  	protected final void refreshBeanFactory() throws BeansException {
  		//如果已经有容器，销毁容器中的bean，关闭容器
  		if (hasBeanFactory()) {
  			destroyBeans();
  			closeBeanFactory();
  		}
  		try {
  			//创建IOC容器
  			DefaultListableBeanFactory beanFactory = createBeanFactory();
  			beanFactory.setSerializationId(getId());
  			//对IOC容器进行定制化，如设置启动参数，开启注解的自动装配等
  			customizeBeanFactory(beanFactory);
  			//调用载入Bean定义的方法，主要这里又使用了一个委派模式，在当前类中只定义了抽象的loadBeanDefinitions方法，具体的实现调用子类容器
  			loadBeanDefinitions(beanFactory);//载入bean定义
  			synchronized (this.beanFactoryMonitor) {
  				this.beanFactory = beanFactory;
  			}
  		}
  		catch (IOException ex) {
  			throw new ApplicationContextException("I/O error parsing bean definition source for " + getDisplayName(), ex);
  		}
  	}
  ```

  5、`org.springframework.context.support.AbstractXmlApplicationContext`中的`loadBeanDefinitions()`载入Bean定义方法

  ```java
  //实现父类抽象的载入Bean定义方法
  	@Override
  	protected void loadBeanDefinitions(DefaultListableBeanFactory beanFactory) throws BeansException, IOException {
  		// Create a new XmlBeanDefinitionReader for the given BeanFactory.
  		//创建XmlBeanDefinitionReader，即创建Bean读取器，并通过回调设置到容器中去，容  器使用该读取器读取Bean定义资源
  		XmlBeanDefinitionReader beanDefinitionReader = new XmlBeanDefinitionReader(beanFactory);
  
  		// Configure the bean definition reader with this context's
  		// resource loading environment.
  		//为Bean读取器设置Spring资源加载器，AbstractXmlApplicationContext的
  		//祖先父类AbstractApplicationContext继承DefaultResourceLoader，因此，容器本身也是一个资源加载器
  		beanDefinitionReader.setEnvironment(this.getEnvironment());
  		beanDefinitionReader.setResourceLoader(this);
  		//为Bean读取器设置SAX xml解析器
  		beanDefinitionReader.setEntityResolver(new ResourceEntityResolver(this));
  
  		// Allow a subclass to provide custom initialization of the reader,
  		// then proceed with actually loading the bean definitions.
  		//当Bean读取器读取Bean定义的Xml资源文件时，启用Xml的校验机制
  		initBeanDefinitionReader(beanDefinitionReader);
  		//Bean读取器真正实现加载的方法
  		loadBeanDefinitions(beanDefinitionReader);//调用重载方法
  	}
  //重载方法
  //Xml Bean读取器加载Bean定义资源
  	protected void loadBeanDefinitions(XmlBeanDefinitionReader reader) throws BeansException, IOException {
  		//获取Bean定义资源的定位
  		Resource[] configResources = getConfigResources();
  		if (configResources != null) {
  			//Xml Bean读取器调用其父类AbstractBeanDefinitionReader读取定位
  			//的Bean定义资源
  			reader.loadBeanDefinitions(configResources);
  		}
  		//如果子类中获取的Bean定义资源定位为空，则获取FileSystemXmlApplicationContext构造方法中setConfigLocations方法设置的资源
  		String[] configLocations = getConfigLocations();
  		if (configLocations != null) {
  			//Xml Bean读取器调用其父类AbstractBeanDefinitionReader读取定位
  			//的Bean定义资源
  			reader.loadBeanDefinitions(configLocations);
  		}
  	}
  ```

  

  6、调用`org.springframework.beans.factory.xml.XmlBeanDefinitionReader`中的`loadBeanDefinitions()`方法

  ```java
  org.springframework.beans.factory.xml.XmlBeanDefinitionReader#loadBeanDefinitions(org.springframework.core.io.support.EncodedResource)
      
      //这里是载入XML形式Bean定义资源文件方法
  	public int loadBeanDefinitions(EncodedResource encodedResource) throws BeanDefinitionStoreException {
  		Assert.notNull(encodedResource, "EncodedResource must not be null");
  		if (logger.isInfoEnabled()) {
  			logger.info("Loading XML bean definitions from " + encodedResource.getResource());
  		}
  
  		Set<EncodedResource> currentResources = this.resourcesCurrentlyBeingLoaded.get();
  		if (currentResources == null) {
  			currentResources = new HashSet<>(4);
  			this.resourcesCurrentlyBeingLoaded.set(currentResources);
  		}
  		if (!currentResources.add(encodedResource)) {
  			throw new BeanDefinitionStoreException(
  					"Detected cyclic loading of " + encodedResource + " - check your import definitions!");
  		}
  		try {
  			//将资源文件转为InputStream的IO流
  			InputStream inputStream = encodedResource.getResource().getInputStream();
  			try {
  				//从InputStream中得到XML的解析源
  				InputSource inputSource = new InputSource(inputStream);
  				if (encodedResource.getEncoding() != null) {
  					inputSource.setEncoding(encodedResource.getEncoding());
  				}
  				//这里是具体的读取过程
  				return doLoadBeanDefinitions(inputSource, encodedResource.getResource());
  			}
  			finally {
  				//关闭从Resource中得到的IO流
  				inputStream.close();
  			}
  		}
  		catch (IOException ex) {
  			throw new BeanDefinitionStoreException(
  					"IOException parsing XML document from " + encodedResource.getResource(), ex);
  		}
  		finally {
  			currentResources.remove(encodedResource);
  			if (currentResources.isEmpty()) {
  				this.resourcesCurrentlyBeingLoaded.remove();
  			}
  		}
  	}
  
  
  org.springframework.beans.factory.xml.XmlBeanDefinitionReader#doLoadBeanDefinitions
  //从特定XML文件中实际载入Bean定义资源的方法
  	protected int doLoadBeanDefinitions(InputSource inputSource, Resource resource)
  			throws BeanDefinitionStoreException {
  		try {
  			//将XML文件转换为DOM对象，解析过程由documentLoader实现
  			Document doc = doLoadDocument(inputSource, resource);
  			//这里是启动对Bean定义解析的详细过程，该解析过程会用到Spring的Bean配置规则
  			return registerBeanDefinitions(doc, resource);
  		}
  		catch (BeanDefinitionStoreException ex) {
  			throw ex;
  		}
  		catch (SAXParseException ex) {
  			throw new XmlBeanDefinitionStoreException(resource.getDescription(),
  					"Line " + ex.getLineNumber() + " in XML document from " + resource + " is invalid", ex);
  		}
  		catch (SAXException ex) {
  			throw new XmlBeanDefinitionStoreException(resource.getDescription(),
  					"XML document from " + resource + " is invalid", ex);
  		}
  		catch (ParserConfigurationException ex) {
  			throw new BeanDefinitionStoreException(resource.getDescription(),
  					"Parser configuration exception parsing XML from " + resource, ex);
  		}
  		catch (IOException ex) {
  			throw new BeanDefinitionStoreException(resource.getDescription(),
  					"IOException parsing XML document from " + resource, ex);
  		}
  		catch (Throwable ex) {
  			throw new BeanDefinitionStoreException(resource.getDescription(),
  					"Unexpected exception parsing XML document from " + resource, ex);
  		}
  	}
  ```

  





  

 





