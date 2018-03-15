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
    

	
