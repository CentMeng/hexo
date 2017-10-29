---
title: Swagger2集成
date: 2017-07-13 11:04:00
categories:
- 后端技术
tags:
- Java架构师
---


> Swagger 是一个规范和完整的框架，用于生成、描述、调用和可视化 RESTful 风格的 Web 服务。总体目标是使客户端和文件系统作为服务器以同样的速度来更新。文件的方法，参数和模型紧密集成到服务器端的代码，允许API来始终保持同步。Swagger 让部署管理和使用功能强大的API从未如此简单。

- Swagger2生成的api页面

<img src = "/img/java/swagger/picture/swaggerui.png">

## Swagger集成（Spring boot）

- 添加Swagger2依赖

> 在pom.xml中加入Swagger2的依赖


```
 <!-- swagger2  -->
        <dependency>
            <groupId>io.springfox</groupId>
            <artifactId>springfox-swagger2</artifactId>
            <version>2.2.2</version>
        </dependency>
        <dependency>
            <groupId>io.springfox</groupId>
            <artifactId>springfox-swagger-ui</artifactId>
            <version>2.2.2</version>
        </dependency>
        <!-- swagger2 -->
```

- 创建Swagger2配置类

> 在Application.java同级创建Swagger2的配置类Swagger2

```
/**
 * @author Vincent.M
 * @mail mengshaojie@188.com
 * @date 17/7/12
 * @copyright ©2017 孟少杰 All Rights Reserved
 * @desc @Configuration注解，让Spring来加载该类配置。再通过@EnableSwagger2注解来启用Swagger2。
 * 再通过createRestApi函数创建Docket的Bean之后，apiInfo()用来创建该Api的基本信息（这些基本信息会展现在文档页面中）。select()函数返回一个ApiSelectorBuilder实例用来控制哪些接口暴露给Swagger来展现，本例采用指定扫描的包路径来定义，Swagger会扫描该包下所有Controller定义的API，并产生文档内容（除了被@ApiIgnore指定的请求）。
 */
@Configuration
@EnableSwagger2
public class Swagger2 {

    @Bean
    public Docket createRestApi() {
        return new Docket(DocumentationType.SWAGGER_2)
                .apiInfo(apiInfo())
                .select()
                .apis(RequestHandlerSelectors.basePackage("com.msj.fuxi"))
                .paths(PathSelectors.any())
                .build();
    }
    private ApiInfo apiInfo() {
        return new ApiInfoBuilder()
                .title("Spring Boot中使用Swagger2构建RESTful APIs")
                .description("更多Spring Boot相关文章请关注：http://blog.didispace.com/")
                .termsOfServiceUrl("http://centmeng.github.io")
                .contact("孟少杰")
                .version("1.0")
                .build();
    }
}

```

- Controller中添加注解

```

    @ApiOperation(value="菜单列表页面", notes="返回子菜单列表")
    @RequestMapping(value = "menu", method = RequestMethod.GET)
    @PrivilegeRequired(value = {"admin","menu_detail"})
    public ModelAndView listMenu() {
        ModelAndView result = new ModelAndView();
        List<Menu> subMenus = menuService.findBySubMenu();
        result.addObject("subMenus",subMenus);
        result.setViewName("datadic/menu/list");
        return result;
    }

    @ApiOperation(value="修改菜单列表", notes="返回菜单信息，和父菜单信息")
    @ApiImplicitParam(name = "id", value = "菜单id", required = true, dataType = "String")
    @RequestMapping(value = "menu/edit/{id}", method = RequestMethod.GET)
    @PrivilegeRequired(value = {"admin","menu_modify"})
    public ModelAndView editZichanTypeDic(@PathVariable String id) {
        ModelAndView result = new ModelAndView();
        List<Menu> subMenus = menuService.findBySubMenu();
        result.addObject("subMenus",subMenus);
        Menu menu = menuService.getById(id);
        if (menu == null) {
            return new ModelAndView("datadic/menu/list");
        }
        List<Menu> menus = menuService.findByParent("");
        result.setViewName("datadic/menu/edit");
        result.addObject("menu", menu);
        result.addObject("create", false);
        result.addObject("parentMenu", menus);
        return result;
    }

    @RequestMapping(value = "menu/create", method = RequestMethod.GET)
    @PrivilegeRequired(value = {"admin","menu_create"})
    public ModelAndView createZichanTypeDic() {
        ModelAndView result = new ModelAndView();
        List<Menu> subMenus = menuService.findBySubMenu();
        result.addObject("subMenus",subMenus);
        result.setViewName("datadic/menu/edit");
        Menu menu = new Menu();
        List<Menu> menus = menuService.findByParent("");
        result.addObject("menu", menu);
        result.addObject("create", true);
        result.addObject("parentMenu", menus);
        return result;
    }


    /**
     * @param pageSize
     * @param startIndex
     * @param sEcho
     * @return
     */
    @ApiOperation(value="菜单列表", notes="分页处理")
    @ApiImplicitParams({
            @ApiImplicitParam(name = "iDisplayLength", value = "每页条数", required = true, dataType = "int"),
            @ApiImplicitParam(name = "iDisplayStart", value = "起始数", required = true, dataType = "int"),
            @ApiImplicitParam(name = "sEcho", value = "", required = true, dataType = "int")
    })
    @RequestMapping(value = "menu/getData", method = RequestMethod.GET)
    @ResponseBody
    @PrivilegeRequired(value = {"admin","menu_detail"})
    public TableData<Menu> getMenuData(@RequestParam("iDisplayLength") int pageSize,
                                             @RequestParam("iDisplayStart") int startIndex,
                                             @RequestParam("sEcho") int sEcho) {
        Sort sort = new Sort(Sort.Direction.ASC, "orderId");
        Pageable pageable = new PageRequest(startIndex / pageSize, pageSize, sort);
        PagedResult<Menu> pageableResult = menuService.list(pageable);
        return new TableData<>(pageableResult, sEcho + 1);
    }
```

完成上述代码添加上，启动Spring Boot程序，访问：http://localhost:8080/swagger-ui.html。就能看到前文所展示的RESTful API的页面。我们也可以在此页面对接口进行测试。

<img src = "/img/java/swagger/picture/swagger测试接口.png">