# Cron[表达式](#)

> Cron表达式的长度为6或7位，其中第7位年份可省略，省略时表示每年。

![在这里插入图片描述](E:\Development\Typora\images\de7f31df94fe4cec92f2e4dd30aa842a.png)

`@Scheduled(cron="* * * * * * *")` 这7位从左到右分别对应：
{秒数} {分钟} {小时} {日期} {月份} {星期} {年份(可为空)}

| 字段         | 允许值            | 允许字符        |
| ------------ | ----------------- | --------------- |
| 秒           | 0-59              | , - * /         |
| 分           | 0-59              | , - * /         |
| 小时         | 0-23              | , - * /         |
| 日期         | 1-31              | , - * ? / L W C |
| 月份         | 1-12 或 JAN-DEC   | , - * /         |
| 星期         | 1-7 或 SUN-SAT    | , - * ? / L C # |
| 年（可为空） | 留空 或 1970-2099 | , - * /         |

## 一、秒

​		**允许值范围: 0~59 ,不允许为空值**，若值不合法，调度器将抛出`SchedulerException`异常

`“*”` 代表每隔1秒钟触发

`“,”` 代表在指定的秒数触发：比如"0,15,45"代表0秒、15秒和45秒时触发任务

`“-”` 代表在指定的范围内触发：比如"25-45"代表从25秒开始触发到45秒结束触发，每隔1秒触发1次

`“/”` 代表触发步进(step)："/“前面的值代表初始值(”*“等同"0”)，后面的值代表偏移量

* `"0/20"`或者`"*/20"`代表从0秒钟开始，每隔20秒钟触发1次，即0秒触发1次，20秒触发1次，40秒触发1次；
* `"5/20"`代表5秒开始触发1次，25秒触发1次，45秒触发1次；
* `"10-45/20"`代表在[10,45]内步进20秒命中的时间点触发，即10秒触发1次，30秒触发1次

## 二、分钟

​		**允许值范围: 0~59 ,不允许为空值**，若值不合法，调度器将抛出`SchedulerException`异常

`“*”` 代表每隔1分钟触发

`“,”` 代表在指定的分钟触发，比如"10,20,40"代表10分钟、20分钟和40分钟时触发任务

`“-”` 代表在指定的范围内触发，比如"5-30"代表从5分钟开始触发到30分钟结束触 发，每隔1分钟触发

`“/”` 代表触发步进(step)，"/“前面的值代表初始值(”*“等同"0”)，后面的值代表偏移量*  **同上**

* 比如"0/25"或者"*/25"代表从0分钟开始，每隔25分钟触发1次，即0分钟触发1次，第25分钟触发1次，第50分钟触发1次；
* "5/25"代表5分钟触发1次，30分钟触发1次，55分钟触发1次；
* "10-45/20"代表在[10,45]内步进20分钟命中的时间点触发，即10分钟触发1次，30分钟触发1次

## 三、小时

​		**允许值范围: 0~23 ,不允许为空值**，若值不合法，调度器将抛出`SchedulerException`异常

`“*”` 代表每隔1小时触发

`“,”` 代表在指定的时间点触发，比如"10,20,23"代表10点钟、20点钟和23点触发任务

`“-”` 代表在指定的时间段内触发，比如"20-23"代表从20点开始触发到23点结束触发，每隔1小时触发

`“/”` 代表触发步进(step)，"/“前面的值代表初始值(”*“等同"0”)，后面的值代表偏移量，比如"0/1"或者"*/1"代表从0点开始触发，每隔1小时触发1次；"1/2"代表从1点开始触发，以后每隔2小时触发一次

## 四、日期

​		**允许值范围: 1-31 ,不允许为空值**，若值不合法，调度器将抛出`SchedulerException`异常

`“W”` 字符代表着平日(Mon-Fri)，并且仅能用于日域中。它用来指定离指定日的最近的一个平日。大部分的商业处理都是基于工作周的，所以 W 字符可能是非常重要的。
例如，日域中的 15W 意味着 “离该月15号的最近一个平日。” 假如15号是星期六，那么 trigger 会在14号(星期五)触发，因为星期四比星期一离15号更近。

`“C”`：代表“Calendar”的意思。它的意思是计划所关联的日期，如果日期没有被关联，则相当于日历中所有日期。例如5C在日期字段中就相当于日历5日以后的第一天。1C在星期字段中相当于星期日后的第一天。

## 五、月份

​		**允许值范围: 1~12 (JAN-DEC),不允许为空值，若值不合法**，调度器将抛出`SchedulerException`异常

`“*”` 代表每个月都触发

`“,”` 代表在指定的月份触发，比如"1,6,12"代表1月份、6月份和12月份触发任务

`“-”` 代表在指定的月份范围内触发，比如"1-6"代表从1月份开始触发到6月份结束触发，每隔1个月触发

`“/”` 代表触发步进(step)，"/“前面的值代表初始值(”*“等同"1”)，后面的值代表偏移量，比如"1/2"或者"*/2"代表从1月份开始触发，每隔2个月触发1次；"6/6"代表从6月份开始触发，以后每隔6个月触发一次；"1-6/12"表达式意味着每年1月份触发

## 六、星期

​		**允许值范围: 1~7 (SUN-SAT),1代表星期天(一星期的第一天)**，以此类推，7代表星期六(一星期的最后一天)，不允许为空值，若值不合法，调度器将抛出`SchedulerException`异常

`“*”` 代表每星期都触发；

`“?”` 与{日期}互斥，即意味着若明确指定{日期}触发，则表示{星期}无意义，以免引起冲突和混乱

`“,”` 代表在指定的星期约定触发，比如"1,3,5"代表星期天、星期二和星期四触发

`“-”` 代表在指定的星期范围内触发，比如"2-4"代表从星期一开始触发到星期三结束触发，每隔1天触发

`“/”` 代表触发步进(step)，"/“前面的值代表初始值(”*“等同"1”)，后面的值代表偏移量，比如"1/3"或者"*/3"代表从星期天开始触发，每隔3天触发1次；"1-5/2"表达式意味着在[1,5]范围内，每隔2天触发，即星期天、星期二、星期四触发

`“L”` 如果{星期}占位符如果是"L"，即意味着星期的的最后一天触发，即星期六触发，L= 7或者 L = SAT，因此，"5L"意味着一个月的最后一个星期四触发

`“#”` 用来指定具体的周数，"#“前面代表星期，”#"后面代表本月第几周，比如"2#2"表示本月第二周的星期一，"5#3"表示本月第三周的星期四，因此，“5L"这种形式只不过是”#"的特殊形式而已

## 七、年份

**允许值范围: 1970~2099 ,允许为空，若值不合法**，调度器将抛出`SchedulerException`异常

`"*"`代表每年都触发

`","`代表在指定的年份才触发，比如"2011,2012,2013"代表2011年、2012年和2013年触发任务

`"-"`代表在指定的年份范围内触发，比如"2011-2020"代表从2011年开始触发到2020年结束触发，每隔1年触发

`"/“`代表触发步进(step)，”/“前面的值代表初始值(”*“等同"1970”)，后面的值代表偏移量，比如"2011/2"或者"*/2"代表从2011年开始触发，每隔2年触发1次

**注意：除了{日期}和{星期}可以使用"?"来实现互斥，表达无意义的信息之外，其他占位符都要具有具体的时间含义，且依赖关系为：年->月->日期(星期)->小时->分钟->秒数**

## 八、特殊字符

| 字符 | 作用                                      | 举例                                                         |
| ---- | ----------------------------------------- | ------------------------------------------------------------ |
| ,    | 列出枚举值                                | 在Minutes域使用5,10，表示在5分和10分各触发一次               |
| -    | 表示触发范围                              | 在Minutes域使用5-10，表示从5分到10分钟每分钟触发一次         |
| *    | 匹配任意值                                | 在Minutes域使用*, 表示每分钟都会触发一次                     |
| /    | 起始时间开始触发，每隔固定时间触发一次    | 在Minutes域使用5/10,表示5分时触发一次，每10分钟再触发一次    |
| ?    | 在DayofMonth和DayofWeek中，用于匹配任意值 | 在DayofMonth域使用?,表示每天都触发一次                       |
| #    | 在DayofMonth中，确定第几个星期几          | 1#3表示第三个星期日                                          |
| L    | 表示最后                                  | 在DayofWeek中使用5L,表示在最后一个星期四触发                 |
| W    | 表示有效工作日(周一到周五)                | 在DayofMonth使用5W，如果5日是星期六，则将在最近的工作日4日触发一次 |



**`“*”`**
“*”字符被用来指定所有的值。如："*"在分钟的字段域里表示“每分钟”。

**`“?”`**
“?”字符只在日期域和星期域中使用。它被用来指定“非明确的值”。当你需要通过在这两个域中的一个来指定一些东西的时候，它是有用的。看下面的例子你就会明白。 月份中的日期和星期中的日期这两个元素时互斥的一起应该通过设置一个问号来表明不想设置那个字段。

**`“-”`**
“-”字符被用来指定一个范围。如：“10-12”在小时域意味着“10点、11点、12点”。

**`“,”`**
“,”字符被用来指定另外的值。如：“MON,WED,FRI”在星期域里表示”星期一、星期三、星期五”。

**`“/”`**
“/”字符用于指定增量。如：“0/15”在秒域意思是每分钟的0，15，30和45秒。“5/15”在分钟域表示每小时的5，20，35和50。符号“*”在“/”前面（如：*/10）等价于0在“/”前面（如：0/10）。记住一条本质：表达式的每个数值域都是一个有最大值和最小值的集合，如：秒域和分钟域的集合是0-59，日期域是1-31，月份域是1-12。字符“/”可以帮助你在每个字符域中取相应的数值。如：“7/6”在月份域的时候只有当7月的时候才会触发，并不是表示每个6月。

**`“L”`**
L是‘last’的省略写法可以表示day-of-month和day-of-week域，但在两个字段中的意思不同，例如day-of-month域中表示一个月的最后一天。如果在day-of-week域表示‘7’或者‘SAT’，如果在day-of-week域中前面加上数字，它表示一个月的最后几天，例如‘6L’就表示一个月的最后一个星期五。

**`“W”`**
字符“W”只允许日期域出现。这个字符用于指定日期的最近工作日。例如：如果你在日期域中写 “15W”，表示：这个月15号最近的工作日。所以，如果15号是周六，则任务会在14号触发。如果15好是周日，则任务会在周一也就是16号触发。如果是在日期域填写“1W”即使1号是周六，那么任务也只会在下周一，也就是3号触发，“W”字符指定的最近工作日是不能够跨月份的。字符“W”只能配合一个单独的数值使用，不能够是一个数字段，如：1-15W是错误的。

“L”和“W”可以在日期域中联合使用，LW表示这个月最后一周的工作日。

**`“#”`**
字符“#”只允许在星期域中出现。这个字符用于指定本月的某某天。例如：“6#3”表示本月第三周的星期五（6表示星期五，3表示第三周）。“2#1”表示本月第一周的星期一。“4#5”表示第五周的星期三。

**`“C”`**
字符“C”允许在日期域和星期域出现。这个字符依靠一个指定的“日历”。也就是说这个表达式的值依赖于相关的“日历”的计算结果，如果没有“日历”关联，则等价于所有包含的“日历”。如：日期域是“5C”表示关联“日历”中第一天，或者这个月开始的第一天的后5天。星期域是“1C”表示关联“日历”中第一天，或者星期的第一天的后1天，也就是周日的后一天（周一）。

## 九、表达式举例

“0 0 12 * * ?” 每天中午12点触发

“0 15 10 ? * *” 每天上午10:15触发

“0 15 10 * * ?” 每天上午10:15触发

“0 15 10 * * ? *” 每天上午10:15触发

“0 15 10 * * ? 2005” 2005年的每天上午10:15触发

“0 * 14 * * ?” 在每天下午2点到下午2:59期间的每1分钟触发

“0 0/5 14 * * ?” 在每天下午2点到下午2:55期间的每5分钟触发

“0 0/5 14,18 * * ?” 在每天下午2点到2:55期间和下午6点到6:55期间的每5分钟触发

“0 0-5 14 * * ?” 在每天下午2点到下午2:05期间的每1分钟触发

“0 10,44 14 ? 3 WED” 每年三月的星期三的下午2:10和2:44触发

“0 15 10 ? * MON-FRI” 周一至周五的上午10:15触发

“0 15 10 15 * ?” 每月15日上午10:15触发

“0 15 10 L * ?” 每月最后一日的上午10:15触发

“0 15 10 ? * 6L” 每月的最后一个星期五上午10:15触发

“0 15 10 ? * 6L 2002-2005” 2002年至2005年的每月的最后一个星期五上午10:15触发

“0 15 10 ? * 6#3” 每月的第三个星期五上午10:15触发





## 十、在线CRON表达式生成器

其实CRON表达式无需多记，需要使用的时候直接使用在线生成器就可以了，地址：https://cron.qqe2.com/