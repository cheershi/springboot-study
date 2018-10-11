# springboot-demo
# springboot启动流程（一）
启动主类代码如下：

    @SpringBootApplication
    @MapperScan(basePackages = "cn.tnar.cloud.model.mapper")
    public class CloudParkApplication {

	public static void main(String[] args) {

		SpringApplication.run(CloudParkApplication.class, args);
	  }
    }

@SpringBootApplication这个注解

	@Target({ElementType.TYPE})
	@Retention(RetentionPolicy.RUNTIME)
	@Documented
	@Inherited
	@SpringBootConfiguration
	@EnableAutoConfiguration
	@ComponentScan(
    excludeFilters = {@Filter(
    type = FilterType.CUSTOM,
    classes = {TypeExcludeFilter.class}
	), @Filter(
    type = FilterType.CUSTOM,
    classes = {AutoConfigurationExcludeFilter.class}
	)}
	)
	public @interface SpringBootApplication {
	
	}
 @Inherited的意思是注解在这个类上的子类可以继承这个注解
 
 @EnableAutoConfiguration：开启自动配置
 
 @ComponentScan：开启包扫描
 
 @SpringBootConfiguration：配置类（相当与beans）
 
 ## 首先进入到SpringApplication的run方法中看一下这个方法的内容
      public static ConfigurableApplicationContext run(Class<?> primarySource, String... args) {
        return run(new Class[]{primarySource}, args);
    }

    public static ConfigurableApplicationContext run(Class<?>[] primarySources, String[] args) {
        return (new SpringApplication(primarySources)).run(args);
    }
   
  在调用run方法启动SpringBoot容器的时候还有一点需要注意的是，调用run方法的时候会返回一个Spring上下文 ConfigurableApplicationContext的实例。
  
 ## SpringApplication的初始化
     public SpringApplication(Class... primarySources) {
        this((ResourceLoader)null, primarySources);
    }

    public SpringApplication(ResourceLoader resourceLoader, Class... primarySources) {
        this.sources = new LinkedHashSet();
        this.bannerMode = Mode.CONSOLE;
        this.logStartupInfo = true;
        this.addCommandLineProperties = true;
        this.headless = true;
        this.registerShutdownHook = true;
        this.additionalProfiles = new HashSet();
        this.isCustomEnvironment = false;
        this.resourceLoader = resourceLoader;
        Assert.notNull(primarySources, "PrimarySources must not be null");
        this.primarySources = new LinkedHashSet(Arrays.asList(primarySources));
	//如果传入的sources有值的话，将Object[]对象转换为List。
        this.webApplicationType = this.deduceWebApplicationType();
        this.setInitializers(this.getSpringFactoriesInstances(ApplicationContextInitializer.class));
        this.setListeners(this.getSpringFactoriesInstances(ApplicationListener.class));
        this.mainApplicationClass = this.deduceMainApplicationClass();
    }
    
