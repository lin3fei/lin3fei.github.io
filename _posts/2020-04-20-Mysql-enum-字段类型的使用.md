---
layout:     post
title:      MySql enum 字段类型的使用
subtitle:   MySql enum
date:       2020-04-20
author:     Lin3Fei
header-img: img/post-bg-swift.jpg
catalog: true
tags:
    - MySql
    - Java
    - enum
---

#### 为什么使用枚举

限定值的取值范围，比如性别（男，女，未知）等

#### 枚举类型使用陷阱

##### 不推荐

超级不推荐在mysql中设置某一字段类型为enum，但是存的值为数字，比如'0'，'1'，'2'，具体原因及解决

- 你会混淆，因为enum可以通过角标取值，但它的角标是从1开始，对于不熟悉这个字段的人这里会出错

- enum类型的字段对于0与‘0’有非常大的区别，如果你是用0当角标做操作，因它没有这个角标，所要会报错；如果你使用‘0’这个值去取枚举值，并做插入操作，你会发现它竟然会成功，但是插入的结果是一个“空”（不是null）

- enum类型对于php等弱语言类型的支持很差，弱语言类型打引号和不打引号的值可能是同一类型，但是对于mysql中enum类型的字段来说，那就不一定是一回事了

- **总结：** 不要拿mysql的enum类型取存一些数字；如果你一定要使用这个字段去存数字，请把这个字段定义为int，然后在java代码中使用枚举类做一个对于这个字段值范围的一个限定

##### 报错
 
你可能会报这个错——Caused by: java.sql.SQLException: Data truncated for column 'Color' at row 1，原因及解决

- Jpa默认使用整数顺序值持久化枚举类型

- Mysql中枚举类型Color定义取值的顺序是RED、GREEN、BLUE，因此，当这三个取值持久化到数据库表时，取值分别是0、1、2

- 意思就是我们这里存往数据库的数据是0、1、2这样的数字，而不是RED、GREEN、BLUE字符串， 但是Mysql数据库中定义的是RED、GREEN、BLUE，并没有其它值所以报错

- **解决：** 在entity中使用@Enumerated(EnumType.STRING)标注你的枚举类型属性，如果标注，默认是Integer

#### 使用例子

##### 插入非数字

- 建表语句

```
create table user (
  id int primary key auto_increment,  
  name varchar(50) not null,
  sex enum('MALE','FEMALE') default 'FEMALE' not null
)
```

- Java 代码中的枚举类

```
public enum Sex {
    MALE, FEMALE
}
```

- Java 代码中的User对象

```
@Data
public class User {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)  
    private Integer id;	
    private String name;
    @Enumerated(EnumType.STRING)
    private Sex sex;
}
```

- 使用

```
    @Autowired
    private UserMapper userMapper;

    public void testSaveUser() {
        User user = new User();
        user.setName("小小");
        user.setSex(Sex.FEMALE);
        userMapper.insert(user);
    }
```

- 结果

id | name | sex 
---|---|---
1|小小|女

##### 插入数字

- 建表语句

```
create table user (
  id int primary key auto_increment,  
  name varchar(50) not null,
  sex int not null default 0 comment '0:女，1:男'
)
```

- Java 代码中的枚举类

```
public enum Sex {

    MALE(1, "男"), FEMALE(0, "女");

    private Integer code;

    private String sex;

    private Sex(Integer code, String sex) {
        this.code = code;
        this.sex = sex;
    }

    public Integer getCode() {
        return code;
    }

    public void setCode(Integer code) {
        this.code = code;
    }

    public String getSex() {
        return sex;
    }

    public void setSex(String sex) {
        this.sex = sex;
    }

}
```

- Java 代码中的User对象

```
@Data
public class User {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)  
    private Integer id;	
    private String name;
    private Sex sex;
}
```

- 使用

```
    @Autowired
    private UserMapper userMapper;

    public void testSaveUser() {
        User user = new User();
        user.setName("小小");
        user.setSex(Sex.FEMALE);
        userMapper.insert(user);
    }
```

- 结果

id | name | sex 
---|---|---
1|小小|0
