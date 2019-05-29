# ORM框架之Mybatis
---
### *Mapper接口
MyBatis通过`*Mapper interface`定义的方法，找到对应`*Mapper.xml`里的SQL语句, 同时也提供了JPA式的直接在接口方法上定义SQL的注解:

```java
public interface PersonMapper {

  // 执行PersonMapper.xml里id为`findAll`的SQL语句
  public List<Person> findAll();
 
  @Insert("INSERT INTO person (name, age) VALUES (#{name}, #{age})")
  public Integer save(Person person);
  
  // ...
}
```
#### MyBatis 常用注解
相对于Spring Data Jpa的 `@Query(nativeQuery)` 与 `@Modifying`, MyBatis提供:
`@Insert`, `@Select`, `@Update`, `@Delete`分别对应数据库的CRUD操作:
```java
@Insert("Insert into person(name, age) values (#{name}), #{age}")
public Integer save(Person person);
  
@Select("SELECT person.id, person.name FROM person WHERE person.id = #{id}")
Person getPerson(Integer id);

@Update("UPDATE person SET name = #{name} WHERE id = #{id}")
public void updatePerson(Person person);
 
@Delete("DELETE FROM person WHERE id = #{id}")
public void deletePersonById(Integer id);
```

`@Results` - 包含一组`@Result`列表:
`@Result`  – 指定数据库字段到实体类属性的映射关系, 也可以关联至其它对象:
```java
@Select("SELECT id, name FROM person WHERE id = #{id}")
@Results(value = {
  @Result(property = "id", column = "id"),
  @Result(property="name", column = "name"),
  @Result(property = "addresses", javaType = List.class, column = "address_id", 
    many = @Many(select = "getAddresses")
  )
})
public Person getPersonById(Integer id);

@Select("SELECT id, street FROM address WHERE id = #{id}")
public Address getAddresses(Integer id);
```
`@Result`的`@Many` – 定义一个一对多的关系映射, 上面的`select`指定的使用`getAddresses`方法从`address`表查询返回一个地址集合.
> 与`@Many`注解对应的, `@One`注解用来指定一个一对一的关系映射

`@MapKey` – 当需要把查询返回的列表转换成`Map`时, `@MapKey`用来指定`Map`的`Key`属性:
```java
@Select("select * from Person")
@MapKey("id")
Map<Integer, Person> getAllPerson();
```

`@Options` – 定义各种执行语句的配置:
```java
@Insert("INSERT INTO address (address) VALUES (#{address})")
@Options(useGeneratedKeys = false, keyProperty = "id" flushCache=true)
public Integer saveAddress(Address address);
```
### 动态SQL
动态SQL是MyBatis非常强大的一个特性, 可以用来构建复杂的SQL语句。

Java编码的方式 :
```java
// 使用`@SelectProvider`指定类与方法名, 用来生成最终的SQL语句
@SelectProvider(type = PersonSqlProvider.class, method="getPersonByName")
public List<Person> findPersons(Integer id, String name);

public class PersonSqlProvider {
  
// 动态SQL拼装
public String selectPersonLike(final String id, final String name) {  
  return new SQL() {{ 
    SELECT("P.ID, P.USERNAME, P.PASSWORD, P.FIRST_NAME, P.LAST_NAME"); 
    FROM("PERSON P");  
    if (id != null) { 
      WHERE("P.ID like #{id}");  
    }
    if (name != null) { 
      WHERE("P.FIRST_NAME like #{name}");  
    }
    ORDER_BY("P.LAST_NAME");  
  }}.toString();  
}
```
> 若`Mapper`中的方法名与SqlProvider中的方法名一样, `method`可以省略

XML的方式 :
```xml
<select id="countByUserId" parameterType="map" resultType="Map">  
	SELECT count(distinct o.user_id) userId, o.channel_id channelId
	FROM tb_order o 
	LEFT JOIN tb_user u ON o.user_id = u.user_id  
	WHERE o.status > 0 AND o.status != 6
	<if test="regStartTime != null"> 
		AND u.reg_time >= #{regStartTime}
	</if>  
	<if test="regEndTime != null">  
		AND u.reg_time < #{regEndTime}
	</if>  
	<if test="payStartTime != null">  
		AND o.create_time >= #{payStartTime}  
	</if>  
	<if test="payEndTime != null">  
		AND o.create_time < #{payEndTime}
	</if>  
	GROUP BY o.channel_id
</select>
```

### 存储过程
`@Select`注解同样也可能执行存储过程:
```java
// 使用`CALL`调用函数,并传递相应的参数
@Select(value= "{CALL getPersonByProc(#{id, mode=IN, jdbcType=INTEGER})}")
@Options(statementType = StatementType.CALLABLE)
public Person getPersonByProc(Integer id);
```
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTE4NDYyODcxOTgsLTQxNzc3MjY5N119
-->