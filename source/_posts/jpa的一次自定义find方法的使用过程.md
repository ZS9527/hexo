---
title: jpa的一次自定义find方法的使用过程
date: 2019-05-17 22:22:54
tags: jpa
---

# 一次jpa自定义查询方法的使用尝试过程

## 项目需求
目前客户有一个需求：每一个用户想要看到的帖子顺序都不一样，用户可以按照自己的喜好排列帖子顺序，并且可以手动把某个帖子置顶显示。
现在项目数据交互使用的框架是spring-boot-starter-data-jpa。之前因为项目的工期很赶，所写的代码为直接使用jpa的findAll方法即可满足查询。现在查询条件的变化后，我想要在原本的基础上改动的内容越小越好。

<!--more-->

## 尝试过程

### 第一次尝试

我尝试使用下面的命名方式去直接自定义查询方法，来根据userId属性查询所关联的权重表，再根据权重表来查询到帖子表进行排序。
然而这种方法只能查询到这个用户已经排序过的帖子，并不可以看得到没有和该用户关联的帖子。放弃

#### 自定义方法名
jpa框架在进行方法名解析时，会先把方法名多余的前缀截取掉，比如 find、findBy、read、readBy、get、getBy，然后对剩下部分进行解析。并且如果方法的最后一个参数是 Sort 或者 Pageable 类型，也会提取相关的信息，以便按规则进行排序或者分页查询。

在创建查询时，我们通过在方法名中使用属性名称来表达，比如 findByUserAddressZip ()。框架在解析该方法时，首先剔除 findBy，然后对剩下的属性进行解析，详细规则如下（此处假设该方法针对的域对象为 AccountInfo 类型）：

- 先判断 userAddressZip （根据 POJO 规范，首字母变为小写，下同）是否为 AccountInfo 的一个属性，如果是，则表示根据该属性进行查询；如果没有该属性，继续第二步；
- 从右往左截取第一个大写字母开头的字符串（此处为 Zip），然后检查剩下的字符串是否为 AccountInfo 的一个属性，如果是，则表示根据该属性进行查询；如果没有该属性，则重复第二步，继续从右往左截取；最后假设 user 为 AccountInfo 的一个属性；
- 接着处理剩下部分（ AddressZip ），先判断 user 所对应的类型是否有 addressZip 属性，如果有，则表示该方法最终是根据 "AccountInfo.user.addressZip" 的取值进行查询；否则继续按照步骤 2 的规则从右往左截取，最终表示根据 "AccountInfo.user.address.zip" 的值进行查询。

可能会存在一种特殊情况，比如 AccountInfo 包含一个 user 的属性，也有一个 userAddress 属性，此时会存在混淆。读者可以明确在属性之间加上下划线以显式表达意图，比如 "findByUser_AddressZip()" 或者 "findByUserAddress_Zip()"。

在查询时，通常需要同时根据多个属性进行查询，且查询的条件也格式各样（大于某个值、在某个范围等等），Spring Data JPA 为此提供了一些表达条件查询的关键字，大致如下：

- And --- 等价于 SQL 中的 and 关键字，比如 findByUsernameAndPassword(String user, Striang pwd)；
- Or --- 等价于 SQL 中的 or 关键字，比如 findByUsernameOrAddress(String user, String addr)；
- Between --- 等价于 SQL 中的 between 关键字，比如 findBySalaryBetween(int max, int min)；
- LessThan --- 等价于 SQL 中的 "<"，比如 findBySalaryLessThan(int max)；
- GreaterThan --- 等价于 SQL 中的">"，比如 findBySalaryGreaterThan(int min)；
- IsNull --- 等价于 SQL 中的 "is null"，比如 findByUsernameIsNull()；
- IsNotNull --- 等价于 SQL 中的 "is not null"，比如 findByUsernameIsNotNull()；
- NotNull --- 与 IsNotNull 等价；
- Like --- 等价于 SQL 中的 "like"，比如 findByUsernameLike(String user)；
- NotLike --- 等价于 SQL 中的 "not like"，比如 findByUsernameNotLike(String user)；
- OrderBy --- 等价于 SQL 中的 "order by"，比如 findByUsernameOrderBySalaryAsc(String user)；
- Not --- 等价于 SQL 中的 "！ ="，比如 findByUsernameNot(String user)；
- In --- 等价于 SQL 中的 "in"，比如 findByUsernameIn(Collection<String> userList) ，方法的参数可以是 Collection 类型，也可以是数组或者不定长参数；
- NotIn --- 等价于 SQL 中的 "not in"，比如 findByUsernameNotIn(Collection<String> userList) ，方法的参数可以是 Collection 类型，也可以是数组或者不定长参数；

### 第二次尝试

在网上搜索相关文章时突然发现了这个Api查询条件的限制类，仔细查找研究了一下。发现还是没有找到可以解决这个的方案。

#### Example matchers 

首先，仓库接口需要继承`QueryByExampleExecutor`接口（继承这个 `JpaSpecificationExecutor` 也行），这样会引入一组以Example作参数的方法。然后创建一个`ExampleMatcher`对象，最后再用`Example`的of方法构造相应的Example对象并传递给相关查询方法。

Example不仅仅局限于默认的设置。你可以给strings定义自己的默认值然后去匹配。使用`ExampleMatcher`绑定null和特定属性的设置。

```java
Person person = new Person();                          
person.setFirstname("Dave");                           

ExampleMatcher matcher = ExampleMatcher.matching()    
  .withMatcher("username", ExampleMatcher.GenericPropertyMatchers.startsWith())  //模糊查询匹配开头，即{username}%
  .withMatcher("address" ,ExampleMatcher.GenericPropertyMatchers.contains())  //全部模糊查询，即%{address}%
  .withIgnorePaths("lastname")  //忽略字段，即不管lastname是什么值都不加入查询条件                 
  .withIncludeNullValues()       
  .withStringMatcherEnding();                          

Example<Person> example = Example.of(person, matcher); 
```

其中：
- `Person person = new Person();` 创建一个domain对象实例。
- 设置属性值去查询。
- `ExampleMatcher matcher = ExampleMatcher.matching()` 创建一个 `ExampleMatcher` 让其可以使用，但没有多余的配置项。
- `.withIgnorePaths("lastname")` Construct a new ExampleMatcher to ignore the property path lastname。用来排除某个属性的查询。
- `.withIncludeNullValues()` Construct a new ExampleMatcher to ignore the property path lastname and to include null values。让空值也参与查询。
- `.withStringMatcherEnding();`  Construct a new ExampleMatcher to ignore the property path lastname, to include null values, and use perform suffix string matching。匹配后缀字符串
- `Example<Person> example = Example.of(person, matcher);` 根据domain对象和配置的`ExampleMatcher`对象来创建一个`Example`

还可以给个别的属性指定行为.(比如.`firstname`和`lastname`以及domain对象的嵌套属性`address.city`) 
- `.ignoreCase()` 可以调整他让他匹配大小写敏感的选项。
- `endsWith` 以`firstname` 结束的前模糊查询。
- `startWith` 以`lastname`开始的后模糊查询。

```java
ExampleMatcher matcher = ExampleMatcher.matching()
  .withMatcher("firstname", endsWith())
  .withMatcher("lastname", startsWith().ignoreCase());
}
```

### 第四次尝试
没办法，要改动的步骤越来越大。以上的方法都不行的前提下，我只好试了试 `Specification` 作为 `findAll` 的参数这种方法。可是虽然用起来要改动的代码很少，但是还是不能查询到我想要的查询结果。查询条件只能加在where上面，而我想要的是用户排过序的加入条件查询，没有排过序的也要排列在后面。

#### JpaSpecificationExecutor
首先，仓库接口要继承 `JpaSpecificationExecutor<T>` 这个类，之后就可以使用 `findAll(Specification<T> spec)`等方法了。

```java
Specification specification = new Specification() {
    @Override
    public Predicate toPredicate(Root root, CriteriaQuery query,
        CriteriaBuilder cb) {
        List<Predicate> list = new ArrayList<>();
        if (Objects.nonNull("1")) {
            Join<WeightSort, ProjectInfo> join = root.join("weightSort", JoinType.LEFT);
            list.add(cb.equal(join.get("user").get("username"), "zs"));
        }

        if (Objects.nonNull(param.getYear())) {
            list.add(cb.equal(root.get("year"), param.getYear()));
        }
        if (Objects.nonNull(param.getTag())) {
            list.add(cb.like(root.get("tag"), param.getTag() + "%"));
        }

        return query.where(list.toArray(new Predicate[0])).getRestriction();
    }
};
Page<ProjectInfo> projectInfoPageTest = projectInfoRepository.findAll(specification, page);
projectInfoPageTest.stream().forEach(projectInfo -> {
    System.out.println(projectInfo.getId());
});
```
其中：
- `join` 为外键关联查询，通过 `project` 类中的 `WeightSort weightSort；`中的 `User user` 中的 `String username`属性来作为条件查询。该条件加在where后面。
- `cb.equal` 为匹配查询，相当于where后面的=号属性查询。
- `cb.like` 为模糊匹配查询，相当于where后面的like属性查询。

上面的代码产生的sql语句为：
```sql
SELECT
	p.* 
FROM
	bs_project_info p
	LEFT JOIN bs_project_info_weight_sorts pw ON p.id = pw.project_info_id
	LEFT JOIN bs_weight_sort w ON w.id = pw.weight_sorts_id
	LEFT JOIN system_user u ON u.id = w.user_id
	WHERE
	 u.username = 'zs'
	 and p.year = 2019
	 and p.tag like 'sql%'
```
但是这种方法还是要写很多行代码，不如把之前的原本的 `findAll(Example example)` 利用起来。
代码如下：

```java
Example example = Example.of(ProjectInfo.builder().weightSort(null).build());
Page<ProjectInfo> pages = projectInfoRepository.findAll((root, query, cb) -> {
   List<Predicate> predicates = new ArrayList<>();
    predicates.add(QueryByExamplePredicateBuilder.getPredicate(root, cb, example));

    Join<WeightSort, ProjectInfo> join = root.join("weightSort", JoinType.LEFT);
    predicates.add(cb.equal(join.get("user").get("username"), "zs"));

    return query.where(predicates.toArray(new Predicate[0])).getRestriction();
}, page);
pages.stream().forEach(projectInfo -> {
    System.out.println(projectInfo.getId());
});
```

这样就可以把一些属性相等的条件放进 `Example` 类里，而且该类本就支持不加入null的条件查询。不用再去判断传入参数为null时不做条件查询。利用上jpa的动态条件查询，节省了很多行代码。



### 最终的结局
没办法，实在是没有找到可以解决这个问题的方法。只好直接使用原生sql语句来满足需求。

#### 原生sql，Query注释

@Query 注解的使用非常简单，只需在声明的方法上面标注该注解，同时提供一个 JP QL 查询语句即可，如下所示：

使用 @Query 提供自定义查询语句示例：

```java
@Query(value = "SELECT "
        + "p.* "
        + " FROM "
        + " bs_project_info p "
        + " LEFT JOIN ( bs_project_info_weight_sorts pw JOIN bs_weight_sort w ON w.id = pw.weight_sorts_id AND w.user_id = :#{#param.userId} )
        +  ON pw.project_info_id = p.id "
        + "where  "
        + " IF( :#{#param.name} IS NOT null, p.name= :#{#param.name},  1=1) and "
        + " IF( :#{#param.year} IS NOT null, p.year = :#{#param.year},  1=1) and "
        + " IF( :#{#param.review} IS NOT null, p.review = :#{#param.review},  1=1) "
        + "ORDER BY "
        + "w.weight desc, p.create_time desc limit :#{#param.pageStart}, :#{#param.sizeStart}", nativeQuery=true)
    List<ProjectInfo> findAllPageProjectByParam(@Param("param") FindProjectParam param);
```

输入参数 `FindProjectParam` 类：
```java
@ApiModel
@Data
@Builder
@NoArgsConstructor
@AllArgsConstructor
@FieldDefaults(level = AccessLevel.PRIVATE)
public class FindProjectParam {

    @ApiModelProperty(value ="名称(不填为全部)", example = Mock.SPECIALNAMEID)
    Long name;
    
    @ApiModelProperty(value ="年度(不填为全部)", example = "2019")
    Integer year;
    
    @ApiModelProperty(value ="项目状态(3-待提交，1-已提交)(不填为全部)", example = Mock.REVIEW)
    Integer review;
    
    @ApiModelProperty(value ="用户id", example = Mock.USERNAME)
    Long userId;
    
    @ApiModelProperty(value ="页数")
    Integer pageStart;
    
    @ApiModelProperty(value ="条数")
    Integer sizeStart;
}

```

查询 `ProjectInfo` 类（数据库表名为：bs_project_info）：
```java
package com.yiring.finance.domain.projectinfo;

import com.yiring.finance.domain.constructioncontent.ConstructionContent;
import com.yiring.finance.domain.dictionary.DictionaryType;
import com.yiring.finance.domain.guide.Guide;
import com.yiring.finance.domain.uploadfile.FileOperation;
import com.yiring.finance.domain.weightsort.WeightSort;
import io.swagger.annotations.ApiModelProperty;
import java.io.Serializable;
import java.time.LocalDateTime;
import java.util.HashSet;
import java.util.Set;
import javax.persistence.CascadeType;
import javax.persistence.Column;
import javax.persistence.Entity;
import javax.persistence.FetchType;
import javax.persistence.GeneratedValue;
import javax.persistence.GenerationType;
import javax.persistence.Id;
import javax.persistence.ManyToOne;
import javax.persistence.OneToMany;
import javax.persistence.Table;
import lombok.AccessLevel;
import lombok.AllArgsConstructor;
import lombok.Builder;
import lombok.Data;
import lombok.NoArgsConstructor;
import lombok.experimental.FieldDefaults;
import org.hibernate.annotaions.Comment;

/**
 * 项目
 *
 * @author zhangshuai
 * @date 2019/4/16 17:31
 */

@Data
@Builder
@NoArgsConstructor
@AllArgsConstructor
@FieldDefaults(level = AccessLevel.PRIVATE)
@Entity
@Comment("项目")
@Table(name = "bs_project_info")
public class ProjectInfo implements Serializable {

    private static final long serialVersionUID = -2079824786038520972L;

    @Comment("主键")
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    Long id;

    @Comment("项目名称")
    @ApiModelProperty(value ="区域项目名称", example = "2", required = true)
    String projectName;

    @Comment("申报年份")
    @ApiModelProperty(value ="申报年份", example = "2", required = true)
    Integer year;

    @Comment("创建时间")
    @ApiModelProperty(value ="创建时间", example = "2", required = true)
    LocalDateTime createTime;

    

    @Builder.Default
    @Comment("权重")
    @ApiModelProperty(value ="权重", required = true)
    @OneToMany(fetch = FetchType.EAGER, cascade = CascadeType.ALL)
    Set<WeightSort> weightSorts = new HashSet<>();
}

```

关联权重`WeightSort`类（数据库名：bs_weight_sort）：
```java
package com.yiring.finance.domain.weightsort;

import com.yiring.finance.domain.user.User;
import io.swagger.annotations.ApiModelProperty;
import java.io.Serializable;
import javax.persistence.Entity;
import javax.persistence.GeneratedValue;
import javax.persistence.GenerationType;
import javax.persistence.Id;
import javax.persistence.ManyToOne;
import javax.persistence.Table;
import lombok.AccessLevel;
import lombok.AllArgsConstructor;
import lombok.Builder;
import lombok.Data;
import lombok.NoArgsConstructor;
import lombok.experimental.FieldDefaults;
import org.hibernate.annotaions.Comment;

/**
 * 用户关联项目权重表
 *
 * @author zhangshuai
 * @date 2019/5/9 11:09
 */

@Data
@Builder
@NoArgsConstructor
@AllArgsConstructor
@FieldDefaults(level = AccessLevel.PRIVATE)
@Entity
@Comment("权重表")
@Table(name = "bs_weight_sort")
public class WeightSort implements Serializable {

    private static final long serialVersionUID = 1009267440536346667L;

    @Comment("主键")
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @ApiModelProperty(value ="主键", example = "2")
    Long id;

    @Comment("绑定用户")
    @ApiModelProperty(value ="绑定用户")
    @ManyToOne
    User user;

    @Comment("权重")
    @ApiModelProperty(value ="权重", example = "2")
    Long weight;
}

```

关联用户`User`表（数据库表名：SYSTEM_USER）：
```java
package com.yiring.finance.domain.user;

import com.yiring.finance.domain.dictionary.DictionaryType;
import com.yiring.finance.domain.permission.Permission;
import com.yiring.finance.domain.role.Role;
import com.yiring.finance.secruity.JwtUser;
import java.io.Serializable;
import java.time.LocalDateTime;
import java.util.HashSet;
import java.util.Set;
import java.util.stream.Collectors;
import javax.persistence.CascadeType;
import javax.persistence.Column;
import javax.persistence.Entity;
import javax.persistence.FetchType;
import javax.persistence.GeneratedValue;
import javax.persistence.GenerationType;
import javax.persistence.Id;
import javax.persistence.Index;
import javax.persistence.ManyToMany;
import javax.persistence.ManyToOne;
import javax.persistence.OneToOne;
import javax.persistence.Table;
import javax.persistence.Transient;
import lombok.AccessLevel;
import lombok.AllArgsConstructor;
import lombok.Builder;
import lombok.Data;
import lombok.NoArgsConstructor;
import lombok.experimental.FieldDefaults;
import org.hibernate.annotaions.Comment;

/**
 * 用户
 *
 * @author fangzhimin
 * @date 2018/9/3 15:27
 */

@Data
@Builder
@NoArgsConstructor
@AllArgsConstructor
@FieldDefaults(level = AccessLevel.PRIVATE)
@Entity
@Comment("用户")
@Table(name = "SYSTEM_USER", indexes = {
        @Index(name = "IDX_USERNAME", columnList = "username", unique = true),
        @Index(name = "IDX_MOBILE", columnList = "mobile", unique = true),
        @Index(name = "IDX_EMAIL", columnList = "email", unique = true)
})
public class User implements Serializable {

    private static final long serialVersionUID = -5787847701210907511L;

    @Comment("主键")
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    Long id;

    @Comment("真实姓名")
    String realName;

    @Comment("用户名")
    @Column(unique = true)
    String username;

    @Comment("密码")
    String password;

    @Comment("手机号")
    @Column(unique = true)
    String mobile;

    @Comment("最后登录IP地址")
    String lastLoginIp;

    @Comment("激活时间")
    LocalDateTime activationTime;

    @Comment("权限更新时间")
    LocalDateTime authorityUpdateTime;

    @Comment("最后重置密码时间")
    LocalDateTime lastPasswordResetTime;

    @Comment("最后登录时间")
    LocalDateTime lastLoginTime;

    @Comment("最后更新信息时间")
    LocalDateTime lastUpdateTime;

    @Comment("创建时间")
    LocalDateTime createTime;

    /**
     * 验证码（非持久化）
     */
    @Transient
    String code;

    /**
     * token（非持久化）
     */
    @Transient
    String token;

}

```

## 后记：
主要还是卡在了不能创建临时表之后查询。大佬自己查询操作了一下构建CriteriaQuery这个类。但是还是不能解决这个问题。只能先记录一下，等待以后的解决。