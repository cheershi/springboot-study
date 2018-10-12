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
	//判断是否是web环境 
        this.setInitializers(this.getSpringFactoriesInstances(ApplicationContextInitializer.class));
	//加载ApplicationContextInitializer类型的对象
        this.setListeners(this.getSpringFactoriesInstances(ApplicationListener.class));
	//加载ApplicationListener类型的对象
        this.mainApplicationClass = this.deduceMainApplicationClass();
	//寻找启动主类
    }
    
### deduceWebApplicationType（）方法

    	private WebApplicationType deduceWebApplicationType() {
        if (ClassUtils.isPresent("org.springframework.web.reactive.DispatcherHandler", (ClassLoader)null) && !ClassUtils.isPresent("org.springframework.web.servlet.DispatcherServlet", (ClassLoader)null) && !ClassUtils.isPresent("org.glassfish.jersey.server.ResourceConfig", (ClassLoader)null)) {
            return WebApplicationType.REACTIVE;
        } else {
            String[] var1 = WEB_ENVIRONMENT_CLASSES;
            int var2 = var1.length;

            for(int var3 = 0; var3 < var2; ++var3) {
                String className = var1[var3];
                if (!ClassUtils.isPresent(className, (ClassLoader)null)) {
                    return WebApplicationType.NONE;
                }
            }

            return WebApplicationType.SERVLET;
        }
    }
 通过是否存在web所需要的接口来判断是否是web应用，就是看类路径下是能加载到"org.springframework.web.reactive.DispatcherHandler"，"org.springframework.web.servlet.DispatcherServlet"
 
 
 ### getSpringFactoriesInstance（）方法
 
    private <T> Collection<T> getSpringFactoriesInstances(Class<T> type) {
        return this.getSpringFactoriesInstances(type, new Class[0]);
    }

    private <T> Collection<T> getSpringFactoriesInstances(Class<T> type, Class<?>[] parameterTypes, Object... args) {
    	//线程上下文加载器
        ClassLoader classLoader = Thread.currentThread().getContextClassLoader();
	//关键代码
        Set<String> names = new LinkedHashSet(SpringFactoriesLoader.loadFactoryNames(type, classLoader));
	//根据上一步获取到的类，创建实例对象
        List<T> instances = this.createSpringFactoriesInstances(type, parameterTypes, classLoader, args, names);
	//排序
        AnnotationAwareOrderComparator.sort(instances);
        return instances;
    }
    
 在上面的代码中最重要的就是SpringFactoriesLoader.loadFactoryNames(type, classLoader)这一句话。 
loadFactoryNames的第一个参数是要加载的类的类型，第二个参数是类加载器。loadFactoryNames方法的内容如下所示：

    public static List<String> loadFactoryNames(Class<?> factoryClass, @Nullable ClassLoader classLoader) {
    //获取要加载的类的全限定名这里是org.springframework.context.ApplicationContextInitializer
        String factoryClassName = factoryClass.getName();
        return (List)loadSpringFactories(classLoader).getOrDefault(factoryClassName, Collections.emptyList());
    }

    private static Map<String, List<String>> loadSpringFactories(@Nullable ClassLoader classLoader) {
        MultiValueMap<String, String> result = (MultiValueMap)cache.get(classLoader);
        if (result != null) {
            return result;
        } else {
            try {
	    //加载资源 从加载的是哪个资源呢？META-INF/spring.factories这个文件
            //注意这里会加载所有类路径下的/META-INF/spring.factories，在Spring的相关jar包中基本上都有这个文件存在，当然也可以在自己的工程中自定义。
                Enumeration<URL> urls = classLoader != null ? classLoader.getResources("META-INF/spring.factories") : ClassLoader.getSystemResources("META-INF/spring.factories");
                LinkedMultiValueMap result = new LinkedMultiValueMap();

                while(urls.hasMoreElements()) {
                    URL url = (URL)urls.nextElement();
                    UrlResource resource = new UrlResource(url);
		    //获取键为前面获取到的全限定类名的值,多个值用 , 分割
                    Properties properties = PropertiesLoaderUtils.loadProperties(resource);
                    Iterator var6 = properties.entrySet().iterator();

                    while(var6.hasNext()) {
                        Entry<?, ?> entry = (Entry)var6.next();
                        List<String> factoryClassNames = Arrays.asList(StringUtils.commaDelimitedListToStringArray((String)entry.getValue()));
                        result.addAll((String)entry.getKey(), factoryClassNames);
                    }
                }

                cache.put(classLoader, result);
                return result;
            } catch (IOException var9) {
                throw new IllegalArgumentException("Unable to load factories from location [META-INF/spring.factories]", var9);
            }
        }
    }
    
### deduceMainApplicationClass()方法：

     private Class<?> deduceMainApplicationClass() {
        try {
	//获取运行时方法调用栈的信息 
            StackTraceElement[] stackTrace = (new RuntimeException()).getStackTrace();
            StackTraceElement[] var2 = stackTrace;
            int var3 = stackTrace.length;

            for(int var4 = 0; var4 < var3; ++var4) {
                StackTraceElement stackTraceElement = var2[var4];
		//找到方法调用链上方法名为main的类
                if ("main".equals(stackTraceElement.getMethodName())) {
		//返回main方法所在的类对象
                    return Class.forName(stackTraceElement.getClassName());
                }
            }
        } catch (ClassNotFoundException var6) {
            ;
        }

        return null;
    }
 
