# SpringBoot-ApplicationContextAware

> 在某些特殊的情况下，Bean需要实现某个功能，但该功能必须借助于Spring容器才能实现，此时就必须让该Bean先获取Spring容器，然后借助于Spring容器实现该功能。为了让Bean获取它所在的Spring容器，可以让该Bean实现ApplicationContextAware接口。ApplicationContextAware 通过它Spring容器会自动把上下文环境对象调用ApplicationContextAware接口中的setApplicationContext方法。在ApplicationContextAware的实现类中，就可以通过这个上下文环境对象得到Spring容器中的Bean。看到—Aware就知道是干什么的了，就是属性注入的，但是这个ApplicationContextAware的不同地方在于，实现了这个接口的bean，当spring容器初始化的时候，会自动的将ApplicationContext注入进来。

![SpringBoot进阶教程(六十九)ApplicationContextAware](E:\Development\Typora\images\506684-20201210153414928-1580224793.png)

## 1.添加实现ApplicationContextAware的工具类

[![复制代码](E:\Development\Typora\images\copycode-16595978211582.gif)](javascript:void(0);)

```java
package learn.utils;

import org.springframework.beans.BeansException;
import org.springframework.context.ApplicationContext;
import org.springframework.context.ApplicationContextAware;
import org.springframework.stereotype.Component;

/**
 * @author toutou
 * @date by 2020/12
 * @des
 */
@Component
public class SpringContextUtil implements ApplicationContextAware {
    private static ApplicationContext applicationContext;

    @Override
    public void setApplicationContext(ApplicationContext context) throws BeansException {
        applicationContext = context;
    }

    public static ApplicationContext getApplicationContext() {
        return applicationContext;
    }
    //获取Bean
    public static <T> T getBean(Class<T> requiredType){
        return getApplicationContext().getBean(requiredType);
    }
    public static <T> T getBean(String name){
        return (T) getApplicationContext().getBean(name);
    }
}
```

[![复制代码](E:\Development\Typora\images\copycode-16595978211582.gif)](javascript:void(0);)

## 2. 接口中直接调用

```java
    @GetMapping("home")
    public Result getUser(){
        UserAccountService userAccountService = SpringContextUtil.getBean(UserAccountService.class);
        return Result.setSuccessResult(userAccountService.getUserAccountById(1));
    }
```

## 3. service中内部调用

由于项目中配置了多个数据源，若所有mybatis mapper访问都集中在单个service方法中，@AutoDBDecision声明的数据源会串。所以需要颗粒化。这样就会在单个service中需要请求内部的方法，这时候也可以用上ApplicationContextAware工具类。

### 3.1 service中声明内部专用调用方法inside()

[![复制代码](E:\Development\Typora\images\copycode-16595978211582.gif)](javascript:void(0);)

```java
/**
 * @author toutou
 * @date by 2020/12
 * des https://www.cnblogs.com/toutou/
 */
public interface UserAccountService {
    default UserAccountService inside() {
        return SpringContextUtil.getBean(UserAccountService.class);
    }

    UserAccountVO getUserAccountById(Integer id);

    UserAccountVO getUserAccountById2(Integer id);
}
```

[![复制代码](E:\Development\Typora\images\copycode-16595978211582.gif)](javascript:void(0);)

### 3.2 impl中调用方法

[![复制代码](E:\Development\Typora\images\copycode-16595978211582.gif)](javascript:void(0);)

```java
/**
 * @author toutou
 * @date by 2020/12
 * des https://www.cnblogs.com/toutou/
 */
@Service
public class UserAccountServiceImpl implements UserAccountService{
    @Autowired
    UserAccountMapper userMapper;

    public UserAccountVO getUserAccountById(Integer id){
        UserAccountVO accountVO = null;
        UserAccount account = userMapper.selectByPrimaryKey(id);
        if (account != null) {
            accountVO = new UserAccountVO();
            accountVO.setId(account.getId());
            accountVO.setAccount(account.getAccount());
            accountVO.setAge(account.getAge());
            accountVO.setEmail(account.getEmail());
            accountVO.setUsername(account.getUsername());
            accountVO.setPhone(account.getPhone());
        }

        return accountVO;
    }

    public UserAccountVO getUserAccountById2(Integer id){
        return inside().getUserAccountById(id);
    }
}
```