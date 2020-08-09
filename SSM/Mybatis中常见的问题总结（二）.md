---
title: Mybatis中常见的问题总结（二）
date: 2017-06-11 19:56:24
tags: 
    - Mybatis
---
<meta name="referrer" content="no-referrer" />

## 参数为集合

**Q：parameterType指的的类型是集合类型还是对象？**

```yaml
A：都可以，甚至不用在xml中指定也可以。

第一，mybatis会对传入的参数进行判断是不是list或者array，

第二，mybatis是根据ONGL表达式，即 【参数.属性】 这样的格式，通过反射去获取和注入属性值，传入的参数为集合的时候，不管指定parameterType的那一个，上面说的两点都能发挥功能。

```

**Q：Parameter '__frch_item_0' not found. Available parameters are [list]**

```yaml
A：首先说明的是frch_item_0这个参数名的来历，在我的xml文件中，<foreach>是这样定义的:

<foreach collection="list" item="item" separator="," open="(" close=")">
            #{item.a}
</foreach>

"frch"代表的是在循环内部参数的前缀，"item" 则标签中指定的item的名字，后面的数字则代表循环中的第几个参数。

接口中的方法的定义如下：
List<User> listById( List<User> id);

这里的item.a等同user.a，但是User类里根本没有a这个属性，所以出现这个问题应该 [第一时间去查看访问的属性名是否正确]
```

**Q：Parameter 'list' not found. Available parameters are [0, 1, param1, param2]**

```yaml
A： 首先解释Available parameters are [0, 1, param1, param2]，

mybatis会将传入的参数按照顺序设定默认名字，即这里看到的0，1，param1，param2，通过@Param注解可以修改参数名，当传入的参数有且仅有一个的时候，可以不使用@Param注解，

在定义的sql语句中，#{}里写的参数是随意的！因为传入的参数只有一个，无论在#{}命名为什么，mybatis都是将这个唯一参数传入，但是在使用@Param注解的情况，就必须使用注解指定的名字或者paramX(X指代顺序数字)，

回到本题，在使用List作为唯一参数的时候，在foreach标签使用的情况下是不会出现上述问题。
```
List作为唯一参数的时候:
```yaml
List<User> listById( List<String> id);
<select id="listById" parameterType="scau.zzf.entity.User"  resultType="scau.zzf.entity.User">
        SELECT * FROM users WHERE id IN
        <foreach collection="list" item="item" separator="," open="(" close=")">
            #{item}
        </foreach>
 </select>
```
但是在传入的参数为多个的时候:
```yaml
List<User> listById( List<String> id,String username); //不使用@Param
 <select id="listById" parameterType="scau.zzf.entity.User"  resultType="scau.zzf.entity.User">
        SELECT * FROM users WHERE username=#{username} AND id IN
        <foreach collection="list" item="item" separator="," open="(" close=")">
            #{item.id}
        </foreach>
</select>
```
此刻错误日志显示为 Parameter 'list' not found. Available parameters are [0, 1, param1, param2]。

这个时候我们自然就想到，**在多个参数的时候必须用@Param 指定参数名**，才能够使用我们的参数规范，否则就必须依据mybatis的参数规范。

但是对于新手来说，可能还差了一步，foreach标签中collection的选择比较多的是list或者array，在传入的参数只有一个的时候，这是绝对正确的，但在多个的时候，这里的collection要依照@Param指定的名称。

## 模糊查询怎么做

新手涉及到使用mybatis做模糊查询的使用，会首先在xml文件里定义sql的时候尽量拼接上%，

然后碰壁，怎么尝试都不对，最后想到了，在传入参数之前添加上%，结果问题迎刃而解，

但是这个方式还不够优雅，可以通过bind标签实现该功能。

```yaml
<select id="findByUsername" resultType="scau.zzf.entity.User">
    <bind name="pattern" value="username+'%'"></bind>
        SELECT * FROM users WHERE username LIKE #{pattern}
</select>
```

## 关联查询引用的数据混乱

**Q：一对多关联查询，多方只返回一条数据，但SQL书写是正确的**
```yaml
A：这种情况经常是查询返回的主键名字是相同的，从而导致返回结果出错。

两张表的数据库字段相同，是难以避免的，需要结合使用 SQL语句别名和resultMap一起使用即可解决。
```
## 如何获取插入对象后该对象的主键值

```yaml
insert成功之后，mybatis会将插入的值自动绑定到插入的对象的Id属性中，以下代码是能正确打印出ID值的。
 <insert id="addUser" >
        INSERT INTO users  (id,username,password,salt) VALUES (#{id},#{username},#{password},#{salt})
 </insert>
 
 iUserService.addUser(user);//插入成功
 System.out.println(user.getId());//打印ID
 
 注：如果使用的主键名不是ID，可以通过修改insert标签中的keyProperty或者keyColumn。
```

## 无法触发缓存刷新
```yaml
简单的来说，这种情况发生在关联查询的时候，但是更新的却只是一张表，查却查的是两张表，

所有当更新一张表的数据的时候，无法触发刷新查询两张表的缓存数据。
```
## org.apache.ibatis.binding.BindingException: Invalid bound statement (not found)

```yaml
找不到接口声明，一般情况下重新检查下DAO和xml文件对应的id是否一致就可以解决了。

这里要提出的是另一种情况，在IntellJ IDEA中，maven项目工程默认是只扫描resourses文件下的资源，也就是说如果你在IntellJ IDEA中的maven项目中，要想在src下放xml文件，必须在pom.xml中加入以下代码
```
```yaml
<build>
        <finalName>${project.artifactId}</finalName>
        <resources>
            <resource>
                <directory>src/main/java</directory>
                <includes>
                    <include>**/*.xml</include>
                </includes>
                <filtering>true</filtering>
            </resource>
            <resource>
                <directory>src/main/resources</directory>
                <includes>
                    <include>**/*.properties</include>
                    <include>**/*.xml</include>
                </includes>
                <filtering>true</filtering>
            </resource>
        </resources>
</bulid>
```