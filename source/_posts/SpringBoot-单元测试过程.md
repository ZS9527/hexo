---
title: SpringBoot-单元测试过程
date: 2020-01-04 16:52:13
tags: springboot
---

# SpringBoot-单元测试过程
> 记录一下在项目中使用过的spring-boot-starter-test

<!--more-->

## 引入依赖
```
<dependency>
         <groupId>org.springframework.boot</groupId>
         <artifactId>spring-boot-starter-test</artifactId>
         <scope>test</scope>
</dependency>

```
包内自带了junit和mockito，如果自己引入过的话，最好使用包内自带的。

## 基本注释
说明一些我用到的注释和方法。
### 类注释和抽象类
- **@TestExecutionListeners：**是一个类级别的注释，用于指定在测试类执行之前，可以做的一些动作。
```
@TestExecutionListeners({TransactionalTestExecutionListener.class, SqlScriptsTestExecutionListener.class})
```
- **@ActiveProfiles：**是一个类级别的注释，用于声明在加载 for测试类时应使用哪些活动bean定义配置文件ApplicationContext。就是指定你这个测试类所用的配置文件。
```
@ActiveProfiles("test")
```
- **@SpringBootTest：**是一个类级别的注释，指定启动类和web环境。
```
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
```
- **AbstractTestNGSpringContextTests：**抽象基础测试类，该类将Spring TestContext Framework 与TestNG 环境中的显式ApplicationContext测试支持集成在一起。

### 方法注释和实现方法
- **@BeforeMethod：**在测试方法前运行的方法。
- **@AfterMethod：**在测试方法后运行的方法。
- **@Test：**测试方法注释。
	1. **groups：**给方法分组执行。
	2. **dependsOnGroups：**说明这个方法依赖的组列表。这些组的方法都已经在此方法调用前被调用。如果被依赖的方法未执行成功，这个测试方法会被跳过。
	3. **dependsOnMethods：**和（2）类似，是说明依赖的方法列表。
	4. **timeOut：**方法应花费的最大毫秒数。
```
@Test(groups = {"home"}, dependsOnGroups = {"login"}, dependsOnMethods = {}, timeOut = 300)
```

- **@Ignore：**这个可以放在类上，将忽略类和其子类的测试。我使用时放在方法上，仅仅只是需要忽略方法。

## 使用例子
在这里放一个我使用时的例子。
```
@TestExecutionListeners({TransactionalTestExecutionListener.class, SqlScriptsTestExecutionListener.class})
@ActiveProfiles("test")
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
public class AuthControllerTest extends AbstractTestNGSpringContextTests {//可以使用AbstractTestNGSpringContextTests来取消事务

    @Autowired
    WebApplicationContext context;

    private ManualRestDocumentation restDocumentation = new ManualRestDocumentation();
    private MockMvc mockMvc;
    private String token = TestsConstant.DEFAULT_TOKEN;

    @BeforeMethod
    public void setUp(Method method) {
        mockMvc = ApplicationTests.initMockMvc(context, restDocumentation);
        this.restDocumentation.beforeTest(getClass(), method.getName());
    }

    @AfterMethod
    public void tearDown() {
        this.restDocumentation.afterTest();
    }

    @Transactional
    //@Test(priority = -1)//权重（默认为0，此处需要先登录才能进行下面的操作，所以需要设置为-1，使login先进行测试，再进行下面测试）
    @Test(groups = {"login"})
    public void login() throws Exception {
        getVerificationCode();
        String url = "/auth/login";

        // 登录本身不需要Authorization, 为了统一处理{ Header - Authorization }, 登录这里先加上请求头, 在文档输出时再过滤掉
        RequestBuilder builder = post(url)//把所有元素拼接起来组成一个访问url
                .header(JwtUtils.TOKEN_HEADER, JwtUtils.getTokenHeaderValue(this.token))//添加头
                .param("username", TestsConstant.USERNAME)
                .param("password", TestsConstant.PASSWORD)//参数名称和值
                .param("code",TestsConstant.CODE)
                .accept(MediaType.APPLICATION_JSON);

        String result = mockMvc.perform(builder)
                .andExpect(status().isOk())
                .andExpect(jsonPath("$.status").value(HttpStatus.OK.value()))//状态
                .andExpect(jsonPath("$.body").exists())//body详细内容
                .andDo(document(TestsConstant.DOCUMENT_IDENTIFIER, preprocessRequest(//打印配置
                        removeHeaders(JwtUtils.TOKEN_HEADER)
                ), requestParameters(
                        parameterWithName("username").description("用户名（手机号）"),
                        parameterWithName("password").description("密码"),
                        parameterWithName("code").description("验证码"),
                        beneathPath("body"),//解析文件时只解析body内容
                        fieldWithPath("roles").description("角色")
                )))
                .andDo(print())//打印出测试结果（详细参数以及结果等等）
                .andReturn()
                .getResponse()
                .getContentAsString();

        this.token = JSONObject.parseObject(result).getJSONObject("body").getString("token");

    }
}
```
