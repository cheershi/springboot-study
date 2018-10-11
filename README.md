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
