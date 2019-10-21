---
title: lombok 使用介绍
date: 2019-02-19 19:56:24
tags: 
    - Java
---

## Lombok使用

在项目中使用Lombok可以减少很多重复代码的书写。比如说getter/setter/toString等方法的编写。

## IDEA中的安装
打开IDEA的Setting –> 选择Plugins选项 –> 选择Browse repositories –> 搜索lombok –> 点击安装 –> 安装完成重启IDEA –> 安装成功

![](https://leanote.com/api/file/getImage?fileId=5a54370fab644126e2000c44)

## 引入依赖

在项目中添加Lombok依赖jar，在pom文件中添加如下部分。(不清楚版本可以在Maven仓库中搜索)

```yaml
<dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
    <version>1.16.18</version>
    <scope>provided</scope>
</dependency>
```

## 使用
在对应的类或者方法上使用对应注解即可。 

原来的代码：
```java
public class test1 {
    String name;
    String age;
    String sex;

    public static test1.test1Builder builder() {
        return new test1.test1Builder();
    }

    public String getName() {
        return this.name;
    }

    public String getAge() {
        return this.age;
    }

    public String getSex() {
        return this.sex;
    }

    public void setName(String name) {
        this.name = name;
    }

    public void setAge(String age) {
        this.age = age;
    }

    public void setSex(String sex) {
        this.sex = sex;
    }

    public boolean equals(Object o) {
        if (o == this) {
            return true;
        } else if (!(o instanceof test1)) {
            return false;
        } else {
            test1 other = (test1)o;
            if (!other.canEqual(this)) {
                return false;
            } else {
                label47: {
                    Object this$name = this.getName();
                    Object other$name = other.getName();
                    if (this$name == null) {
                        if (other$name == null) {
                            break label47;
                        }
                    } else if (this$name.equals(other$name)) {
                        break label47;
                    }

                    return false;
                }

                Object this$age = this.getAge();
                Object other$age = other.getAge();
                if (this$age == null) {
                    if (other$age != null) {
                        return false;
                    }
                } else if (!this$age.equals(other$age)) {
                    return false;
                }

                Object this$sex = this.getSex();
                Object other$sex = other.getSex();
                if (this$sex == null) {
                    if (other$sex != null) {
                        return false;
                    }
                } else if (!this$sex.equals(other$sex)) {
                    return false;
                }

                return true;
            }
        }
    }

    protected boolean canEqual(Object other) {
        return other instanceof test1;
    }

    public int hashCode() {
        int PRIME = true;
        int result = 1;
        Object $name = this.getName();
        int result = result * 59 + ($name == null ? 43 : $name.hashCode());
        Object $age = this.getAge();
        result = result * 59 + ($age == null ? 43 : $age.hashCode());
        Object $sex = this.getSex();
        result = result * 59 + ($sex == null ? 43 : $sex.hashCode());
        return result;
    }

    public String toString() {
        return "test1(name=" + this.getName() + ", age=" + this.getAge() + ", sex=" + this.getSex() + ")";
    }

    @ConstructorProperties({"name", "age", "sex"})
    public test1(String name, String age, String sex) {
        this.name = name;
        this.age = age;
        this.sex = sex;
    }

    public test1() {
    }

    public static class test1Builder {
        private String name;
        private String age;
        private String sex;

        test1Builder() {
        }

        public test1.test1Builder name(String name) {
            this.name = name;
            return this;
        }

        public test1.test1Builder age(String age) {
            this.age = age;
            return this;
        }

        public test1.test1Builder sex(String sex) {
            this.sex = sex;
            return this;
        }

        public test1 build() {
            return new test1(this.name, this.age, this.sex);
        }

        public String toString() {
            return "test1.test1Builder(name=" + this.name + ", age=" + this.age + ", sex=" + this.sex + ")";
        }
    }
}
```

现在的代码：

```java
@Data //生成getter,setter等函数
@AllArgsConstructor //生成全参数构造函数
@NoArgsConstructor//生成无参构造函数
@Builder
public class test1 {
    String name;
    String age;
    String sex;
}
```

代码测试：

```java
 public static void main(String[] args) {
        //使用@Builder注解后，可以直接通过Builder设置字段参数
        test1 t1=new test1.test1Builder()
                .name("wang")
                .age("12")
                .sex("man")
                .build();

        System.out.println("name is"+t1.getName()+'\n'+"age is :"+t1.getAge());

    }
```
## Lombok有哪些注解
- @Setter:注解在属性上；为属性提供 setting 方法
- @Getter: 注解在属性上；为属性提供 getting 方法
- @Data:注解在类上；提供类所有属性的 getting 和 setting 方法，此外还提供了equals、canEqual、hashCode、toString 方法

- @Log(这是一个泛型注解，具体有很多种形式)
- @AllArgsConstructor:注解在类上；为类提供一个全参的构造方法
- @NoArgsConstructor:注解在类上；为类提供一个无参的构造方法
- @EqualsAndHashCode
- @Builder
- @NonNull
- @Cleanup
- @ToString
- @RequiredArgsConstructor
- @Value
- @SneakyThrows
- @Synchronized
