## **前言**

springboot会通过@SpringBootApplication自动扫描配置类,自动导入和配置各个spring-boot-starter组件
通过调用SpringApplication.run(XXXApplication.class, args), 启动了spring项目, 
该方法执行了两个操作, 一是初始化了一个SpringApplication实例, 二是调用run方法

## **初始化SpringApplication实例**

**一.通过加载各个Web类型的容器类,判断当前模块web类型**
    ​         WebApplicationType.deduceFromClasspath方法, 获取Web类型

**二.加载Application初始化器**
    ​         getSpringFactoriesInstances()方法, 
    ​         调用SpringFactoriesLoader.loadFactoryNames, 
    ​         首先加载META-INF/spring.factories配置的类名到缓存中, 
    ​         之后实例化了ApplicationContextInitializer的子类列表, 
    ​         赋值给SpringApplication的initializers列表
​         
**三.加载Application监听器**
    ​         getSpringFactoriesInstances()方法, 
    ​         调用**SpringFactoriesLoader.loadFactoryNames**,
    ​         从SpringFactoriesLoader的缓存中获取并实例化ApplicationListener子类列表, 
    ​         赋值SpringApplication的listeners列表

**四.找到启动类**
             通过抛出一个RuntimeException, 
             遍历其堆栈信息, 
             获取到类名中包含main方法的类, 
             实例化这个类, 
             赋值给SpringApplication的mainApplicationClass对象



## **run方法：**

**一：发布启动事件ApplicationStartingEvent**
            1. 发布启动事件ApplicationStartingEvent
            调用getRunListeners()方法, 实例化一个SpringApplicationRunListeners对象, 
            SpringApplicationRunListeners的构造参数通过getSpringFactoriesInstances()方法获得
            调用SpringApplicationRunListeners对象的starting()方法, 发布SpringApplication启动事件 
                1.1 **SpringApplicationRunListeners**
                1.2 EventPublishingRunListener（用来发布springboot加载过程中的各个事件）
                1.3 SimpleApplicationEventMulticaster（用于发布spring启动过程中的各个事件）
            2. 启动事件的监听器
                2.1 LoggingApplicationListener日志监听器,配置日志
                2.2 BackgroundPreinitializer后台初始化器, 多线程加载耗时任务
                2.3 DelegatingApplicationListener代理监听器, 继续发布事件

**二：封装启动参数new DefaultApplicationArguments(args)**
            处理main函数的参数,将其封装为一个DefaultApplicationArguments对象,为接下来准备环境提供参数

**三：准备环境prepareEnvironment**
            1. prepareEnvironment()
            2. 获取或者创建环境getOrCreateEnvironment()
                2.1 AbstractEnvironment
                2.2 StandardEnvironment
                2.3 StandardServletEnvironment
            3. 配置环境configureEnvironment()
                 3.1 配置属性configurePropertySources
                 3.2 配置profile
            4. 发布ApplicationEnvironmentPreparedEvent事件
                 4.1 **ConfigFileApplicationListener**
                 4.2 AnsiOutputApplicationListener
                 4.3 LoggingApplicationListener
                 4.4 ClasspathLoggingApplicationListener
                 4.5 BackgroundPreinitializer
                 4.6 DelegatingApplicationListener
                 4.7 FileEncodingApplicationListener
            5. bindToSpringApplication绑定环境
            6. 环境转换EnvironmentConverter
            7. ConfigurationPropertySources.attach(environment)
            
**四：打印Banner printBanner()**
            1. 打印Banner printBanner()
                1.1 实例化ResourceLoader, 默认为null, 所以实例化一个DefaultResourceLoader
                1.2 实例化一个SpringApplicationBannerPrinter对象
                1.3 调用SpringApplicationBannerPrinter的print方法
            2. SpringApplicationBannerPrinter
                2.1 加载图片Banner 
                2.2 加载文本Banner
                2.3 图片和文本Banner都不存在的话,如果通过构造函数,传入了fallbackBanner,那么返回fallbackBanner
                2.4 图片, 文本和fallbackBanner都不存在, 那么返回默认的SpringBootBanner
                2.5 打印Banner, 返回一个PrintedBanner 

**五：创建应用上下文createApplicationContext()**
            1. web类型为SERVLET, 所以实例化了一个AnnotationConfigServletWebServerApplicationContext对象
            2. 实例化过程中会调用父类GenericApplicationContext构造函数
               去实例化一个DefaultListableBeanFactory, 作为Spring IOC容器 
            3. AnnotationConfigServletWebServerApplicationContext的构造函数
               实例化了AnnotatedBeanDefinitionReader对象, 用于读取Spring的bean定义
            4. 在实例化AnnotatedBeanDefinitionReader的过程中, 注册了几个bean, 用来处理相应的注解

**六：异常上报SpringBootExceptionReporter**
            FailureAnalyzers异常失败原因分析器实现SpringBootExceptionReporter接口
            获取并打印Spring启动过程中的异常信息

**七：准备环境prepareContext()**
            1. 统一ApplicationContext和Application使用的environment
            2. 后置处理ApplicationContext
                2.1 设置ApplicationContext的beanNameGenerator
                2.2 设置ApplicationContext的resourceLoader和classLoader
                2.3 设置ApplicationContext的类型转换Service
            3. 执行Initializers
                3.1 DelegatingApplicationContextInitializer
                3.2 SharedMetadataReaderFactoryContextInitializer
                3.3 ContextIdApplicationContextInitializer
                3.4 ConfigurationWarningsApplicationContextInitializer
                3.5 ServerPortInfoApplicationContextInitializer
                3.6 ConditionEvaluationReportLoggingListener
            4. 发布contextPrepared事件
                    发布ApplicationContextInitializedEvent事件,Application容器初始化完成事件, 
                    对该事件感兴趣的监听器有
                       4.1 BackgroundPreinitializer 后台进程初始化器, 用于多线程执行后台耗时任务
                       4.2 DelegatingApplicationListener 代理监听器, 继续分发事件
            5. 打印启动和profile日志
            6. 注册单例bean
                    注册了两个单例Bean
                       6.1 springApplicationArguments,命令行参数bean,值为applicationArgument
                       6.2 banner bean, 值为printedBanner
            7. 初始化BeanDefinitionLoader, 加载启动类
            8. 发布contextLoaded事件
            
**八：刷新应用上下文refreshContext**
            重点！不过这里基本上都是使用spring-context中的AbstractApplicationContext.refresh()
            没什么新东西，从spring工程中进去看注释，内容从context看到ioc

再经过发布了ApplicationStartedEvent, ApplicationReadyEvent事件, 以及启动失败异常处理, 发布启动失败事件
至此，springboot启动完成