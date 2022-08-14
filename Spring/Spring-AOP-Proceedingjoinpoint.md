# Spring-AOP-Proceedingjoinpoint

## 简述

​    Proceedingjoinpoint 继承了JoinPoint，在JoinPoint的基础上暴露出 proceed()， 这个方法是AOP代理链执行的方法。

​    JoinPoint仅能获取相关参数，无法执行连接点。暴露出proceed()这个方法，就能支持 aop:around 这种切面（而其他的几种切面只需要用到JoinPoint，这跟切面类型有关），就能控制走代理链还是走自己拦截的其他逻辑。

​    建议看一下 JdkDynamicAopProxy的invoke方法，了解一下代理链的执行原理。



## 类源码

### **JoinPoint**

```java
public interface JoinPoint {  
   String toString();         //连接点所在位置的相关信息  
   String toShortString();    //连接点所在位置的简短相关信息  
   String toLongString();     //连接点所在位置的全部相关信息  
   Object getThis();          //返回AOP代理对象，也就com.sun.proxy.$Proxy18
   Object getTarget();        //返回目标对象，一般我们都需要它或者（也就是定义方法的接口或类，为什么会是接口呢？
                              //这主要是在目标对象本身是动态代理的情况下，例如Mapper。所以返回的是定义方法的对象如
                              //aoptest.daoimpl.GoodDaoImpl或com.b.base.BaseMapper<T, E, PK>）
   Object[] getArgs();        //返回被通知方法参数列表  
   Signature getSignature();  //返回当前连接点签名。其getName()方法返回方法的                               //FQN，如void aoptest.dao.GoodDao.delete()
                              //或com.b.base.BaseMapper.insert(T)(需要注意的是，很多时候我们定义了子类继承父类的时候，
                              //我们希望拿到基于子类的FQN，无法直接拿到，要依赖于
                              //AopUtils.getTargetClass(point.getTarget())获取原始代理对象，下面会详细讲解)
   SourceLocation getSourceLocation();//返回连接点方法所在类文件中的位置  
   String getKind();           //连接点类型  
   StaticPart getStaticPart(); //返回连接点静态部分  
  }  

```



#### **JoinPoint.StaticPart**

​		提供访问连接点的静态部分，如被通知方法签名、连接点类型等。

```java
public interface StaticPart {  
   Signature getSignature();    //返回当前连接点签名  
   String getKind();            //连接点类型  
   int getId();                 //唯一标识  
   String toString();           //连接点所在位置的相关信息  
   String toShortString();      //连接点所在位置的简短相关信息  
   String toLongString();       //连接点所在位置的全部相关信息  
}
```



#### Signature

```java
public interface Signature {
  String toString();
 //获得连接点较短的信息
 String toShortString();
 //获得连接点较长的信息
 String toLongString();
 //得到连接方法名 常用
 String getName();
 //得到方法的声明类型 Modifier.toString（jp.getSignature().getModifiers()）
 int getModifiers(); 
 //得到连接点的类型 
 Class getDeclaringType();
 //得到连接点方法的类名 常用
 String getDeclaringTypeName();
}
```



### **ProceedingJoinPoint**

```java
public interface ProceedingJoinPoint extends JoinPoint {  
    	void set$AroundClosure(AroundClosure var1);
		// 执行方法 ，返回执行的结果 因为 环绕通知 不像 @Before @After 能够区分出，切点方法执行的前后要执行的前后顺序，因此使用 proceed（）来 区分 方法执行的前后
       public Object proceed() throws Throwable;  
    	//使用自己定义的 参数去Object[] var1 去执行方法
       public Object proceed(Object[] args) throws Throwable;  
 }
```



## 用法

## **获得切点对应的方法（Method）**

​    本处Method指的是java.lang.reflect.Method

​    若切入点表达式是方法，则获得的是切入点方法的信息。若切入点表达式是注解，则获得的是使用了切入点注解的方法的信息。

```java
@Around("pointCut()")
public void around3(ProceedingJoinPoint joinPoint) throws Throwable {
    MethodSignature methodSignature = (MethodSignature) joinPoint.getSignature();
    Method method = methodSignature.getMethod();

}
```

## **获得参数（Object[]）**

```java
@Around("pointCut()")
public void around3(ProceedingJoinPoint joinPoint) throws Throwable {
    Object[] objects = joinPoint.getArgs();
    for (Object o : objects) {
        System.out.println(o);
        if (o instanceof User) {
            System.out.println("id是："     + ((User)(o)).getId() +
                                "; 名字是：" + ((User)(o)).getUserName() +
                                "; 备注是：" + ((User)(o)).getNote());
        }
    }
}
```

​    备注：因为SpringAOP仅支持在方法上使用自定义注解。所以，不能通过下边这种方法获得注解在参数上的参数值。

```java
@Around("pointCut()")
public void around3(ProceedingJoinPoint joinPoint) throws Throwable {
    MethodSignature methodSignature = (MethodSignature) joinPoint.getSignature();
    Method method = methodSignature.getMethod();
    Annotation[][] parameterAnnotations= method.getParameterAnnotations();
    for (Annotation[] annotations : parameterAnnotations) {
        for (Annotation annotation : annotations) {
            System.out.println(annotation.annotationType().getName());
        }
    }
}

```

## **获得目标类**

​    若切入点表达式是方法，则获得的是切入点方法所在类。若切入点表达式是注解，则获得的是使用此注解的方法所在的类。

```java
@Around("pointCut()")
public void around3(ProceedingJoinPoint joinPoint) throws Throwable {
    System.out.println("joinPoint.getTarget().toString() : " + 
        joinPoint.getTarget().toString());
}
```

# **Method的作用**

| **方法名**                              | **作用**                                                     |
| --------------------------------------- | ------------------------------------------------------------ |
| getAnnotation(Class<T> annotationClass) | 获得使用了切入点对应的注解的注解类实例。                     |
| getName()                               | 方法的名字                                                   |
| Class<?> getReturnType()                | 获得切入点对应的方法的返回值类。                             |
| getParameterAnnotations()               | 获得使用了切入点对应的注解的参数数组。由于SpringAOP不支持参数上用自定义注解，本法基本用不到。 |





