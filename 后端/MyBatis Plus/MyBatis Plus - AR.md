# MyBatis Plus - AR

​		ActiveRecord （简称 AR ）一直广受动态语言（ PHP 、 Ruby 等）的喜爱，而 Java 作为准静态语言，对于

ActiveRecord 往往只能感叹其优雅，所以我们也在 AR 道路上进行了一定的探索，喜欢大家能够喜欢。

## 1.**什么是** **ActiveRecord** **？**

> ActiveRecord 也属于 ORM （对象关系映射）层，由 Rails 最早提出，遵循标准的 ORM 模型：表映射到记录，记
>
> 录映射到对象，字段映射到对象属性。配合遵循的命名和配置惯例，能够很大程度的快速实现模型的操作，而
>
> 且简洁易懂。

## 2.ActiveRecord 的主要思想是

- 每一个数据库表对应创建一个类，类的每一个对象实例对应于数据库中表的一行记录；通常表的每个字段
- 在类中都有相应的 Field ；
- ActiveRecord 同时负责把自己持久化，在 ActiveRecord 中封装了对数据库的访问，即 CURD; ；
- ActiveRecord 是一种领域模型 (Domain Model) ，封装了部分业务逻辑；
- 在 MP 中，开启 AR 非常简单，只需要将实体对象继承 Model 即可。

## 3.ActiveRecord的各项操作

```java
//AR：根据id查询
    @Test
    public void testSelectById(){
        User user = new User();
        user.setId(1L);
 
        User user1 = user.selectById();
        System.out.println(user1);
 
    }
 
    //AR：插入数据
    @Test
    public void testInsert(){
        User user=new User();
        user.setName("肖和");
        user.setUserName("xh");
        user.setAge(18);
        user.setMail("test6@itcast.cn");
        user.setPassword("123456");
        boolean insert = user.insert();
        System.out.println(insert);
    }
    //AR：更新数据
    @Test
    public void testUpdate(){
        User user=new User();
        user.setId(1L);
        user.setUserName("小胖");
        boolean result = user.updateById();
        System.out.println("result = > "+result);
    }
 
    //AR：删除数据
    @Test
    public void testDelete(){
        User user=new User();
 
        boolean result = user.deleteById(10L);
        System.out.println("result = > "+result);
    }
 
 
    //AR：根据条件查询数据数据
    @Test
    public void testSelect(){
        User user=new User();
        QueryWrapper<User> wrapper=new QueryWrapper<>();
        wrapper.eq("age",18);
        List<User> userList = user.selectList(wrapper);
        for (User user1 : userList) {
            System.out.println(user1);
        }
 
    }
```

