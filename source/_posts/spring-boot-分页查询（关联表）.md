---
title: spring-boot 分页查询（关联表）
date: 2016-08-08 22:27:42
categories:
- spring-boot
tags:
- java开发
---
<img src="/img/java.jpg" />

引言：由于公司转型，使我原本android开发工程师，转变为后台开发工程师，对于后台，除了大学利用servlet写过些项目，其他就一无所知。公司使用[spring-boot](http://projects.spring.io/spring-boot/)框架.那么spring-boot框架究竟是什么呢？Spring-boot是微框架，是由Pivotal团队提供的全新框架，其设计目的是用来简化新Spring应用的初始搭建以及开发过程。该框架使用了特定的方式来进行配置，从而使开发人员不再需要定义样板化的配置。好了废话不说了，下面讲讲分页实例吧。

### 首先我们创建两个实例（表）
TestPage 和TestPageRef，其中TestPageRef关联TestPage的Id,其中这两个表都继承TimeScopeEntity（增加了id和创建时间字段）

```
/**
 * 分页测试表
 * @author  CentMeng <mengshaojie@188.com>
 * @date    Aug 8, 2016 3:30:45 PM
 * @copyright ©2016 孟少杰 All Rights Reserved
 * @desc  
 */
@Data
@NoArgsConstructor
@AllArgsConstructor
@Entity
@Table(name = "T_TESTPAGE")
public class TestPage extends TimeScopeEntity{
    
    private String name;
    
    private String content;
    
}

```

```
/**
 * 分页测试表关联表
 * @author  CentMeng <mengshaojie@188.com>
 * @date    Aug 8, 2016 3:30:45 PM
 * @copyright ©2016 孟少杰 All Rights Reserved
 * @desc  
 */
@Data
@NoArgsConstructor
@AllArgsConstructor
@Entity
@Table(name = "T_TESTPAGEREF")
public class TestPageRef extends TimeScopeEntity{
    
    private String testId;
    
    private boolean enabled;

}

```

### 写Dao层接口，主要是继承PagingAndSortingRepository

这里我们写一个获取list的方法接口,返回的是Page对象

也可以返回某些字段，将t，替换成t.id,t.content等等

```
import javax.transaction.Transactional;
import net.luoteng.muser.entity.TestPage;
import org.springframework.data.domain.Page;
import org.springframework.data.domain.Pageable;
import org.springframework.data.jpa.repository.Query;
import org.springframework.data.repository.PagingAndSortingRepository;
import org.springframework.data.repository.query.Param;

/**
 * 分页测试
 * @author  CentMeng <mengshaojie@188.com>
 * @date    Aug 8, 2016 3:39:55 PM
 * @copyright ©2016 孟少杰 All Rights Reserved
 * @desc  
 */
@Transactional
public interface TestRepository extends PagingAndSortingRepository<TestPage,String>{
    @Query("select t from TestPage t,TestPageRef r where t.id = r.testId and r.enabled = :enabled")
    public Page<TestPage> findList( @Param("enabled") boolean enabled,Pageable pageable);

}

```

### 写Service层

Service层要分接口层和实现层，springboot的注入为接口注入

#### 接口
```
package net.luoteng.muser.service;

import net.luoteng.muser.common.PageResult;
import net.luoteng.muser.entity.TestPage;
import org.springframework.data.domain.Pageable;

/**
 *
 * 分页测试
 * @author  CentMeng <mengshaojie@188.com>
 * @date    Aug 8, 2016 3:37:51 PM
 * @copyright ©2016 孟少杰 All Rights Reserved
 * @desc  
 */
public interface TestService {
    
      public PageResult<TestPage> list(boolean enabled,Pageable pageable);
}
```

#### 实现

```

import java.util.List;
import javax.transaction.Transactional;
import net.luoteng.muser.common.PageResult;
import net.luoteng.muser.dao.TestRepository;
import net.luoteng.muser.entity.TestPage;
import net.luoteng.muser.service.TestService;
import org.apache.commons.collections.CollectionUtils;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.data.domain.Page;
import org.springframework.data.domain.Pageable;
import org.springframework.stereotype.Component;

/**
 *
 * @author  CentMeng <mengshaojie@188.com>
 * @date    Aug 8, 2016 4:21:25 PM
 * @copyright ©2016 孟少杰 All Rights Reserved
 * @desc  
 */
@Component
@Transactional
public class TestImpl implements TestService{

    @Autowired
    TestRepository testRepository;
    
    @Override
    public PageResult<TestPage> list(boolean enabled, Pageable pageable) {
      Page<TestPage> testPages = testRepository.findList(enabled,pageable);
        List<TestPage> tests = testPages.getContent();

        if (!CollectionUtils.isEmpty(tests)) {
            PageResult pageResult = new PageResult(tests, testPages.getTotalElements());
            return pageResult;
        } else {
            return null;
        }
    }

}

```

Page返回参数说明：

```
{
  "content":[
    {"id":123,"title":"blog122","content":"this is blog content"},
    {"id":122,"title":"blog121","content":"this is blog content"},
    {"id":121,"title":"blog120","content":"this is blog content"},
    {"id":120,"title":"blog119","content":"this is blog content"},
    {"id":119,"title":"blog118","content":"this is blog content"},
    {"id":118,"title":"blog117","content":"this is blog content"},
    {"id":117,"title":"blog116","content":"this is blog content"},
    {"id":116,"title":"blog115","content":"this is blog content"},
    {"id":115,"title":"blog114","content":"this is blog content"},
    {"id":114,"title":"blog113","content":"this is blog content"},
    {"id":113,"title":"blog112","content":"this is blog content"},
    {"id":112,"title":"blog111","content":"this is blog content"},
    {"id":111,"title":"blog110","content":"this is blog content"},
    {"id":110,"title":"blog109","content":"this is blog content"},
    {"id":109,"title":"blog108","content":"this is blog content"}],
  "last":false,
  "totalPages":9,
  "totalElements":123,
  "size":15,
  "number":0,
  "first":true,
  "sort":[{
    "timeCreated":"DESC",
  }],
  "numberOfElements":15
}
```

以id倒序排列的10条数据

当前页不是最后一页，后面还有数据

总共有9页

每页大小为15

当前页为第0页

当前页是第一页

当前页是以id倒序排列的

当前页一共有15条数据

### Controller层
关键代码

```
	@Autowired
    TestService testService;

   /**
     *
     * @param page
     * @param pageSize
     * @return
     */
    @RequestMapping(value = "/test/page", method = RequestMethod.GET)
    public RestResponse testPage(
            @RequestParam(value = "page", defaultValue = "0") Integer page,
            @RequestParam(value = "pageSize", defaultValue = "10") Integer pageSize) {
        RestResponse result = new RestResponse();
		//排序对象
        Sort sort = new Sort(Sort.Direction.DESC, "timeCreated");
        Pageable pageable = new PageRequest(page, pageSize, sort);

        PageResult<TestPage> pageResult = testService.list(true, pageable);

        return result.success(pageResult);
    }

```

OK这样，分页关联表查询效果就实现了

### 注解解释
只是个人查看记忆用，非本次讲解内容

@Data 自动创建getter和setter
@NoArgsConstructor 
@AllArgsConstructor 这两个是声明构造函数
@Entity 申明是实体，需要创建表的都会用到
@Table(name = "T_TESTPAGE") 创建表，可以设置表名，以及唯一键
@Component 在service实现层使用
@Transactional 事务 在service实现层和dao层都需要使用
@Autowired 自动引入，自己构造的则不用使用此注解
@RequestMapping(value = "/test/page", method = RequestMethod.GET) 配置请求路径和请求方式