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

