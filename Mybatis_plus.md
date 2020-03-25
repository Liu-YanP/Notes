# mybatis_plus

在mybatis的基础上增加了许多新特性。

引入依赖：

```xml
 <!-- 整合spring -->
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-tx</artifactId>
      <version>5.2.3.RELEASE</version>
    </dependency>

    <dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-jdbc</artifactId>
    <version>5.2.3.RELEASE</version>
    </dependency>	

<!-- mysql jdbc -->
    <dependency>
      <groupId>mysql</groupId>
      <artifactId>mysql-connector-java</artifactId>
      <version>8.0.17</version>
    </dependency>

    <!--mybatis plus -->
    <dependency>
      <groupId>com.baomidou</groupId>
      <artifactId>mybatis-plus</artifactId>
      <version>3.3.0</version>
    </dependency>

    <!-- druid连接池 -->
    <dependency>
      <groupId>com.alibaba</groupId>
      <artifactId>druid</artifactId>
      <version>1.1.21</version>
    </dependency>
```

**配置mybatis_plus**

```xml
<!--定义sqlSessionFactory-->
    <bean id="sqlSessionFactory" class="com.baomidou.mybatisplus.extension.spring.MybatisSqlSessionFactoryBean">
        <property name="dataSource" ref="dataSource"/>
        <property name="globalConfig" ref="globalConfig"/>
        <property name="plugins">
            <array>
                <bean class="com.baomidou.mybatisplus.extension.plugins.PaginationInterceptor"/>
            </array>
        </property>
    </bean>

    <bean id="globalConfig" class="com.baomidou.mybatisplus.core.config.GlobalConfig">
        <property name="dbConfig" ref="dbConfig"/>
    </bean>

    <bean id="dbConfig" class="com.baomidou.mybatisplus.core.config.GlobalConfig.DbConfig">
        <property name="keyGenerator" ref="keyGenerator"/>
    </bean>

    <bean id="keyGenerator" class="com.baomidou.mybatisplus.extension.incrementer.H2KeyGenerator"/>

    <!--扫描mapper所在的包-->
    <bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
        <property name="basePackage" value="com.liu.dao"/>
    </bean>
```

## 基本使用

**定义实体类**

`@TableName(“name”)`:指定实体对应的表名

`@TableId`：用于指定该字段为主键

`@TableField("name")`：指定该字段对应表中的字段

```java
package com.liu.entity;

import java.time.LocalDateTime;

import com.baomidou.mybatisplus.annotation.TableField;
import com.baomidou.mybatisplus.annotation.TableId;
import com.baomidou.mybatisplus.annotation.TableName;

@TableName("user")  //默认类名为表名，若不一样可使用该标签指定表明
public class User {
	
    //主键
    @TableId   //默认会使用id字段作为主键，进行自增操作。若主键不是id，可以使用该标签指定主键
    private Long id;
    //姓名
    @TableField("name")  //默认会使用驼峰转下划线，进行字段映射。若不一样可以使用该字段指定映射字段
    private String name;
    //年龄
    private Integer age;
    //电子邮件
    private String email;
    //上级
    private Long managerId;
    //创建时间
    private LocalDateTime createTime;
    
    //备注信息，在表中没有对应字段
    private transient String remark;
}
```

解决实体类中的字段在表中没有对应字段三种方法：

1. 定义字段时使用`transient`修饰
2. 定义字段时使用`static`修饰
3. 在该字段上使用注解`@TableFiled(exist=false)`

## 查询方法

### 基本查询方法

`T selectById(Long id)`：根据主键查询数据，返回实体对象

`List<T> selectBatchIds(List<Long> idsList)`: 根据多个id查询数据，返回list列表

`List<T> selectByMap(Map<string, object> map)` 根据map中设置的多个字段查询数据，返回list

*map中的key值是数据库中的列名，并不是实体中的字段名*

### 条件构造器查询

```java
/**
 * 查询姓名中带“雨”，且年龄小于40岁的人
 * name like '%雨%‘ and age<40
 */
@Test
public void selectByWrapper(){
    QueryWrapper<User> queryWrapper = new QueryWrapper<User>();
    queryWrapper.like("name","雨").le("age",40);
    List<User> userList = userMapper.selectList(queryWrapper);
    userList.forEach(System.out::println);
}

/**
 * 查询姓“王”或者年龄大于等于25岁的人，并且按年龄倒序，id升序排列
 * name like "王%" or age>=25 order by age desc,id asc;
 */
@Test
public void selectByWrapper2(){
    QueryWrapper<User> queryWrapper = new QueryWrapper<User>();
    queryWrapper.likeRight("name","王").or().ge("age",30).orderByDesc("age").orderByAsc("id");
    List<User> userList = userMapper.selectList(queryWrapper);
    userList.forEach(System.out::println);
}
```

## 逻辑删除

在表中设置一个字段，标记该行数据是否删除。不在实际数据库中删除。

实体类字段上加上`@TableLogic`注解

```java
@TableLogic
private Integer deleted;
```

## 乐观锁

意图：

当要更新一条记录的时候，希望这条记录没有被别人更新

乐观锁实现方式：

- 取出记录时，获取当前version
- 更新时，带上这个version
- 执行更新时， set version = newVersion where version = oldVersion
- 如果version不对，就更新失败

### 配置

```xml
<bean class="com.baomidou.mybatisplus.extension.plugins.OptimisticLockerInterceptor"/>
```

### 注解实体字段 `@Version` 必须要!

```java
@Version
private Integer version;
```

**在 `update(entity, wrapper)` 方法下, `wrapper` 不能复用!!!**

## 自动填充

编写自动填充处理类。

```java
package com.liu.tool;

import com.baomidou.mybatisplus.core.handlers.MetaObjectHandler;
import org.apache.ibatis.reflection.MetaObject;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.stereotype.Component;

import java.time.LocalDateTime;

public class MyMetaObjectHandler implements MetaObjectHandler {
    Logger logger = LoggerFactory.getLogger(MyMetaObjectHandler.class);

    @Override
    public void insertFill(MetaObject metaObject) {

        Object value = metaObject.getValue("createTime");
        logger.info("插入时填充！"+value);
        if (value==null){
            this.strictInsertFill(metaObject,"createTime",LocalDateTime.class, LocalDateTime.now());
        }
    }

    @Override
    public void updateFill(MetaObject metaObject) {
        Object value = metaObject.getValue("updateTime");
        logger.info("更新时填充！"+value);
        if (value==null){  //若设置了不自动填充，没设置自动填充
            this.strictUpdateFill(metaObject,"updateTime",LocalDateTime.class,LocalDateTime.now());

        }
    }
}
```

配置进globalconfig参数

```xml
<bean id="globalConfig" class="com.baomidou.mybatisplus.core.config.GlobalConfig">
    <property name="dbConfig" ref="dbConfig"/>
        <!--设置自动填充-->
    <property name="metaObjectHandler" >
        <bean class="com.liu.tool.MyMetaObjectHandler"/>
    </property>
</bean>
```

设置实体注解

```java
@TableField(fill = FieldFill.INSERT)  //自动填充，插入时填充
private LocalDateTime createTime;

@TableField(fill = FieldFill.UPDATE)  //更新时填充
private LocalDateTime updateTime;
```