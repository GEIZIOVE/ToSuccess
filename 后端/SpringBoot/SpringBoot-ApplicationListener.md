  ApplicationContext事件机制是观察者设计模式的实现，通过ApplicationEvent类和ApplicationListener接口，可以实现ApplicationContext事件处理。

 如果容器中有一个`ApplicationListener Bean`，每当`ApplicationContext`发布`ApplicationEvent`时，ApplicationListener Bean将自动被触发。这种事件机制都必须需要程序显示的触发。

其中spring有一些内置的事件，当完成某种操作时会发出某些事件动作。比如监听`ContextRefreshedEvent`事件，当所有的bean都初始化完成并被成功装载后会触发该事件，实现`ApplicationListener<ContextRefreshedEvent>`接口可以收到监听动作，然后可以写自己的逻辑。

同样事件可以自定义、监听也可以自定义，完全根据自己的业务逻辑来处理。

# 一、事件监听器使用

## 1、自己实现一个事件监听器

首先创建一个自定义的事件监听器：

```java
public class MyApplicationListener implements ApplicationListener<ApplicationEvent> {
    @Override
    public void onApplicationEvent(ApplicationEvent applicationEvent) {
        System.out.println("======>MyApplicationListener: "+applicationEvent);
    }
}
```

然后还是在项目的spring.factories中配置监听器

```properties
org.springframework.context.ApplicationListener=\
com.roy.applicationListener.MyApplicationListener
```

然后配置启动类。在启动类中发布一个自己的事件。

```java
@SpringBootApplication
public class P1Application implements CommandLineRunner {
    public static void main(String[] args) {
        final SpringApplication application = new SpringApplication(P1Application.class);
//        application.addInitializers(new MyApplicationContextInitializer());
        application.run(args);
    }
    @Autowired
    private ApplicationContext applicationContext;

    @Override
    public void run(String... args) throws Exception {
        //自行发布一个事件。
        applicationContext.publishEvent(new ApplicationEvent("selfEvent") {
        });
    }
}
```

正常启动SpringBoot应用，就能打印出启动过程中的关键事件的日志。这里把关键的事件日志给整理出来：

```dart
======>MyApplicationListener: org.springframework.boot.context.event.ApplicationStartingEvent[source=org.springframework.boot.SpringApplication@60c6f5b]
======>MyApplicationListener: org.springframework.boot.context.event.ApplicationEnvironmentPreparedEvent[source=org.springframework.boot.SpringApplication@60c6f5b]
======>MyApplicationListener: org.springframework.boot.context.event.ApplicationContextInitializedEvent[source=org.springframework.boot.SpringApplication@60c6f5b]
======>MyApplicationListener: org.springframework.boot.context.event.ApplicationPreparedEvent[source=org.springframework.boot.SpringApplication@60c6f5b]
======>MyApplicationListener: org.springframework.context.event.ContextRefreshedEvent[source=org.springframework.context.annotation.AnnotationConfigApplicationContext@1d296da, started on Wed Apr 28 13:33:33 CST 2021]
======>MyApplicationListener: org.springframework.boot.context.event.ApplicationStartedEvent[source=org.springframework.boot.SpringApplication@60c6f5b]
======>MyApplicationListener: org.springframework.boot.availability.AvailabilityChangeEvent[source=org.springframework.context.annotation.AnnotationConfigApplicationContext@1d296da, started on Wed Apr 28 13:33:33 CST 2021]
======>MyApplicationListener: com.roy.P1Application$1[source=selfEvent]  ##！！自行发布的事件。
======>MyApplicationListener: org.springframework.boot.context.event.ApplicationReadyEvent[source=org.springframework.boot.SpringApplication@60c6f5b]
======>MyApplicationListener: org.springframework.boot.availability.AvailabilityChangeEvent[source=org.springframework.context.annotation.AnnotationConfigApplicationContext@1d296da, started on Wed Apr 28 13:33:33 CST 2021]
======>MyApplicationListener: org.springframework.context.event.ContextClosedEvent[source=org.springframework.context.annotation.AnnotationConfigApplicationContext@1d296da, started on Wed Apr 28 13:33:33 CST 2021]
```

这里就打印出了SpringBoot应用启动过程中的多个内部事件。实际上这多个内部事件也就对应了启动过程的各个阶段，是梳理SpringBoot启动流程非常好的入口。

> Spring事件机制的其他细节这里就不多说了，大家可以自行了解。我们这里还是专注于SpringBoot的部分。

## 2、事件监听器的其他配置方式：

这个事件监听机制是Spring非常重要的一个机制，有非常多的配置方式。除了上面提到的基于spring.factories文件配置的方式，还有其他几种配置方式。

### 2.1 SpringApplication.addListener

跟之前的Initializer一样，这个事件监听器也可以在SpringApplication中直接添加。

```java
@SpringBootApplication
public class P1Application implements CommandLineRunner {
    public static void main(String[] args) {
        final SpringApplication application = new SpringApplication(P1Application.class);
        //添加事件监听器
        application.addListeners(new MyApplicationListener());
        application.run(args);
    }
}
```

### 2.2 基于注解添加

基于注解，将MyApplicationListener配置到Spring的IOC容器中。

```java
@Configuration
public class MyApplicationListener implements ApplicationListener<ApplicationEvent> {
    @Override
    public void onApplicationEvent(ApplicationEvent applicationEvent) {
        System.out.println("======>MyApplicationListener: "+applicationEvent);
    }
}
```

### 2.3 在SpringBoot的配置文件中配置

另外还一种方式，可以在SpringBoot的配置文件application.properties中配置

```properties
context.listener.classes=com.roy.applicationListener.MyApplicationListener
```

这几种方式都可以配置事件监听器。另外，其实在上一章节`ApplicationContextInitializer`中也能看到，在SpringBoot内部也在应用初始化中扩展出了很多通过application添加事件监听器的扩展。



## 3.内置事件

| 序号 | Spring 内置事件 & 描述                                       |
| :--- | :----------------------------------------------------------- |
| 1    | **ContextRefreshedEvent**[ApplicationContext](https://so.csdn.net/so/search?q=ApplicationContext&spm=1001.2101.3001.7020) 被初始化或刷新时，该事件被发布。这也可以在 ConfigurableApplicationContext接口中使用 refresh() 方法来发生。此处的初始化是指：所有的Bean被成功装载，后处理Bean被检测并激活，所有Singleton Bean 被预实例化，ApplicationContext容器已就绪可用 |
| 2    | **ContextStartedEvent**当使用 ConfigurableApplicationContext （ApplicationContext子接口）接口中的 start() 方法启动 ApplicationContext 时，该事件被发布。你可以调查你的数据库，或者你可以在接受到这个事件后重启任何停止的应用程序。 |
| 3    | **ContextStoppedEvent**当使用 ConfigurableApplicationContext 接口中的 stop() 停止 ApplicationContext 时，发布这个事件。你可以在接受到这个事件后做必要的清理的工作。 |
| 4    | **ContextClosedEvent**当使用 ConfigurableApplicationContext 接口中的 close() 方法关闭 ApplicationContext 时，该事件被发布。一个已关闭的上下文到达生命周期末端；它不能被刷新或重启。 |
| 5    | **RequestHandledEvent**这是一个 web-specific 事件，告诉所有 bean HTTP 请求已经被服务。只能应用于使用DispatcherServlet的Web应用。在使用Spring作为前端的MVC控制器时，当Spring处理用户请求结束后，系统会自动触发该事件。 |

业务方监听事件举例

比如要监听ContextRefreshedEvent的时可以实现ApplicationListener接口，并且传入要监听的事件

```java
@Component
public class TestApplicationListener implements ApplicationListener<ContextRefreshedEvent>{
    @Override
    public void onApplicationEvent(ContextRefreshedEvent contextRefreshedEvent) {
        System.out.println(contextRefreshedEvent);
        System.out.println("TestApplicationListener............................");
    }
}
```

## 4.自定义事件

可以自定义事件，然后做完业务处理后手动发出。同上集成某个监听接口，接收到事件后进行业务处理

### 事件定义:

```java
public class EmailEvent extends ApplicationEvent{
　　 private String address;
　　 private String text;
　　 public EmailEvent(Object source, String address, String text){
　　 super(source);
　　　　　 this.address = address;
　　　　　 this.text = text;
　　 }
　　 public EmailEvent(Object source) {
　　　　　super(source);
　　 }
　　 //......address和text的setter、getter
}
```

### 监听定义

```java
public class EmailNotifier implements ApplicationListener{
　　 public void onApplicationEvent(ApplicationEvent event) {
　　　　　if (event instanceof EmailEvent) {
　　　　　　　 EmailEvent emailEvent = (EmailEvent)event;
　　　　　　　 System.out.println("邮件地址：" + emailEvent.getAddress());
　　　　　　　 System.our.println("邮件内容：" + emailEvent.getText());
　　　　　} else {
　　　　　　　 System.our.println("容器本身事件：" + event);
　　　　　}
　　 }
}
```

### 业务触发

```java
public class SpringTest {
　　 public static void main(String args[]){
　　　　　ApplicationContext context = new ClassPathXmlApplicationContext("bean.xml");
　　　　　//创建一个ApplicationEvent对象
　　　　　EmailEvent event = new EmailEvent("hello","abc@163.com","This is a test");
　　　　　//主动触发该事件
　　　　　context.publishEvent(event);
　　 }
}
```

> **\*不管是内置监听还是外部自定义监听一定要把实现ApplicationListener的类定义成一个bean才行，可以是通过注解@Component等也可以通过xml的方式去执行。\**

# 二、核心机制解读

事件机制的使用方式很多，不同的配置方式也有不同的加载流程。我们这里还是只解读SpringBoot中如何通过`spring.factories`文件来加载事件监听器的。

SpringBoot中对于监听器的处理，也跟`ApplicationContextInitializer`的处理流程是差不多的。首先在SpringApplication的构造方法中加载所有的监听器：

```java
public SpringApplication(ResourceLoader resourceLoader, Class<?>... primarySources) {
       ...
        this.setInitializers(this.getSpringFactoriesInstances(ApplicationContextInitializer.class));
    	//加载spring.facotries中注册的所有事件监听器
        this.setListeners(this.getSpringFactoriesInstances(ApplicationListener.class));
        ...
    }
```

然后在SpringApplication的run方法中启动所有监听器：

```java
public ConfigurableApplicationContext run(String... args) {
        ...
        SpringApplicationRunListeners listeners = this.getRunListeners(args);//<====注册SpringApplicationRunListener
        listeners.starting(bootstrapContext, this.mainApplicationClass); //<===会发布ApplicationStartingEvent事件

        try {
            ApplicationArguments applicationArguments = new DefaultApplicationArguments(args);
            ConfigurableEnvironment environment = this.prepareEnvironment(listeners, bootstrapContext, applicationArguments);//<===在这个过程中会发布ApplicationEnvironmentPreparedEvent
            this.configureIgnoreBeanInfo(environment);
            Banner printedBanner = this.printBanner(environment);
            context = this.createApplicationContext();
            context.setApplicationStartup(this.applicationStartup);
            this.prepareContext(bootstrapContext, context, environment, listeners, applicationArguments, printedBanner);// <==发布ApplicationContextInitializedEvent ApplicationPreparedEvent事件
            this.refreshContext(context);// <===发布ContextRefreshedEvent事件
            this.afterRefresh(context, applicationArguments);
            stopWatch.stop();
            if (this.logStartupInfo) {
                (new StartupInfoLogger(this.mainApplicationClass)).logStarted(this.getApplicationLog(), stopWatch);
            }

            listeners.started(context);//<===发布ApplicationStartedEvent AvailabilityChangeEvent事件
            this.callRunners(context, applicationArguments);
        } catch (Throwable var10) {
            this.handleRunFailure(context, var10, listeners);
            throw new IllegalStateException(var10);
        }

        try {
            listeners.running(context);//发布ApplicationReadyEvent 和 AvailabilityChangeEvent事件
            return context;
        } catch (Throwable var9) {
            this.handleRunFailure(context, var9, (SpringApplicationRunListeners)null);
            throw new IllegalStateException(var9);
        }
    }
```

首先在注册SpringApplicationRunListener时，就会解析spring.factories，读取其中的org.springframework.boot.SpringApplicationRunListener配置。

```properties
# Run Listeners
org.springframework.boot.SpringApplicationRunListener=\
org.springframework.boot.context.event.EventPublishingRunListener
```

然后从发布事件的地方往下调试，可以看到SpringBoot事件发布的核心对象EventPublishingRunListener。通过其中的initialMulticaster组件来发布不同的事件。 而他实现事件监听的方式就是在发布事件时，实时调用一下已经注册的所有对应事件的监听器。

```java
public class EventPublishingRunListener implements SpringApplicationRunListener, Ordered {

	private final SpringApplication application;

	private final String[] args;

	private final SimpleApplicationEventMulticaster initialMulticaster;

	public EventPublishingRunListener(SpringApplication application, String[] args) {
		this.application = application;
		this.args = args;
		this.initialMulticaster = new SimpleApplicationEventMulticaster();
		for (ApplicationListener<?> listener : application.getListeners()) {
			this.initialMulticaster.addApplicationListener(listener);
		}
	}

	@Override
	public int getOrder() {
		return 0;
	}

	@Override
	public void starting(ConfigurableBootstrapContext bootstrapContext) {
		//发布事件
        this.initialMulticaster
				.multicastEvent(new ApplicationStartingEvent(bootstrapContext, this.application, this.args)); 
	}
    .....
}
```

调用Spring中的SimpleApplicationEventMulticaster组件发布事件。

```java
public class SimpleApplicationEventMulticaster extends AbstractApplicationEventMulticaster {
    ...

    public void multicastEvent(ApplicationEvent event, @Nullable ResolvableType eventType) {
        ResolvableType type = eventType != null ? eventType : this.resolveDefaultEventType(event);
        Executor executor = this.getTaskExecutor();
        //根据event查找注册的监听器
        Iterator var5 = this.getApplicationListeners(event, type).iterator();

        while(var5.hasNext()) {
            ApplicationListener<?> listener = (ApplicationListener)var5.next();
            if (executor != null) {
                executor.execute(() -> {
                    //调用Listener的onApplicationEvent方法。
                    this.invokeListener(listener, event);
                });
            } else {
                this.invokeListener(listener, event);
            }
        }

    }

```

这其中initialMulticaster对象已经是Spring-Context包中的内容。所以这里就完成了SpringBoot中事务监听机制的梳理。

这个事件机制也是SpringBoot使用过程中非常好的功能扩展点，因为应用启动过程中，这些事件都已经默认发布了，可以叠加自己想要的应用初始化工作。这些关键事件的发布顺序也是非常重要的。例如，如果你的扩展功能需要用到Spring的IOC容器，那就只能去监听ContextRefreshedEvent之后的几个内部事件。

# 三、SpringBoot中的核心实现

接下来梳理SpringBoot当中通过spring.factories默认注册的事务监听器

```properties
#spirng-boot.jar
org.springframework.context.ApplicationListener=\
org.springframework.boot.ClearCachesApplicationListener,\
org.springframework.boot.builder.ParentContextCloserApplicationListener,\
org.springframework.boot.context.FileEncodingApplicationListener,\
org.springframework.boot.context.config.AnsiOutputApplicationListener,\
org.springframework.boot.context.config.DelegatingApplicationListener,\
org.springframework.boot.context.logging.LoggingApplicationListener,\
org.springframework.boot.env.EnvironmentPostProcessorApplicationListener,\
org.springframework.boot.liquibase.LiquibaseServiceLocatorApplicationListener
#spring-boot-autoconfigure
org.springframework.context.ApplicationListener=\
org.springframework.boot.autoconfigure.BackgroundPreinitializer
```

那接下来同样是梳理几个有代表性的事务监听器逻辑。

例如BackgroundPreinitializer，他会将几个比较耗时的初始化工作提前到应用启动过程中加载，并且单独启动一个线程来加快加载速度。

```java
@Order(LoggingApplicationListener.DEFAULT_ORDER + 1)
public class BackgroundPreinitializer implements ApplicationListener<SpringApplicationEvent> {

    //这段说明很重要
	/**
	 * System property that instructs Spring Boot how to run pre initialization. When the
	 * property is set to {@code true}, no pre-initialization happens and each item is
	 * initialized in the foreground as it needs to. When the property is {@code false}
	 * (default), pre initialization runs in a separate thread in the background.
	 * @since 2.1.0
	 */
	public static final String IGNORE_BACKGROUNDPREINITIALIZER_PROPERTY_NAME = "spring.backgroundpreinitializer.ignore";

	private static final AtomicBoolean preinitializationStarted = new AtomicBoolean();

	private static final CountDownLatch preinitializationComplete = new CountDownLatch(1);

	private static final boolean ENABLED;

	static {
		ENABLED = !Boolean.getBoolean(IGNORE_BACKGROUNDPREINITIALIZER_PROPERTY_NAME) && !NativeDetector.inNativeImage()
				&& Runtime.getRuntime().availableProcessors() > 1;
	}
	//监听SpringApplicationEvent事件。这个事件是之前调试过的几个重要事件的父接口
	@Override
	public void onApplicationEvent(SpringApplicationEvent event) {
		if (!ENABLED) {
			return;
		}
		if (event instanceof ApplicationEnvironmentPreparedEvent
				&& preinitializationStarted.compareAndSet(false, true)) {
			performPreinitialization();
		}
		if ((event instanceof ApplicationReadyEvent || event instanceof ApplicationFailedEvent)
				&& preinitializationStarted.get()) {
			try {
				preinitializationComplete.await();
			}
			catch (InterruptedException ex) {
				Thread.currentThread().interrupt();
			}
		}
	}
	...
}
```

接下来其他几个事件监听器的功能也简单总结下。有兴趣的建议自行调试下代码，这样才能形成自己的理解。

- org.sf.boot.ClearCachesApplicationListener：应用上下文加载完成后对缓存做清除工作，响应事件ContextRefreshedEvent
- org.sf.boot.builder.ParentContextCloserApplicationListener：监听双亲应用上下文的关闭事件并往自己的孩子应用上下文中传播，相关事件ParentContextAvailableEvent/ContextClosedEvent
- org.sf.boot.context.FileEncodingApplicationListener：如果系统文件编码和环境变量中指定的不同则终止应用启动。 具体的方法是比较系统属性file.encoding和环境变量spring.mandatory-file-encoding是否相等(大小写不敏感)。
- org.sf.boot.context.config.AnsiOutputApplicationListener：根据spring.output.ansi.enabled参数配置AnsiOutput
- org.sf.boot.context.config.DelegatingApplicationListener：监听到事件后转发给环境变量context.listener.classes指定的那些事件监听器
- org.sf.boot.context.logging.LoggingApplicationListener 配置LoggingSystem。使用logging.config环境变量指定的配置或者缺省配置
- org.springframework.boot.env.EnvironmentPostProcessorApplicationListener： 加载spring.factories文件中配置的EnvironmentPostProcessor。
- org.sf.boot.liquibase.LiquibaseServiceLocatorApplicationListener 使用一个可以和Spring Boot可执行jar包配合工作的版本替换liquibase ServiceLocator

> org.sf.boot.context.config.ConfigFileApplicationListener ： 指定SpringBoot的配置文件地址。 但是在2.4.5版本已经移除。

最后，有没有觉得EnvironmentPostProcessorApplicationListener这个事件监听器挺有意思的？SpringBoot直接将spring.factories中EnvironmentPostProcessor机制的加载工作交给了事件监听器，接下来我们就趁热打铁，来继续看下EnvironmentPostProcessor 的处理机制。

# 四、EnvironmentPostProcessor使用

EnvironmentPostProcessor是在环境信息加载完成后进行一些补充处理。例如下面一个示例可以在SpringBoot读取完application.properties后，补充读取另一个配置文件：

```java
//@Component
public class MyEnvironmentPostProcessor  implements EnvironmentPostProcessor{
 
	@Override
	public void postProcessEnvironment(ConfigurableEnvironment environment, SpringApplication application) {
		try(InputStream input = new FileInputStream("E:\\ds.properties")) {
			Properties properties = new Properties();
			properties.load(input);
			PropertiesPropertySource propertySource = new PropertiesPropertySource("ve", properties);
			environment.getPropertySources().addLast(propertySource);
			System.out.println("====加载外部配置文件完毕====");
		} catch (Exception e) {
			e.printStackTrace();
		}		
	}
复制代码
```

使用的方式同样可以配置到spring.factories文件中，或者通过@Component注解加入到IOC容器中。

```properties
org.springframework.boot.env.EnvironmentPostProcessor=\
com.roy.environmentPostProcessor.MyEnvironmentPostProcessor
```

# 四、EnvironmentPostProcessor 加载机制

EnvironmentPostProcessor 是通过EnvironmentPostProcessorApplicationListener监听Spring事件来对运行环境进行补充的后续处理。先来看下他监听了哪些事件。

```java
public class EnvironmentPostProcessorApplicationListener implements SmartApplicationListener, Ordered {

    //执行优先级非常高
	/**
	 * The default order for the processor.
	 */
	public static final int DEFAULT_ORDER = Ordered.HIGHEST_PRECEDENCE + 10;
	...
	@Override
	public void onApplicationEvent(ApplicationEvent event) {
		if (event instanceof ApplicationEnvironmentPreparedEvent) {
			onApplicationEnvironmentPreparedEvent((ApplicationEnvironmentPreparedEvent) event);
		}
		if (event instanceof ApplicationPreparedEvent) {
			onApplicationPreparedEvent((ApplicationPreparedEvent) event);
		}
		if (event instanceof ApplicationFailedEvent) {
			onApplicationFailedEvent((ApplicationFailedEvent) event);
		}
	}
复制代码
```

从这几个事件当中可以看出，**在他的子处理器中是不能引用IOC容器的**。

然后再往下继续调试他的onApplicationEnvironmentPreparedEvent方法，找下他解析spring.factories，加载EnvironmentPostProcessor 实现的机制。这个机制就不再是使用 SpringFactoriesLoader 了。

```java
//处理EnvironmentPostProcessor的方式。
private void onApplicationEnvironmentPreparedEvent(ApplicationEnvironmentPreparedEvent event) {
		ConfigurableEnvironment environment = event.getEnvironment();
		SpringApplication application = event.getSpringApplication();
    	//重点跟踪getEnvironmentPostProcessors方法
		for (EnvironmentPostProcessor postProcessor : getEnvironmentPostProcessors(event.getBootstrapContext())) {
			postProcessor.postProcessEnvironment(environment, application);
		}
	}
复制代码
```

一路往下调试，会跟踪到ReflectionEnvironmentPostProcessorsFactory的getEnvironmentPostProcessors方法。

```java
public List<EnvironmentPostProcessor> getEnvironmentPostProcessors(DeferredLogFactory logFactory,
			ConfigurableBootstrapContext bootstrapContext) {
		Instantiator<EnvironmentPostProcessor> instantiator = new Instantiator<>(EnvironmentPostProcessor.class,
				(parameters) -> {
					parameters.add(DeferredLogFactory.class, logFactory);
					parameters.add(Log.class, logFactory::getLog);
					parameters.add(ConfigurableBootstrapContext.class, bootstrapContext);
					parameters.add(BootstrapContext.class, bootstrapContext);
					parameters.add(BootstrapRegistry.class, bootstrapContext);
				});
		return instantiator.instantiate(this.classNames);
	}
```

这个parameters就是给这一组实例添加一些默认的参数类。后面各个子类可以创建包含了这些参数的构造方法来进行实例化。 **有没有发现，这就是一个简单的IOC属性注入吗？学IOC有比这个更经典的入手案例吗？**

具体实例化的方式在Instantiator这个类中

```java
private T instantiate(Class<?> type) throws Exception {
		Constructor<?>[] constructors = type.getDeclaredConstructors();
		Arrays.sort(constructors, CONSTRUCTOR_COMPARATOR);
		for (Constructor<?> constructor : constructors) {
            //args就来自于上面指定的几个类
			Object[] args = getArgs(constructor.getParameterTypes());
			if (args != null) {
				ReflectionUtils.makeAccessible(constructor);
				return (T) constructor.newInstance(args);
			}
		}
		throw new IllegalAccessException("Unable to find suitable constructor");
	}
复制代码
```

# 五、SpringBoot中注册的EnvironmentPostProcessor实现

接下来还是梳理下SpringBoot在spring.factories当中注册的EnvironmentPostProcessor实现：

```properties
#spirng-boot.jar当中
# Environment Post Processors
org.springframework.boot.env.EnvironmentPostProcessor=\
org.springframework.boot.cloud.CloudFoundryVcapEnvironmentPostProcessor,\
org.springframework.boot.context.config.ConfigDataEnvironmentPostProcessor,\
org.springframework.boot.env.RandomValuePropertySourceEnvironmentPostProcessor,\
org.springframework.boot.env.SpringApplicationJsonEnvironmentPostProcessor,\
org.springframework.boot.env.SystemEnvironmentPropertySourceEnvironmentPostProcessor,\
org.springframework.boot.reactor.DebugAgentEnvironmentPostProcessor
```

这些实现类具体是干什么的，这个我暂时还看不太懂，应该跟具体的一些细节场景有关了。网上也暂时没有找到相关的资料。就不再去具体分析了。

> 不得不吐槽下，网上的帖子千篇一律只讲了怎么用EnvironmentPostProcesser读取额外的配置文件，但是这些机制还真没有人来分析过。