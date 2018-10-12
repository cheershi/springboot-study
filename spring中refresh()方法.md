# spring中refresh（）方法：

    public void refresh() throws BeansException, IllegalStateException {
        Object var1 = this.startupShutdownMonitor;
        synchronized(this.startupShutdownMonitor) {
            //准备工作包括设置启动时间，是否激活标识位，初始化属性源(property source)配置
            this.prepareRefresh();
            //创建beanFactory（过程是根据xml为每个bean生成BeanDefinition并注册到生成的beanFactory）
            ConfigurableListableBeanFactory beanFactory = this.obtainFreshBeanFactory();
            //准备创建好的beanFactory（给beanFactory设置ClassLoader，设置SpEL表达式解析器，设置类型转化器【能将xml String类型转成相应对象】，增加内置ApplicationContextAwareProcessor对象，忽略各种Aware对象，注册各种内置的对账对象【BeanFactory，ApplicationContext】等，注册AOP相关的一些东西，注册环境相关的一些bean
            this.prepareBeanFactory(beanFactory);

            try {
                this.postProcessBeanFactory(beanFactory);
                //实例化并调用BeanFactory中扩展了BeanFactoryPostProcessor的Bean的postProcessBeanFactory方法
                this.invokeBeanFactoryPostProcessors(beanFactory);
                //实例化和注册beanFactory中扩展了BeanPostProcessor的bean
                this.registerBeanPostProcessors(beanFactory);
                //实例化，注册和设置国际化工具类MessageSource
                this.initMessageSource();
                //实例化，注册和设置消息广播类（如果没有自己定义使用默认的SimpleApplicationEventMulticaster实现，此广播使用同步的通知方式）
                this.initApplicationEventMulticaster();
                //设置样式工具ThemeSource
                this.onRefresh();
                //添加用户定义的消息接收器到上面设置的消息广播ApplicationEventMulticaster
                this.registerListeners();
                //设置自定义的类型转化器ConversionService，设置自定义AOP相关的类LoadTimeWeaverAware，清除临时的ClassLoader，冻结配置（没看明白干什么的），实例化所有的类（懒加载的类除外）
                this.finishBeanFactoryInitialization(beanFactory);
                //注册和设置跟bean生命周期相关的类（默认使用DefaultLifecycleProcessor），调用扩展了SmartLifecycle接口的start方法，使用上注册的广播类消息广播类ApplicationEventMulticaster广播ContextRefreshedEvent事件 
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
