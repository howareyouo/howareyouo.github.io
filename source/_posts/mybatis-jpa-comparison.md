# 扩展框架对比

## Spring Data JPA
Spring Data JPA 基于 JPA , 提供了 持久层(Repository) 的封装, 进一步简化了 **查询业务逻辑** 代码 :
- 按照约定的 **方法命名规则** 编写 DAO 层接口，就可以在不写接口实现的情况下，实现对数据库的访问和操作。
- 提供了除基本 CRUD 之外，分页、排序等功能。
- 配合 **Specification**，**ExampleMatcher** 与 **QueryDSL** (需插件支持)或者Native SQL来解决复杂的查询操作。


## tk.mybatis (通用Mapper) 
通用 Mapper 基于 Mybatis, 实现了任意 MyBatis 通用方法的框架，解决 MyBatis 使用中 90% 的基本操作，节省开发人员大量的时间:
- 通用 Mapper 实现了所有 mybatis-generator 生成 XML 中的方法外, 还添加了更多额外实用的方法。
- 对于复杂查询，提供通用的 `Example` 支持相关的单表操作。

----------------------

## 复杂查询比较

示例实体类 ：
```java
@lombok.Data
class User{
	@Id
	@GeneratedValue
	private Long id;
	private String username;
	private String nickname
	private String email;
	private String status;
}
```
### Spring Data JPA:

基于 **ExampleMatcher** :
>  还是别基于了， `ExampleMatcher` 有两个缺陷：
	1. 不支持嵌套与分组条件如：  `firstname = ?0 or (firstname = ?1 and lastname = ?2)`. 
	2. 只支持对`String`类型的`starts/contains/ends/regex`匹配和其它类型的`完全匹配`.

基于 **Specification** :
```java
@Service
class UserRepository extends CrudRepository<User, Long>, JpaSpecificationExecutor {

	public List<User> findUsers(String username, String nickname) {
	
		// 定义Specification
        Specification<Student> specification = new Specification<Student>(){
            @Override
            public Predicate toPredicate(Root<Student> root, CriteriaQuery<?> query, CriteriaBuilder cb) {
                // 用于暂时存放查询条件的集合
                List<Predicate> predicatesList = new ArrayList<>();

                // equal
                if (StringUtils.isNotEmpty(username)){
                    Predicate usernamePredicate = cb.equal(root.get("username"), username);
                    predicatesList.add(usernamePredicate );
                }
                
                // like
                if (StringUtils.isNotEmpty(nickname)){
                    Predicate nicknamePredicate = cb.like(root.get("nickname"), '%' + nickname + '%');
                    predicatesList.add(nicknamePredicate);
                }
                
                // order
                query.orderBy(cb.asc(root.get("id")));
                
                // 最终将查询条件拼好然后return
                Predicate[] predicates = new Predicate[predicatesList.size()];
                return cb.and(predicatesList.toArray(predicates));
            }
        };
        return repository.findAll(specification);
    }
}
```

### tk.mybatis (通用Mapper):
```java
@Service
class UserService {

	@Autowired UserMapper userMapper;
	
	public List<User> findUsers(String username, String nickname) {
	
		// 创建通用的Example
		Example example = new Example(User.class);
		Example.Criteria criteria = example.createCriteria();

         // equal
         if (StringUtils.isNotEmpty(username)){
             criteria.andEqualTo("username", username);
         }
         
         // like
         if (StringUtils.isNotEmpty(nickname)){
	         criteria.andLike("nickname", "%" + nickname + "%");
         }
         
         // order
        example.setOrderByClause("id desc");
        
        return userMapper.selectByExample(example);
    }
}
```

### 对比

|对比项     | Spring Data JPA | MyBatis + 通用Mapper | 优胜者 |
| -------- | --------------- | ------------------- | ---- |
|设计理念   | 基于JPA的再次封装<br>首创按方法名推断SQL，酷炫至极 | 重点关注关系到对象(Relation -> Object)的映射与动态SQL构建| Spring Data JPA |
|上手难度   | 起手不要太简单，但涉及到复杂查询时会引入众多概念:<br>Specification / ExampleMatcher / Projection / QueryDSL 等| 一如既往的简单: <br>MyBatis只做关系映射这一件事 | MyBatis |
|复杂SQL拼装| 支持, 但与代码混合, 可读性不好 |  可选XML进行拼装<br>多一个选择多一份自信 | MyBatis |
|慢查询优化 | 需要团队有理解JPA原理的大佬才行 |  无门栏，网上有较多案例  | MyBatis |

### 个人总结

1. 表关联较多、查询业务复杂的项目，优先MyBatis
2. 持续维护/迭代频繁的项目建议MyBatis，这种项目需要变化很灵活，对SQL的灵活修改要求较高
3. 小型项目，或者关系模型较为清晰稳定的项目，可以JPA
4. 微服架构下，一个微服务会对应自已的数据库，多表连接查询的需求会大大的降低，可以考虑JPA


<!--stackedit_data:
eyJoaXN0b3J5IjpbMTY0MzAyODQ3MV19
-->