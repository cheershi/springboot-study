# springboot启动原理(三)

prepareContext（）方法

    private void prepareContext(ConfigurableApplicationContext context, ConfigurableEnvironment environment, SpringApplicationRunListeners listeners, ApplicationArguments applicationArguments, Banner printedBanner) {
        //将之前创建的环境变量的对象放入到应用上下文中
        context.setEnvironment(environment);
        //像Spring容器中先行注入BeanNameGenerator和ResourceLoader的实例
        this.postProcessApplicationContext(context);
        //调用之前在SpringBoot中加载的ApplicationContextInitializer的实现类，先进行一些初始化的动作
    //在之前的文章中我分析过在SpringBoot启动的过程中会从META-INF/spring.factories中加载ApplicationContextInitializer的对象
        this.applyInitializers(context);
        //这里是一个空实现
        listeners.contextPrepared(context);
        if (this.logStartupInfo) {
            this.logStartupInfo(context.getParent() == null);
            this.logStartupProfileInfo(context);
        }
        //将之前得到的的应用参数解析器注入到Spring容器中
        context.getBeanFactory().registerSingleton("springApplicationArguments", applicationArguments);
        if (printedBanner != null) {
        //将之前得到的的springBootBanner注入到Spring容器中
            context.getBeanFactory().registerSingleton("springBootBanner", printedBanner);
        }
        // 加载sources 这里是指我们的启动类
        Set<Object> sources = this.getAllSources();
        //sources不能为空
        Assert.notEmpty(sources, "Sources must not be empty");
        this.load(context, sources.toArray(new Object[0]));
        //监听上下文的变化
        listeners.contextLoaded(context);
    }
    
load（）方法：

    protected void load(ApplicationContext context, Object[] sources) {
        if (logger.isDebugEnabled()) {
            logger.debug("Loading source " + StringUtils.arrayToCommaDelimitedString(sources));
        }
        //创建BeanDefinition的加载器 直接new BeanDefinitionLoader
        BeanDefinitionLoader loader = this.createBeanDefinitionLoader(this.getBeanDefinitionRegistry(context), sources);
        if (this.beanNameGenerator != null) {
        //如果beanNameGenerator 对象不为空的话，则在BeanDefinitionLoader中set beanNameGenerator 
            loader.setBeanNameGenerator(this.beanNameGenerator);
        }

        if (this.resourceLoader != null) {
        /如果resourceLoader 对象不为空的话，则在BeanDefinitionLoader中set resourceLoader 
            loader.setResourceLoader(this.resourceLoader);
        }

        if (this.environment != null) {
        //如果environment 对象不为空的话，则在BeanDefinitionLoader中set environment
            loader.setEnvironment(this.environment);
        }
        //用BeanDefinitionLoader进行加载
        loader.load();
    }
    
getBeanDefinitionRegistry（）方法：

    private BeanDefinitionRegistry getBeanDefinitionRegistry(ApplicationContext context) {
        //由于AnnotationConfigEmbeddedWebApplicationContext实现了BeanDefinitionRegistry接口，所以这里直接返回AnnotationConfigEmbeddedWebApplicationContext的实例
        if (context instanceof BeanDefinitionRegistry) {
            return (BeanDefinitionRegistry)context;
        } else if (context instanceof AbstractApplicationContext) {
        //DefaultListableBeanFactory也实现了BeanDefinitionRegistry的接口
            return (BeanDefinitionRegistry)((AbstractApplicationContext)context).getBeanFactory();
        } else {
            throw new IllegalStateException("Could not locate BeanDefinitionRegistry");
        }
    }
    
loader.load()

    public int load() {
        int count = 0;
        //这里的sources即是我们上面说的sources
        Object[] var2 = this.sources;
        int var3 = var2.length;

        for(int var4 = 0; var4 < var3; ++var4) {
            Object source = var2[var4];
            count += this.load(source);
        }

        return count;
    }
    
    
    private int load(Object source) {
        Assert.notNull(source, "Source must not be null");
        //如果是Class类型 
        if (source instanceof Class) {
            return this.load((Class)source);
        }
        //如果是Resource类型
        else if (source instanceof Resource) {
            return this.load((Resource)source);
        } else if (source instanceof Package) {
            return this.load((Package)source);
        } else if (source instanceof CharSequence) {
            return this.load((CharSequence)source);
        } else {
        //不是上面说的四种类型，则抛出异常
            throw new IllegalArgumentException("Invalid source type " + source.getClass());
        }
    }
    
    private int load(Class<?> source) {
    //如果是Groovy环境
    if (isGroovyPresent()) {
        // Any GroovyLoaders added in beans{} DSL can contribute beans here
        if (GroovyBeanDefinitionSource.class.isAssignableFrom(source)) {
            GroovyBeanDefinitionSource loader = BeanUtils.instantiateClass(source,
                    GroovyBeanDefinitionSource.class);
            load(loader);
        }
    }
    //判断当前Class上面是否有Component注解
    if (isComponent(source)) {
        //将启动应用主类注入到Spring容器中 由于SpringBoot的自动配置的特性，所以这里在注入Spring容器的过程中会判断应不应该将这个类注入到Spring容器中
        this.annotatedReader.register(source);
        return 1;
    }
    return 0;
    }
