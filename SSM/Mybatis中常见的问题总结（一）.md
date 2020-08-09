---
title: Mybatis中常见的问题总结（一）
date: 2017-06-09 19:56:24
tags: 
    - Mybatis
---
<meta name="referrer" content="no-referrer" />

## org.apache.ibatis.binding.BindingException: Parameter 'XXXX' not found.

该问题的产生主要是mybatis中实例对象与xml文件之间映射关系上方法传参出现问题：
1.当方法中不传参或者只传一个参数
```java
public interface UserMapper {
            List<User> selectAllUser();
            int selectById(int id);
            int insert(Student stu);
}
```
那么xml文件中可以直接使用#{}方式获取参数，无需加@Param
```yaml
<insert id="insert" parameterType="com.ysq.model.student"><!--对应insert方法传过来的参数类型-->
　　　　insert into inbox_template (name,age)
　　　　values (#{name},#{age})
        <!--因为传过来的只有stu一个参数，因此我们不需要使用对象.属性，直接使用属性即可-->
</insert>
```
2.当方法中传入多个参数，则我们需要给方法中的每个参数加上@Param，否则在xml使用#{传入的参数名}会报错误（找不到参数名）

```java
User userLogin(@Param("username") String username, @Param("password") String password);
```

3.当方法中传人的既有参数又有对象时，对象也需要注解，一般参数的直接取，对象的需要对象.属性
```java
int selectBySelective(@Param("page")int page,@Param("stu")Student stu);
```

xml映射文件：

```yaml
<select id="selectBySelective" resultMap="BaseResultMap">
        select
        <include refid="Base_Column_List" />
        from student
        where name = #{stu.name} and id = #{stu.id} limit 0 ,#{page}
</select>
```
这种情况我们可以优化：将参数和对象在封装成一个对象，然后再传人即可（类似分页时，我们需要传入pageSize，User等）如下：

```yaml
<update id="update" parameterType="com.yoho.crm.dal.model.student"><!--此时obj则是对应page中的属性了-->
 update inbox_template
 set 
 name = #{obj.name},
 age = #{obj.age}
 #这里obj为封装对象中的一个属性对象，如果我们需要这个属性对象中的属性，则需对象.属性获取
 where id = #{obj.id}
</update>
```
## MyBatis 传List参数 nested exception is org.apache.ibatis.binding.BindingException: Parameter 'idList' not found.

以下是相关代码：

Mapper.java
```java
List<Order> selectOrderByIdList(List<Integer> idList);
```
mapper.xml

```yaml
<select id="selectOrderByIdList" resultMap="BaseResultMap">
        select
        <include refid="Base_Column_List" />
        from order
        where order_id in
        <foreach collection="idList" item="orderId" open="(" close=")" separater=",">
        #{orderId}
        </foreach>
</select>
```

解决方案：
```java
List<Order> selectOrderByIdList(@Parame("idList")List<Integer> idList);
```

或：
```yaml
<select id="selectOrderByIdList" resultMap="BaseResultMap">
        select
        <include refid="Base_Column_List" />
        from order
        where order_id in
        <foreach collection="list" item="orderId" open="(" close=")" separater=",">
        #{orderId}
        </foreach>
</select>
```

原因：

foreach元素的属性主要有item，index，collection，open，separator，close。

item表示集合中每一个元素进行迭代时的别名，index指定一个名字，用于表示在迭代过程中，每次迭代到的位置，open表示该语句以什么开始，separator表示在每次进行迭代之间以什么符号作为分隔符，close表示以什么结束。

在使用foreach的时候最关键的也是最容易出错的就是collection属性，该属性是必须指定的，但是在不同情况下，该属性的值是不一样的，主要有一下3种情况： 

（1）如果传入的是单参数且参数类型是一个List的时候，collection属性值为list .

（2）如果传入的是单参数且参数类型是一个array数组的时候，collection的属性值为array .

（3）如果传入的参数是多个的时候，我们就需要把它们封装成一个Map了，当然单参数也可以封装成map，实际上如果你在传入参数的时候，在MyBatis里面也是会把它封装成一个Map的，map的key就是参数名，所以这个时候collection属性值就是传入的List或array对象在自己封装的map里面的key.

