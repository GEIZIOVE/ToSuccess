# Spring MVC-RequestContextHolder

## 1.概述

![img](E:\Development\Typora\images\843313-20190820220720353-797417310.png)

​		在**实际**开发中：想在`Service`层或者**某个工具类**层里获取`HttpServletRequest`对象，甚至response的都有。 其中一种方式是，把request当作入参，一层一层的传递下去。不过这种有点费劲，且做起来很不优雅。(我之前的做法…)

这里介绍另外一种方案：`RequestContextHolder`，任意地方使用如下代码：

```java
HttpServletRequest request = ((ServletRequestAttributes)RequestContextHolder.getRequestAttributes()).getRequest();
```



## 2.源码详解

**`RequestContextHolder`顾名思义,持有上下文的Request容器.使用是很简单的**，它所有方法都是`static`的

该类主要维护了两个全局容器（基于`ThreadLocal`，使用了两个[ThreadLocal](https://so.csdn.net/so/search?q=ThreadLocal&spm=1001.2101.3001.7020)来存储当前线程下的request。）

```java
	// jsf是JSR-127标准的一种用户界面框架  过时的技术，所以此处不再做讨论
	private static final boolean jsfPresent =
			ClassUtils.isPresent("javax.faces.context.FacesContext", 			RequestContextHolder.class.getClassLoader());
	
	//现成和request绑定的容器
	private static final ThreadLocal<RequestAttributes> requestAttributesHolder =
			new NamedThreadLocal<>("Request attributes");
	// 和上面比较，它是被子线程继承的request   Inheritable:可继承的
	private static final ThreadLocal<RequestAttributes> inheritableRequestAttributesHolder =new NamedInheritableThreadLocal<>("Request context");
```

现在主要是它的一些get/set方法之类的：

```java
	public static void resetRequestAttributes() {
		requestAttributesHolder.remove(); //移除线程变量
		inheritableRequestAttributesHolder.remove();
	}

	// 把传入的RequestAttributes和当前线程绑定。 注意这里传入false：表示不能被继承
	public static void setRequestAttributes(@Nullable RequestAttributes attributes) {
		setRequestAttributes(attributes, false); //可具体去看看源码
	}
	
	//兼容继承和非继承  只要得到了就成
	public static RequestAttributes getRequestAttributes() {
		RequestAttributes attributes = requestAttributesHolder.get();
		if (attributes == null) {
			attributes = inheritableRequestAttributesHolder.get();
		}
		return attributes;
	}
	
	//在没有jsf的时候，效果完全同getRequestAttributes()  因为jsf几乎废弃了，所以效果可以说一致
	public static RequestAttributes currentRequestAttributes() throws IllegalStateException{
        RequestAttributes attributes = getRequestAttributes();
		if (attributes == null) {
			if (jsfPresent) {
				attributes = FacesRequestAttributesFactory.getFacesRequestAttributes();
			}
			if (attributes == null) {
                throw new IllegalStateException("No thread-bound request found: " ...);
                			}
		}
		return attributes;
        }                                           
    }
```

​		对于方法 `getRequestAttributes()`，实际上就是在类中直接调用了`ThreadLocal`（requestAttributesHolder）的`get()`方法，该方法会判断，如果当前线程的`ThreadLocalMap`存在，就直接使用这个`ThreadLocalMap`，它里面存放着当前线程的信息。如果当前线程的`ThreadLocalMap`不存在就创建一个新的`ThreadLocalMap`，并保存。

![image-20220817155553852](E:\Development\Typora\images\image-20220817155553852.png)



算了，我还是贴一下`setRequestAttributes()`源码

```java
	/**
	 * 将给定的 RequestAttributes 绑定到当前线程
	 * @param attributes the RequestAttributes to expose,
	 * or {@code null} to reset the thread-bound context
	 * @param inheritable whether to expose the RequestAttributes as inheritable
	 * for child threads (using an {@link InheritableThreadLocal})
	 */ 
public static void setRequestAttributes(@Nullable RequestAttributes attributes, boolean inheritable) {
		if (attributes == null) {
			resetRequestAttributes();
		}
		else {
			if (inheritable) {
				inheritableRequestAttributesHolder.set(attributes);
				requestAttributesHolder.remove();
			}
			else {
				requestAttributesHolder.set(attributes);
				inheritableRequestAttributesHolder.remove();
			}
		}
	}
```

## 3.具体使用

​		在代码中，我们并不会使用通过`RequestContextHolder`中获得的`RequestAttributes`，可以看看下面的`RequestAttributes`类，里面的方法太少了，我们没有使用框架之间都会使用servlet包下的`HttpServletRequest`。

​		因此我们要将它转换成`ServletRequestAttributes`。它是Spring提供操作`HttpServletRequest`的类。

转换：

```java
ServletRequestAttributes attributes = (ServletRequestAttributes) RequestContextHolder.getRequestAttributes();
//或者 
//两个方法在没有使用JSF的项目中是没有区别的  
RequestAttributes requestAttributes =(ServletRequestAttributes)  RequestContextHolder.currentRequestAttributes(); 
```

```java 
//从session里面获取对应的值
String str = (String) requestAttributes.getAttribute("name",RequestAttributes.SCOPE_SESSION);
 
HttpServletRequest request = ((ServletRequestAttributes)requestAttributes).getRequest();
HttpServletResponse response = ((ServletRequestAttributes)requestAttributes).getResponse();
```

`RequestAttributes` 注意他是一个接口哦！ 所以我们一般使用他的Spring实现类`ServletRequestAttributes`

### RequestAttributes

```JAVA
public interface RequestAttributes {
    //指示请求范围的常量
    int SCOPE_REQUEST = 0;
    //指示会话范围的常量
	//如果有这样的区别，这最好是指本地隔离的会话。否则，它只是指普通会话
    int SCOPE_SESSION = 1;
    //请求对象的标准引用名称：“request”。
    String REFERENCE_REQUEST = "request";
    //请求对象的标准引用名称：“request”。
    String REFERENCE_SESSION = "session";

    /**
    *	回给定名称的作用域属性的值（如果有）。
    *	参形：
	*	name – 属性的名称 scope - 范围标识符
	*	返回值：
	*	当前属性值，如果未找到，则为null
	**/
    @Nullable
    Object getAttribute(String name, int scope);

    /**
    *	设置给定名称的作用域属性的值，替换现有值（如果有）。
    *	参形：
	*	name – 属性的名称 
	*	value – 属性的值 
	*	scope - 范围标识符
	**/
    void setAttribute(String name, Object value, int scope);

    /**
    * 删除给定名称的作用域属性（如果存在）。
    **/
    void removeAttribute(String name, int scope);

    /**
    *	检索范围内所有属性的名称。
	*	参形：
	*	scope - 范围标识符
	*	返回值：
	*	属性名称为字符串数组
	**/
    String[] getAttributeNames(int scope);

    void registerDestructionCallback(String var1, Runnable var2, int var3);

    @Nullable
    Object resolveReference(String var1);

    //返回当前基础会话的 id。
    String getSessionId();
	//为底层会话公开最佳可用互斥锁：即，为底层会话同步的对象。
    Object getSessionMutex();
}
```

### ServletRequestAttributes

我们来看一下`ServletRequestAttributes`，在`ServletRequestAttributes`类里面有一个HttpServletRequest request变量，看到这个变量我们应该熟悉怎么去操作它了。

```java
public class ServletRequestAttributes extends AbstractRequestAttributes {
    public static final String DESTRUCTION_CALLBACK_NAME_PREFIX = ServletRequestAttributes.class.getName() + ".DESTRUCTION_CALLBACK.";
    
    protected static final Set<Class<?>> immutableValueTypes = new HashSet(16);
    
    private final HttpServletRequest request;
    @Nullable
    private HttpServletResponse response;
    @Nullable
    private volatile HttpSession session;
    private final Map<String, Object> sessionAttributesToUpdate;
    
    public final HttpServletRequest getRequest() {return this.request;}
    @Nullable
	public final HttpServletResponse getResponse() {return this.response;}
    @Nullable
	protected final HttpSession getSession(boolean allowCreate) {}
    //然后其他操作获取设置Attribute的方法....
    //......
```



对于`ServletRequestAttributes`类还有3个子类

![image-20220818002856501](E:\Development\Typora\images\image-20220818002856501.png)



`DispatcherServletWebRequest`继续继承的子类，没什么好说的，唯一就是复写了此方法：

```javascript
	@Override
	public Locale getLocale() {
		return RequestContextUtils.getLocale(getRequest());
	}
```

`ServletWebRequest`是`ServletRequestAttributes`的子类，还实现了接口`NativeWebRequest（提供一些获取Native Request的方法，其实没太大作用）`：它代理得就更加的全一些，比如：

```java
	@Nullable
	public HttpMethod getHttpMethod() {
		return HttpMethod.resolve(getRequest().getMethod());
	}
	
	@Override
	@Nullable
	public String getHeader(String headerName) {
		return getRequest().getHeader(headerName);
	}
	getHeaderValues/getParameter/... 等等一些列更多的代理方法
```

`StandardServletAsyncWebRequest`这个和异步拦截器相关，属于异步上下文范畴，此处不做讨论。

## 4.解决疑问

### **4.1 request和response怎么和当前请求挂钩?**

首先分析RequestContextHolder这个类,里面有两个ThreadLocal保存当前线程下的request,关于ThreadLocal可以参考我的另一篇博文[[Java学习记录--ThreadLocal使用案例](https://link.zhihu.com/?target=http%3A//www.jianshu.com/p/5675690b351e)]

```java
//得到存储进去的request
private static final ThreadLocal<RequestAttributes> requestAttributesHolder =
new NamedThreadLocal<RequestAttributes>("Request attributes");
//可被子线程继承的request
private static final ThreadLocal<RequestAttributes> inheritableRequestAttributesHolder =
new NamedInheritableThreadLocal<RequestAttributes>("Request context");
```



再看`getRequestAttributes()`方法,相当于直接获取ThreadLocal里面的值,这样就保证了每一次获取到的Request是该请求的request.

```java
    public static RequestAttributes getRequestAttributes() {
        RequestAttributes attributes = requestAttributesHolder.get();
        if (attributes == null) {
            attributes = inheritableRequestAttributesHolder.get();
        }
        return attributes;
    }
```

### **4.2request和response等是什么时候设置进去的?**

找这个的话需要对springMVC结构的`DispatcherServlet`的结构有一定了解才能准确的定位该去哪里找相关代码.

在IDEA中会显示如下的继承关系.

![img](E:\Development\Typora\images\v2-e4bbd502427ab89db852b438644793f9_b.jpg)

左边1这里是Servlet的接口和实现类.

右边2这里是使得SpringMVC具有[spring](http://lib.csdn.net/base/javaee)的一些环境变量和Spring容器.类似的XXXAware接口就是对该类提供Spring感知,简单来说就是如果想使用Spring的XXXX就要实现XXXAware,spring会把需要的东西传送过来.

那么剩下要分析的的就是三个类,简单看下源码

1. HttpServletBean 进行初始化工作

2. FrameworkServlet 初始化 WebApplicationContext,并提供service方法预处理请

3. DispatcherServlet 具体分发处理.

那么就可以在**FrameworkServlet**查看到该类重写了service(),doGet(),doPost()...等方法,这些实现里面都有一个预处理方法`**processRequest(request, response)**;`,所以定位到了我们要找的位置

查看`**processRequest(request, response);**`的实现,具体可以分为三步:

1. 获取上一个请求的参数
2. 重新建立新的参数
3. 设置到XXContextHolder
4. 父类的service()处理请求
5. 恢复request
6. 发布事件



```java
protected final void processRequest(HttpServletRequest request, HttpServletResponse response)
throws ServletException, IOException {
long startTime = System.currentTimeMillis();
Throwable failureCause = null;
//获取上一个请求保存的LocaleContext
    LocaleContext previousLocaleContext = LocaleContextHolder.getLocaleContext();
//建立新的LocaleContext
    LocaleContext localeContext = buildLocaleContext(request);
//获取上一个请求保存的RequestAttributes
    RequestAttributes previousAttributes = RequestContextHolder.getRequestAttributes();
//建立新的RequestAttributes
    ServletRequestAttributes requestAttributes = buildRequestAttributes(request, 
response, previousAttributes);
    WebAsyncManager asyncManager = WebAsyncUtils.getAsyncManager(request);
asyncManager.registerCallableInterceptor(FrameworkServlet.class.getName(), 
new RequestBindingInterceptor());
//具体设置的方法
    initContextHolders(request, localeContext, requestAttributes);
try {
        doService(request, response);
    }
catch (ServletException ex) {
failureCause = ex;
throw ex;
    }
catch (IOException ex) {
   failureCause = ex;
   throw ex;
    }
catch (Throwable ex) {
   failureCause = ex;
   throw new NestedServletException("Request processing failed", ex);
    }
finally {
//恢复
        resetContextHolders(request, previousLocaleContext, previousAttributes);
if (requestAttributes != null) {
requestAttributes.requestCompleted();
        }
if (logger.isDebugEnabled()) {
if (failureCause != null) {
this.logger.debug("Could not complete request", failureCause);
            }
else {
if (asyncManager.isConcurrentHandlingStarted()) {
                    logger.debug("Leaving response open for concurrent processing");
                }
else {
this.logger.debug("Successfully completed request");
                }
            }
        }
//发布事件
        publishRequestHandledEvent(request, response, startTime, failureCause);
    }
}
```



再看initContextHolders(request, localeContext, requestAttributes)方法,把新的RequestAttributes设置进LocalThread,实际上保存的类型为**ServletRequestAttributes**,这也是为什么在使用的时候可以把**RequestAttributes**强转为**ServletRequestAttributes**.



```java
private void initContextHolders(HttpServletRequest request, 
                                LocaleContext localeContext, 
                                RequestAttributes requestAttributes) {
if (localeContext != null) {
        LocaleContextHolder.setLocaleContext(localeContext, 
this.threadContextInheritable);
    }
if (requestAttributes != null) {
        RequestContextHolder.setRequestAttributes(requestAttributes, 
this.threadContextInheritable);
    }
if (logger.isTraceEnabled()) {
        logger.trace("Bound request context to thread: " + request);
    }
}
```



因此RequestContextHolder里面最终保存的为**ServletRequestAttributes**,这个类相比`**RequestAttributes**`方法是多了很多.

![img](E:\Development\Typora\images\v2-776dae1be28205c2e2e53bb0019941d8_b.jpg)

