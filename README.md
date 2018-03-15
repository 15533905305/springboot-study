# springboot-study

#### 第一节 测试action方法  
 1、  
  @RunWith(SpringJUnit4ClassRunner.class)
@SpringApplicationConfiguration(classes = MockServletContext.class)
@WebAppConfiguration
public class Chapter1ApplicationTests {

	private MockMvc mvc;

	@Before
	public void setUp() throws Exception {
		mvc = MockMvcBuilders.standaloneSetup(new HelloController()).build();
	}

	@Test
	public void getHello() throws Exception {
		mvc.perform(MockMvcRequestBuilders.get("/hello").accept(MediaType.APPLICATION_JSON)).andExpect(status().isOk())
				.andExpect(content().string(equalTo("Hello World")));
	}

}  
 
Controller 控制层编写

@RestController
public class HelloController {

    @RequestMapping("/hello")
    public String index() {
        return "Hello World";
    }

}


#### 第二节 spring2.0新特性增加监听器
	
在Spring Boot 2.0中对事件模型做了一些增强，主要就是增加了ApplicationStartedEvent事件，所以在2.0版本中所有的事件按执行的先后顺序如下：

ApplicationStartingEvent
ApplicationEnvironmentPreparedEvent
ApplicationPreparedEvent
ApplicationStartedEvent <= 新增的事件
ApplicationReadyEvent
ApplicationFailedEvent
从上面的列表中，我们可以看到ApplicationStartedEvent位于ApplicationPreparedEvent之后，ApplicationReadyEvent之前。


#### 第三节  自定义属性值与加载 和激活配置文件   

com.didispace.blog.name=程序猿DD
com.didispace.blog.title=Spring Boot教程
com.didispace.blog.desc=${com.didispace.blog.name}正在努力写《${com.didispace.blog.title}》

# 随机字符串
com.didispace.blog.value=${random.value}
# 随机int
com.didispace.blog.number=${random.int}
# 随机long
com.didispace.blog.bignumber=${random.long}
# 10以内的随机数
com.didispace.blog.test1=${random.int(10)}
# 10-20的随机数
com.didispace.blog.test2=${random.int[10,20]}

# 多环境配置文件激活属性
spring.profiles.active=dev


 @Value("${com.didispace.blog.name}")
    private String name;
    @Value("${com.didispace.blog.title}")
    private String title;
    @Value("${com.didispace.blog.desc}")
    private String desc;

    @Value("${com.didispace.blog.value}")
    private String value;
    @Value("${com.didispace.blog.number}")
    private Integer number;
    @Value("${com.didispace.blog.bignumber}")
    private Long bignumber;
    @Value("${com.didispace.blog.test1}")
    private Integer test1;
    @Value("${com.didispace.blog.test2}")
    private Integer test2;
    
    ==============================================================================
    
    com.didispace.foo=bar
    com.didispace.database-platform=sql

    com.didispace.post[0]=Why Spring Boot
    com.didispace.post[1]=Why Spring Cloud
    com.didispace.posts[0].title=Why Spring Boot
    com.didispace.posts[0].content=It is perfect!
    com.didispace.posts[1].title=Why Spring Cloud
    com.didispace.posts[1].content=It is perfect too!


@SpringBootApplication
public class Application {

	public static void main(String[] args) {
		ApplicationContext context = SpringApplication.run(Application.class, args);

		Binder binder = Binder.get(context.getEnvironment());

		// 绑定简单配置
		FooProperties foo = binder.bind("com.didispace", Bindable.of(FooProperties.class)).get();
		System.out.println(foo.getFoo());

		// 绑定List配置
		List<String> post = binder.bind("com.didispace.post", Bindable.listOf(String.class)).get();
		System.out.println(post);

		List<PostInfo> posts = binder.bind("com.didispace.posts", Bindable.listOf(PostInfo.class)).get();
		System.out.println(posts);

		// 读取配置
		System.out.println(context.getEnvironment().containsProperty("com.didispace.database-platform"));
		System.out.println(context.getEnvironment().containsProperty("com.didispace.databasePlatform"));

	}

}

@Data
@ConfigurationProperties(prefix = "com.didispace")
public class FooProperties {

    private String foo;

    private String databasePlatform;

}


@Data
@ConfigurationProperties
public class PostInfo {

    private String title;
    private String content;

}
    
#### 第四节 渲染web视图 （Thymeleaf,Freemarker,Velocity）
	
它们默认的模板配置路径为：src/main/resources/templates


		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-thymeleaf</artifactId>
		</dependency>
		
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-freemarker</artifactId>
		</dependency>
		
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-velocity</artifactId>
		</dependency>	

@Controller
public class HelloController {
    
    @ResponseBody
    @RequestMapping("/hello")
    public String hello() {
        return "Hello World";
    }

    @RequestMapping("/")
    public String index(ModelMap map) {
        map.addAttribute("host", "http://blog.didispace.com");
        return "aaa";
    }

}

<!DOCTYPE html>
<html xmlns:th="http://www.w3.org/1999/xhtml">
<head lang="en">
    <meta charset="UTF-8" />
    <title>aaaa</title>
</head>
<body>
<h1 th:text="${host}">Thymeleaf</h1>
	
<h1>${host}</h1> FreeMarker

<h1>${host}</h1> Velocity模板
</body>
</html>




静态资源访问

在我们开发Web应用的时候，需要引用大量的js、css、图片等静态资源。

默认配置

Spring Boot默认提供静态资源目录位置需置于classpath下，目录名需符合如下规则：

/static
/public
/resources
/META-INF/resources
举例：我们可以在src/main/resources/目录下创建static，在该位置放置一个图片文件。启动程序后，尝试访问http://localhost:8080/D.jpg。如能显示图片，配置成功。

渲染Web页面

在之前的示例中，我们都是通过@RestController来处理请求，所以返回的内容为json对象。那么如果需要渲染html页面的时候，要如何实现呢？

模板引擎

在动态HTML实现上Spring Boot依然可以完美胜任，并且提供了多种模板引擎的默认配置支持，所以在推荐的模板引擎下，我们可以很快的上手开发动态网站。

Spring Boot提供了默认配置的模板引擎主要有以下几种：

Thymeleaf
FreeMarker
Velocity
Groovy
Mustache
Spring Boot建议使用这些模板引擎，避免使用JSP，若一定要使用JSP将无法实现Spring Boot的多种特性，具体可见后文：支持JSP的配置

当你使用上述模板引擎中的任何一个，它们默认的模板配置路径为：src/main/resources/templates。当然也可以修改这个路径，具体如何修改，可在后续各模板引擎的配置属性中查询并修改。

Thymeleaf

Thymeleaf是一个XML/XHTML/HTML5模板引擎，可用于Web与非Web环境中的应用开发。它是一个开源的Java库，基于Apache License 2.0许可，由Daniel Fernández创建，该作者还是Java加密库Jasypt的作者。

Thymeleaf提供了一个用于整合Spring MVC的可选模块，在应用开发中，你可以使用Thymeleaf来完全代替JSP或其他模板引擎，如Velocity、FreeMarker等。Thymeleaf的主要目标在于提供一种可被浏览器正确显示的、格式良好的模板创建方式，因此也可以用作静态建模。你可以使用它创建经过验证的XML与HTML模板。相对于编写逻辑或代码，开发者只需将标签属性添加到模板中即可。接下来，这些标签属性就会在DOM（文档对象模型）上执行预先制定好的逻辑。

示例模板：

<table>
  <thead>
    <tr>
      <th th:text="#{msgs.headers.name}">Name</td>
      <th th:text="#{msgs.headers.price}">Price</td>
    </tr>
  </thead>
  <tbody>
    <tr th:each="prod : ${allProducts}">
      <td th:text="${prod.name}">Oranges</td>
      <td th:text="${#numbers.formatDecimal(prod.price,1,2)}">0.99</td>
    </tr>
  </tbody>
</table>
可以看到Thymeleaf主要以属性的方式加入到html标签中，浏览器在解析html时，当检查到没有的属性时候会忽略，所以Thymeleaf的模板可以通过浏览器直接打开展现，这样非常有利于前后端的分离。

在Spring Boot中使用Thymeleaf，只需要引入下面依赖，并在默认的模板路径src/main/resources/templates下编写模板文件即可完成。

<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-thymeleaf</artifactId>
</dependency>
在完成配置之后，举一个简单的例子，在快速入门工程的基础上，举一个简单的示例来通过Thymeleaf渲染一个页面。

@Controller
public class HelloController {

    @RequestMapping("/")
    public String index(ModelMap map) {
        // 加入一个属性，用来在模板中读取
        map.addAttribute("host", "http://blog.didispace.com");
        // return模板文件的名称，对应src/main/resources/templates/index.html
        return "index";  
    }

}
<!DOCTYPE html>
<html>
<head lang="en">
    <meta charset="UTF-8" />
    <title></title>
</head>
<body>
<h1 th:text="${host}">Hello World</h1>
</body>
</html>
如上页面，直接打开html页面展现Hello World，但是启动程序后，访问http://localhost:8080/，则是展示Controller中host的值：http://blog.didispace.com，做到了不破坏HTML自身内容的数据逻辑分离。


#### 第五节Restful接口
 
 @RestController
@RequestMapping(value="/users")     // 通过这里配置使下面的映射都在/users下，可去除
public class UserController {

    static Map<Long, User> users = Collections.synchronizedMap(new HashMap<Long, User>());

    @RequestMapping(value="/", method=RequestMethod.GET)
    public List<User> getUserList() {
        // 处理"/users/"的GET请求，用来获取用户列表
        // 还可以通过@RequestParam从页面中传递参数来进行查询条件或者翻页信息的传递
        List<User> r = new ArrayList<User>(users.values());
        return r;
    }

    @RequestMapping(value="/", method=RequestMethod.POST)
    public String postUser(@ModelAttribute User user) {
        // 处理"/users/"的POST请求，用来创建User
        // 除了@ModelAttribute绑定参数之外，还可以通过@RequestParam从页面中传递参数
        users.put(user.getId(), user);
        return "success";
    }

    @RequestMapping(value="/{id}", method=RequestMethod.GET)
    public User getUser(@PathVariable Long id) {
        // 处理"/users/{id}"的GET请求，用来获取url中id值的User信息
        // url中的id可通过@PathVariable绑定到函数的参数中
        return users.get(id);
    }

    @RequestMapping(value="/{id}", method=RequestMethod.PUT)
    public String putUser(@PathVariable Long id, @ModelAttribute User user) {
        // 处理"/users/{id}"的PUT请求，用来更新User信息
        User u = users.get(id);
        u.setName(user.getName());
        u.setAge(user.getAge());
        users.put(id, u);
        return "success";
    }

    @RequestMapping(value="/{id}", method=RequestMethod.DELETE)
    public String deleteUser(@PathVariable Long id) {
        // 处理"/users/{id}"的DELETE请求，用来删除User
        users.remove(id);
        return "success";
    }

}

@RunWith(SpringJUnit4ClassRunner.class)
@SpringApplicationConfiguration(classes = MockServletContext.class)
@WebAppConfiguration
public class ApplicationTests {

	private MockMvc mvc;

	@Before
	public void setUp() throws Exception {
		mvc = MockMvcBuilders.standaloneSetup(
				new HelloController(),
				new UserController()).build();
	}

	@Test
	public void getHello() throws Exception {
		mvc.perform(MockMvcRequestBuilders.get("/hello").accept(MediaType.APPLICATION_JSON))
				.andExpect(status().isOk())
				.andExpect(content().string(equalTo("Hello World")));
	}

	@Test
	public void testUserController() throws Exception {
//  	测试UserController
		RequestBuilder request = null;

		// 1、get查一下user列表，应该为空
		request = get("/users/");
		mvc.perform(request)
				.andExpect(status().isOk())
				.andExpect(content().string(equalTo("[]")));

		// 2、post提交一个user
		request = post("/users/")
				.param("id", "1")
				.param("name", "测试大师")
				.param("age", "20");
		mvc.perform(request)
//				.andDo(MockMvcResultHandlers.print())
				.andExpect(content().string(equalTo("success")));

		// 3、get获取user列表，应该有刚才插入的数据
		request = get("/users/");
		mvc.perform(request)
				.andExpect(status().isOk())
				.andExpect(content().string(equalTo("[{\"id\":1,\"name\":\"测试大师\",\"age\":20}]")));

		// 4、put修改id为1的user
		request = put("/users/1")
				.param("name", "测试终极大师")
				.param("age", "30");
		mvc.perform(request)
				.andExpect(content().string(equalTo("success")));

		// 5、get一个id为1的user
		request = get("/users/1");
		mvc.perform(request)
				.andExpect(content().string(equalTo("{\"id\":1,\"name\":\"测试终极大师\",\"age\":30}")));

		// 6、del删除id为1的user
		request = delete("/users/1");
		mvc.perform(request)
				.andExpect(content().string(equalTo("success")));

		// 7、get查一下user列表，应该为空
		request = get("/users/");
		mvc.perform(request)
				.andExpect(status().isOk())
				.andExpect(content().string(equalTo("[]")));

	}

}
