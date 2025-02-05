---
title: spark性能优化三之优化数据结构
categories: spark  
tags: [spark]
---


优化数据结构
要减少内存的消耗,除了使用高效的序列化类库以外,还有一个很重要的事情,就是优化数据结构,从而避免java语法特性中所导致的额外的内存开销,比如基于指针的java数据结构,以及包装类型

有一个关键的问题,就是优化什么数据结构,其实主要就是优化你的算子函数,内部使用到的局部数据,或者是算子函数外部的数据都可以进行数据结构的优化,优化之后,都会减少其对内存的消耗和占用


如何优化数据结构?
1.优先使用数组以及字符串,而不是集合类,也就是说,优先使用array,而不是ArrayList,LinkedList,HashMap等集合
比如,有个List<Integer> list = new ArrayList<Integer>(),将其替换为int[] arr = new int[] ,这样的话,array比List少了额外信息的存储开销,还能使用原始数据类型(int)来存储数据,比List中使用Integer这种包装类型存储数据要节省内存的多

还比如,通常企业级应用中的做法是,对于HashMap,List这种数据,统一用String拼接成特殊格式的字符串,比如Map<Integer,Person>= new HashMap<Integer,Person>(),可以优化为特殊的字符串格式:
```
id:name,address|id:name,address....
```

2.避免使用多层嵌套的对象结构,比如说:
```
public class Teacher{
	private List<Student> students = new ArrayList<Student>()

}
```
这就是不好的例子,因为Teacher类的内部又嵌套了大量的小Student对象

比如说,对于上述例子,也完全可以使用特殊的字符串来进行数据的存储,比如,用json字符串拉存储数据,就是一个很好的选择
```
{"teacherId":1,"teacherNameA":"leo",students:[{"studentId":1,"studentName":"tom"}]}

```



3.对于有些能够避免的场景,尽量使用int替代String,因为String虽然比ArrayList,HashMap等数据结构高效多了,占用内存少了,但是之前分析过,还有额外的信息的消耗,比如之前用string表示id,那么现在完全可以用数字类型的int,来进行替代

在这里提醒,在spark应用中,id就不要用常用的uuid了,因为无法转成int,就用自增的int类型的id即可
