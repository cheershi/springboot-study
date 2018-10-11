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
