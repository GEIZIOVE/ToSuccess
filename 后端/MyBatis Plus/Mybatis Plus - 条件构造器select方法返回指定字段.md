# [Mybatis](https://so.csdn.net/so/search?q=Mybatis&spm=1001.2101.3001.7020)-Plus条件构造器select方法返回指定字段

**条件构造器select用法**

> 1.返回特定的几个字段 select(字段…)
> 2.排除某几个字段 select(entiyClass,[predicate](https://so.csdn.net/so/search?q=predicate&spm=1001.2101.3001.7020)判断)
> 3.分组[聚合函数](https://so.csdn.net/so/search?q=聚合函数&spm=1001.2101.3001.7020) select(“聚合函数”)

## 搭配service的.listMaps方法,将返回的数据封装到map中

避免了多余字段的返回

### 实例一：指定需要查询的具体字段

```java
@Autowired
private ProjectBuildService projectBuildService;

public AjaxResult listAll( ) {
    //只返回了id和name两个字段    
    List<Map<String,Object>> list = projectBuildService.listMaps(Wrappers.<ProjectBuild>lambdaQuery().select(ProjectBuild::getBuildId,ProjectBuild::getBuildName)
           .orderByAsc(ProjectBuild::getBuildId)
           // 以上select表示要返回的字段,
    );
    return AjaxResult.success(list);
}
```

### 实例二：排除不需要返回的字段

```java
@Test
public void selectByQueryWrapper8(){
    QueryWrapper<Employee> queryWrapper=new QueryWrapper();
 
    queryWrapper.lambda()
             .select(Employee.class,fieldInfo->!fieldInfo.getColumn().equals("birthday")&&!fieldInfo.getColumn().equals("gender"))
             .gt(Employee::getSalary,3500)
             .like(Employee::getName,"小");
    List<Employee> employeeList = employeeMapper.selectList(queryWrapper);
    System.out.println(employeeList);
}
```

### 实例三：返回聚合函数值

sql实现：

```java
SELECT departmentId,AVG(salary) AS avg_salary FROM t_employee GROUP BY department_id;
1
@Test
public void selectByQueryWrapper9(){
    QueryWrapper<Employee> queryWrapper=new QueryWrapper();
  
    queryWrapper.select("department_id","AVG(salary) AS avg_salary")
             .groupBy("department_id");
    List<Employee> employeeList = employeeMapper.selectList(queryWrapper);
}
```

## 问题解决

**问题描述:** map中的key没有使用驼峰命名
**解决方法:** 使用hutool工具包中的 MapUtil.toCamelCaseMap(map);

```java
//处理map中key未驼峰命名
list = list.stream().map(MapUtil::toCamelCaseMap).collect(Collectors.toList());
```