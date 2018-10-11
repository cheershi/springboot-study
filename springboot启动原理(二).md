# springboot启动原理(二)

### run（）方法

    public ConfigurableApplicationContext run(String... args) {
        //启动应用的检测
        StopWatch stopWatch = new StopWatch();
        stopWatch.start();
        //SpringBoot的上下文
        ConfigurableApplicationContext context = null;
        //失败分析报告
        Collection<SpringBootExceptionReporter> exceptionReporters = new ArrayList();
        this.configureHeadlessProperty();
        //SpringBoot的runlistener
        SpringApplicationRunListeners listeners = this.getRunListeners(args);
        listeners.starting();

        Collection exceptionReporters;
        try {
            //参数解析
            ApplicationArguments applicationArguments = new DefaultApplicationArguments(args);
            //配置环境变量
            ConfigurableEnvironment environment = this.prepareEnvironment(listeners, applicationArguments);
            this.configureIgnoreBeanInfo(environment);
            //输出Banner信息 
            Banner printedBanner = this.printBanner(environment);
            //创建应用上下文
            context = this.createApplicationContext();
            exceptionReporters = this.getSpringFactoriesInstances(SpringBootExceptionReporter.class, new Class[]{ConfigurableApplicationContext.class}, context);
            //refresh上下文之前的准备
            this.prepareContext(context, environment, listeners, applicationArguments, printedBanner);
            this.refreshContext(context);
            this.afterRefresh(context, applicationArguments);
            stopWatch.stop();
            if (this.logStartupInfo) {
                (new StartupInfoLogger(this.mainApplicationClass)).logStarted(this.getApplicationLog(), stopWatch);
            }

            listeners.started(context);
            this.callRunners(context, applicationArguments);
        } catch (Throwable var10) {
            this.handleRunFailure(context, var10, exceptionReporters, listeners);
            throw new IllegalStateException(var10);
        }

        try {
            listeners.running(context);
            return context;
        } catch (Throwable var9) {
            this.handleRunFailure(context, var9, exceptionReporters, (SpringApplicationRunListeners)null);
            throw new IllegalStateException(var9);
        }
    }

StopWatch主要是监控启动过程，统计启动时间，检测应用是否已经启动或者停止。

对于getSpringFactoriesInstances这个方法你应该不陌生来吧。这里也是从META-INF/spring.factories中获取类型为org.springframework.boot.SpringApplicationRunListener的配置值，这个默认的配置值为：org.springframework.boot.context.event.EventPublishingRunListener。我们进入到EventPublishingRunListener这个类看一下它的构造函数

    public EventPublishingRunListener(SpringApplication application, String[] args) {
        this.application = application;
        this.args = args;
        //创建一个SimpleApplicationEventMulticaster
        this.initialMulticaster = new SimpleApplicationEventMulticaster();
        Iterator var3 = application.getListeners().iterator();
         //把之前在SpringApplication中获取到的listener循环放入到SimpleApplicationEventMulticaster中
        while(var3.hasNext()) {
            ApplicationListener<?> listener = (ApplicationListener)var3.next();
            this.initialMulticaster.addApplicationListener(listener);
        }

    }

通过上面的分析，我们可以看到EventPublishingRunListener把SpringApplication中的监听器，都放到了SimpleApplicationEventMulticaster中，进行了统一的管理。listeners.starting();

DefaultApplicationArguments的构造函数的内容

    public DefaultApplicationArguments(String[] args) {
        //首先判断不能为null，
        Assert.notNull(args, "Args must not be null");
        //调用Source对应用参数进行解析
        this.source = new DefaultApplicationArguments.Source(args);
        this.args = args;
    }
    
    Source(String[] args) {
    //调用父类的构造函数 Source的继承关系
        super(args);
    }
    public SimpleCommandLinePropertySource(String... args) {
    //对应参数进行解析的工作
        super(new SimpleCommandLineArgsParser().parse(args));
    }

在配置应用参数的时候，是这样这样配置的 - -key=value，为什么要以- -开头呢？在SimpleCommandLineArgsParser的parse方法中你会找到答案

prepareEnvironment（）方法：

    private ConfigurableEnvironment prepareEnvironment(SpringApplicationRunListeners listeners, ApplicationArguments applicationArguments) {
        //获取环境变量 
        ConfigurableEnvironment environment = this.getOrCreateEnvironment();
        //将应用参数放入到环境变量持有对象中
        this.configureEnvironment((ConfigurableEnvironment)environment, applicationArguments.getSourceArgs());
        //监听器监听环境变量对象的变化
        listeners.environmentPrepared((ConfigurableEnvironment)environment);
        this.bindToSpringApplication((ConfigurableEnvironment)environment);
        //如果非web环境，则转换为StandardEnvironment对象
        if (!this.isCustomEnvironment) {
            environment = (new EnvironmentConverter(this.getClassLoader())).convertEnvironmentIfNecessary((ConfigurableEnvironment)environment, this.deduceEnvironmentClass());
        }

        ConfigurationPropertySources.attach((Environment)environment);
        return (ConfigurableEnvironment)environment;
    }
    
getOrCreateEnvironment（）方法：

        private ConfigurableEnvironment getOrCreateEnvironment() {
        //如果已经创建过存放环境变量的对象了，则直接返回
        if (this.environment != null) {
            return this.environment;
        } else {
            //如果是web环境则创建StandardServletEnvironment对象
            //非web环境，创建StandardEnvironment
            switch(this.webApplicationType) {
            case SERVLET:
                return new StandardServletEnvironment();
            case REACTIVE:
                return new StandardReactiveWebEnvironment();
            default:
                return new StandardEnvironment();
            }
        }
    }
    

Banner printedBanner = printBanner(environment);

这句话是输出SpringBoot的Banner信息，可以从指定的位置加载信息，可以输出为文字形式，也可以输出为图片形式，如我们常见的SpringBoot的logo就是在这里输出的
 
    class SpringBootBanner implements Banner {
    private static final String[] BANNER = new String[]{"", "  .   ____          _            __ _ _", " /\\\\ / ___'_ __ _ _(_)_ __  __ _ \\ \\ \\ \\", "( ( )\\___ | '_ | '_| | '_ \\/ _` | \\ \\ \\ \\", " \\\\/  ___)| |_)| | | | | || (_| |  ) ) ) )", "  '  |____| .__|_| |_|_| |_\\__, | / / / /", " =========|_|==============|___/=/_/_/_/"};
    private static final String SPRING_BOOT = " :: Spring Boot :: ";
    private static final int STRAP_LINE_SIZE = 42;

    SpringBootBanner() {
    }
    
createApplicationContext()方法：

    protected ConfigurableApplicationContext createApplicationContext() {
        Class<?> contextClass = this.applicationContextClass;
        if (contextClass == null) {
            try {
                //根据webApplicationType来创建不同的context
                switch(this.webApplicationType) {
                case SERVLET:
                    contextClass = Class.forName("org.springframework.boot.web.servlet.context.AnnotationConfigServletWebServerApplicationContext");
                    break;
                case REACTIVE:
                    contextClass = Class.forName("org.springframework.boot.web.reactive.context.AnnotationConfigReactiveWebServerApplicationContext");
                    break;
                default:
                    contextClass = Class.forName("org.springframework.context.annotation.AnnotationConfigApplicationContext");
                }
            } catch (ClassNotFoundException var3) {
                throw new IllegalStateException("Unable create a default ApplicationContext, please specify an ApplicationContextClass", var3);
            }
        }

        return (ConfigurableApplicationContext)BeanUtils.instantiateClass(contextClass);
    }
    
因为我们是web开发环境，所以这里我们的web上下文是AnnotationConfigEmbeddedWebApplicationContext这个对象

