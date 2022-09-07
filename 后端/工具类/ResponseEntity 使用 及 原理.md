# ResponseEntity 使用 及 原理

## 一.使用

ResponseEntity可以作为controller的返回值，比如对于一个处理下载二进制文件的接口，可以这么定义：

```java
@RequestMapping("/download")
public ResponseEntity<byte[]> download(@RequestParam String fileName) throws IOException {
    byte[] bytes = xxx;
    return new ResponseEntity<>(bytes, headers, HttpStatus.OK);
}
```
也可以使用流式风格的构造器：

```java
@RequestMapping("/download")
public ResponseEntity<byte[]> download(@RequestParam String fileName) throws IOException {
    byte[] bytes = xxx;
    return ResponseEntity.ok()
            .headers(headers)
            .body(bytes);
}        
```


当然，ResponseEntity不仅仅可以用于处理下载，非ModelAndView的其他场景均可以使用。在WEB API项目中，如果没有特定的Java Bean封装的返回类型，可以使用该类型。

 

## 二.原理

那么这个ResponseEntity到底是啥？SpringMVC框架又是如何处理的？



```java
public class HttpEntity<T> {
private final HttpHeaders headers;
 
@Nullable //提示给代码编译这里可能会出现空值 允许出现空值
private final T body;
 
// 省略其他代码
}
```


ResponseEntity**继承了HttpEntity类**，HttpEntity代表一个http请求或者响应实体，其内部有两个成员变量：**header**及**body**，代表http请求或响应的header及body，其中的body是泛化的。



```java
public class ResponseEntity<T> extends HttpEntity<T> {
private final Object status;
 
public ResponseEntity(@Nullable T body, @Nullable MultiValueMap<String, String> headers, HttpStatus status) {
	super(body, headers);
	Assert.notNull(status, "HttpStatus must not be null");
	this.status = status;
}
 
// 省略其他代码
}
```

下面看下ResponseEntity类，扩展了HttpEntity类**，新增了status成员变量**，这样，一个ResponseEntity基本可以代表完整的http的请求或响应了。这其实就是ResponseEntity类的作用。

**使用ResponseEntity作为controller的返回值**，我们可以方便地处理响应的header，状态码以及body。**而通常使用的@ResponseBody注解，只能处理body部分。这也是为什么通常在下载场景中会使用ResponseEntity，因为下载需要设置header里的content-type以及特殊的status（比如206）。**

 

HttpEntityMethodProcessor

该类继承自AbstractMessageConverterMethodProcessor类，AbstractMessageConverterMethodProcessor类我们之前在介绍@Responsebody注解原理时提到过，其中RequestResponseBodyMethodProcessor是用来处理@Responsebody注解的。这里的HttpEntityMethodProcessor是AbstractMessageConverterMethodProcessor的另一个子类。该抽象类常见的子类其实就这俩了。在HttpEntityMethodProcessor中：


```java
@Override
public boolean supportsReturnType(MethodParameter returnType) {
	return (HttpEntity.class.isAssignableFrom(returnType.getParameterType()) &&
			!RequestEntity.class.isAssignableFrom(returnType.getParameterType()));
}
```
可以看到，该processor专门处理返回值类型是ResponseEntity类型的controller返回值。处理过程如下：




```java
@Override
public void handleReturnValue(@Nullable Object returnValue, MethodParameter returnType,
		ModelAndViewContainer mavContainer, NativeWebRequest webRequest) throws Exception {
mavContainer.setRequestHandled(true);
if (returnValue == null) {
	return;
}
 
ServletServerHttpRequest inputMessage = createInputMessage(webRequest);
ServletServerHttpResponse outputMessage = createOutputMessage(webRequest);
 
Assert.isInstanceOf(HttpEntity.class, returnValue);
HttpEntity<?> responseEntity = (HttpEntity<?>) returnValue;
 
HttpHeaders outputHeaders = outputMessage.getHeaders();
HttpHeaders entityHeaders = responseEntity.getHeaders();
if (!entityHeaders.isEmpty()) {
	entityHeaders.forEach((key, value) -> {
		if (HttpHeaders.VARY.equals(key) && outputHeaders.containsKey(HttpHeaders.VARY)) {
			List<String> values = getVaryRequestHeadersToAdd(outputHeaders, entityHeaders);
			if (!values.isEmpty()) {
				outputHeaders.setVary(values);
			}
		}
		else {
			outputHeaders.put(key, value);
		}
	});
}
 
if (responseEntity instanceof ResponseEntity) {
	int returnStatus = ((ResponseEntity<?>) responseEntity).getStatusCodeValue();
	outputMessage.getServletResponse().setStatus(returnStatus);
	if (returnStatus == 200) {
		if (SAFE_METHODS.contains(inputMessage.getMethod())
				&& isResourceNotModified(inputMessage, outputMessage)) {
			// Ensure headers are flushed, no body should be written.
			outputMessage.flush();
			ShallowEtagHeaderFilter.disableContentCaching(inputMessage.getServletRequest());
			// Skip call to converters, as they may update the body.
			return;
		}
	}
	else if (returnStatus / 100 == 3) {
		String location = outputHeaders.getFirst("location");
		if (location != null) {
			saveFlashAttributes(mavContainer, webRequest, location);
		}
	}
}
 
// Try even with null body. ResponseBodyAdvice could get involved.
writeWithMessageConverters(responseEntity.getBody(), returnType, inputMessage, outputMessage);
 
// Ensure headers are flushed even if no body was written.
outputMessage.flush();
}
```

分开来看下：

```java
mavContainer.setRequestHandled(true);
if (returnValue == null) {
	return;
}
```

这段代码是为了标识该请求已经被处理过，不需要走view渲染逻辑。

```java
HttpHeaders outputHeaders = outputMessage.getHeaders();
HttpHeaders entityHeaders = responseEntity.getHeaders();
if (!entityHeaders.isEmpty()) {
	entityHeaders.forEach((key, value) -> {
		if (HttpHeaders.VARY.equals(key) && outputHeaders.containsKey(HttpHeaders.VARY)) {
			List<String> values = getVaryRequestHeadersToAdd(outputHeaders, entityHeaders);
			if (!values.isEmpty()) {
				outputHeaders.setVary(values);
			}
		}
		else {
			outputHeaders.put(key, value);
		}
	});
}
```

这段代码处理ResponseEntity里的header部分，写入httpResponse对象的header里。

接下来的代码负责将entity中的status也写入httpResponse对象中。

最后body部分是由这段代码处理的：

```java
// Try even with null body. ResponseBodyAdvice could get involved.
writeWithMessageConverters(responseEntity.getBody(), returnType, inputMessage, outputMessage);
```

调用了父类的模板方法里，该模板方法负责将内部的messageConverter实例一次调用，根据mediaType找出合适的：

```java
for (HttpMessageConverter<?> converter : this.messageConverters) {
	GenericHttpMessageConverter genericConverter = (converter instanceof GenericHttpMessageConverter ?
			(GenericHttpMessageConverter<?>) converter : null);
	if (genericConverter != null ?
			((GenericHttpMessageConverter) converter).canWrite(targetType, valueType, selectedMediaType) :
			converter.canWrite(valueType, selectedMediaType)) {
		body = getAdvice().beforeBodyWrite(body, returnType, selectedMediaType,
				(Class<? extends HttpMessageConverter<?>>) converter.getClass(),
				inputMessage, outputMessage);
		if (body != null) {
			Object theBody = body;
			LogFormatUtils.traceDebug(logger, traceOn ->
					"Writing [" + LogFormatUtils.formatValue(theBody, !traceOn) + "]");
			addContentDispositionHeader(inputMessage, outputMessage);
			if (genericConverter != null) {
				genericConverter.write(body, targetType, selectedMediaType, outputMessage);
			}
			else {
				((HttpMessageConverter) converter).write(body, selectedMediaType, outputMessage);
			}
		}
		else {
			if (logger.isDebugEnabled()) {
				logger.debug("Nothing to write: null body");
			}
		}
		return;
	}
}
```

这部分内容就不深入了，在另一篇文章中会介绍。

小结：

**ResponseEntity对应一个http请求或者响应，可以用在controller的返回值里，方便处理header及status。**

ResponseEntity类型的返回值由一个特殊的HttpEntityMethodProcessor类型的returnTypeHandler来处理，主要是将ResponseEntity里设置的header和status写入到httpResponse中，body部分调用的是父类的模板方法。



