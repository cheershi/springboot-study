# springboot启动原理（四）

AnnotationConfigEmbeddedWebApplicationContext这个类继承了EmbeddedWebApplicationContext类，GenericWebApplicationContext类(这个要注意)，它还实现了BeanDefinitionRegistry这个接口，还实现了ResourceLoader这个接口，这个类可以说是一个全能类了

refreshContext（）方法：


    public void refresh() throws BeansException, IllegalStateException {
        Object var1 = this.startupShutdownMonitor;
        synchronized(this.startupShutdownMonitor) {
            this.prepareRefresh();
            ConfigurableListableBeanFactory beanFactory = this.obtainFreshBeanFactory();
            this.prepareBeanFactory(beanFactory);

            try {
                this.postProcessBeanFactory(beanFactory);
                this.invokeBeanFactoryPostProcessors(beanFactory);
                this.registerBeanPostProcessors(beanFactory);
                this.initMessageSource();
                this.initApplicationEventMulticaster();
                this.onRefresh();
                this.registerListeners();
                this.finishBeanFactoryInitialization(beanFactory);
                this.finishRefresh();
            } catch (BeansException var9) {
                if (this.logger.isWarnEnabled()) {
                    this.logger.warn("Exception encountered during context initialization - cancelling refresh attempt: " + var9);
                }

                this.destroyBeans();
                this.cancelRefresh(var9);
                throw var9;
            } finally {
                this.resetCommonCaches();
            }

        }
    }
 
 prepareRefresh方法
表示在真正做refresh操作之前需要准备做的事情：

设置Spring容器的启动时间，撤销关闭状态，开启活跃状态。

初始化属性源信息(Property)

验证环境信息里一些必须存在的属性


    protected void prepareRefresh() {
        //设置Spring容器的启动时间
        this.startupDate = System.currentTimeMillis();
        //撤销关闭状态
        this.closed.set(false);
        //开启活跃状态。
        this.active.set(true);
        if (this.logger.isInfoEnabled()) {
            this.logger.info("Refreshing " + this);
        }
         //初始化属性源信息(Property)
        this.initPropertySources();
        //验证环境信息里一些必须存在的属性
        this.getEnvironment().validateRequiredProperties();
        this.earlyApplicationEvents = new LinkedHashSet();
    }

prepareBeanFactory方法

从Spring容器获取BeanFactory(Spring Bean容器)并进行相关的设置为后续的使用做准备：

1.设置classloader(用于加载bean)，设置表达式解析器(解析bean定义中的一些表达式)，添加属性编辑注册器(注册属性编辑器)

2.添加ApplicationContextAwareProcessor这个BeanPostProcessor。取消ResourceLoaderAware、ApplicationEventPublisherAware、MessageSourceAware、ApplicationContextAware、EnvironmentAware这5个接口的自动注入。因为ApplicationContextAwareProcessor把这5个接口的实现工作做了

3.设置特殊的类型对应的bean。BeanFactory对应刚刚获取的BeanFactory；ResourceLoader、ApplicationEventPublisher、ApplicationContext这3个接口对应的bean都设置为当前的Spring容器

4.注入一些其它信息的bean，比如environment、systemProperties等

postProcessBeanFactory方法

BeanFactory设置之后再进行后续的一些BeanFactory操作。

不同的Spring容器做不同的操作。比如GenericWebApplicationContext容器会在BeanFactory中添加ServletContextAwareProcessor用于处理ServletContextAware类型的bean初始化的时候调用setServletContext或者setServletConfig方法(跟ApplicationContextAwareProcessor原理一样)。

AnnotationConfigEmbeddedWebApplicationContext对应的postProcessBeanFactory方法：

    @Override
    protected void postProcessBeanFactory(ConfigurableListableBeanFactory beanFactory) {
      // 调用父类EmbeddedWebApplicationContext的实现
      super.postProcessBeanFactory(beanFactory);
      // 查看basePackages属性，如果设置了会使用ClassPathBeanDefinitionScanner去扫描basePackages包下的bean并注册
      if (this.basePackages != null && this.basePackages.length > 0) {
        this.scanner.scan(this.basePackages);
      }
      // 查看annotatedClasses属性，如果设置了会使用AnnotatedBeanDefinitionReader去注册这些bean
      if (this.annotatedClasses != null && this.annotatedClasses.length > 0) {
        this.reader.register(this.annotatedClasses);
      }
    }
父类EmbeddedWebApplicationContext的实现：

    @Override
    protected void postProcessBeanFactory(ConfigurableListableBeanFactory beanFactory) {
      beanFactory.addBeanPostProcessor(
          new WebApplicationContextServletContextAwareProcessor(this));
      beanFactory.ignoreDependencyInterface(ServletContextAware.class);
    }


