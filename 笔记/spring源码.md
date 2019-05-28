



```java
Spring源码解析

org.springframework.context.annotation.AnnotationConfigApplicationContext
org.springframework.context.annotation.AnnotationConfigApplicationContext#AnnotationConfigApplicationContext()
	public AnnotationConfigApplicationContext() {
		//创建一个读取注解的bean定义读取器
    	this.reader = new AnnotatedBeanDefinitionReader(this);
		this.scanner = new ClassPathBeanDefinitionScanner(this);
	}



org.springframework.context.support.GenericApplicationContext
    private final DefaultListableBeanFactory beanFactory;

    //实例化 DefaultListableBeanFactory
    public GenericApplicationContext() {
            this.beanFactory = new DefaultListableBeanFactory();
        }


org.springframework.beans.factory.support.DefaultListableBeanFactory
    //bean加载排序
    private Comparator<Object> dependencyComparator;

    /** Map of bean definition objects, keyed by bean name. */
    private final Map<String, BeanDefinition> beanDefinitionMap = new ConcurrentHashMap<>(256);




org.springframework.beans.factory.config.BeanDefinition



org.springframework.beans.factory.annotation.AnnotatedBeanDefinition
	//描述加了注解的类
	public interface AnnotatedBeanDefinition extends BeanDefinition

//////////////////////////////////////////////////////////////////////////////
org.springframework.context.annotation.AnnotatedBeanDefinitionReader

    public AnnotatedBeanDefinitionReader(BeanDefinitionRegistry registry) {
            this(registry, getOrCreateEnvironment(registry));
    }
    
	/**
	 * Get the Environment from the given registry if possible, otherwise return a new
	 * StandardEnvironment.
	 */
	private static Environment getOrCreateEnvironment(BeanDefinitionRegistry registry) {
		Assert.notNull(registry, "BeanDefinitionRegistry must not be null");
		if (registry instanceof EnvironmentCapable) {
			return ((EnvironmentCapable) registry).getEnvironment();
		}
		return new StandardEnvironment();
	}

	
	public AnnotatedBeanDefinitionReader(BeanDefinitionRegistry registry, Environment 			environment) {
		Assert.notNull(registry, "BeanDefinitionRegistry must not be null");
		Assert.notNull(environment, "Environment must not be null");
		this.registry = registry;
		this.conditionEvaluator = new ConditionEvaluator(registry, environment, null);
		AnnotationConfigUtils.registerAnnotationConfigProcessors(this.registry);
	}


org.springframework.context.annotation.AnnotationConfigUtils
/**
	 * Register all relevant annotation post processors in the given registry.
	 * @param registry the registry to operate on
	 */
	public static void registerAnnotationConfigProcessors(BeanDefinitionRegistry 				registry) {
		registerAnnotationConfigProcessors(registry, null);
	}
	
    public static Set<BeanDefinitionHolder> registerAnnotationConfigProcessors(
        BeanDefinitionRegistry registry, @Nullable Object source) {

        DefaultListableBeanFactory beanFactory = unwrapDefaultListableBeanFactory(registry);
        if (beanFactory != null) {
            if (!(beanFactory.getDependencyComparator() instanceof AnnotationAwareOrderComparator)) {
                //排序DependencyComparator
            beanFactory.setDependencyComparator(AnnotationAwareOrderComparator.INSTANCE);
            }
            if (!(beanFactory.getAutowireCandidateResolver() instanceof ContextAnnotationAutowireCandidateResolver)) {
                beanFactory.setAutowireCandidateResolver(new ContextAnnotationAutowireCandidateResolver());
            }
        }
//+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
        Set<BeanDefinitionHolder> beanDefs = new LinkedHashSet<>(8);
   if(!registry.containsBeanDefinition(CONFIGURATION_ANNOTATION_PROCESSOR_BEAN_NAME)) {
            //RootBeanDefinition是BeanDefinition的一种，用来描述spring内部提供的类
  RootBeanDefinition def = new RootBeanDefinition(ConfigurationClassPostProcessor.class);
  def.setSource(source);
  beanDefs.add(registerPostProcessor(registry, def, CONFIGURATION_ANNOTATION_PROCESSOR_BEAN_NAME));
    }
//+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
        if (!registry.containsBeanDefinition(AUTOWIRED_ANNOTATION_PROCESSOR_BEAN_NAME)) {
            RootBeanDefinition def = new RootBeanDefinition(AutowiredAnnotationBeanPostProcessor.class);
            def.setSource(source);
            beanDefs.add(registerPostProcessor(registry, def, AUTOWIRED_ANNOTATION_PROCESSOR_BEAN_NAME));
        }

        // Check for JSR-250 support, and if present add the CommonAnnotationBeanPostProcessor.
        if (jsr250Present && !registry.containsBeanDefinition(COMMON_ANNOTATION_PROCESSOR_BEAN_NAME)) {
            RootBeanDefinition def = new RootBeanDefinition(CommonAnnotationBeanPostProcessor.class);
            def.setSource(source);
            beanDefs.add(registerPostProcessor(registry, def, COMMON_ANNOTATION_PROCESSOR_BEAN_NAME));
        }

        // Check for JPA support, and if present add the PersistenceAnnotationBeanPostProcessor.
        if (jpaPresent && !registry.containsBeanDefinition(PERSISTENCE_ANNOTATION_PROCESSOR_BEAN_NAME)) {
            RootBeanDefinition def = new RootBeanDefinition();
            try {
                def.setBeanClass(ClassUtils.forName(PERSISTENCE_ANNOTATION_PROCESSOR_CLASS_NAME,
                                                    AnnotationConfigUtils.class.getClassLoader()));
            }
            catch (ClassNotFoundException ex) {
                throw new IllegalStateException(
                    "Cannot load optional framework class: " + PERSISTENCE_ANNOTATION_PROCESSOR_CLASS_NAME, ex);
            }
            def.setSource(source);
            beanDefs.add(registerPostProcessor(registry, def, PERSISTENCE_ANNOTATION_PROCESSOR_BEAN_NAME));
        }

        if (!registry.containsBeanDefinition(EVENT_LISTENER_PROCESSOR_BEAN_NAME)) {
            RootBeanDefinition def = new RootBeanDefinition(EventListenerMethodProcessor.class);
            def.setSource(source);
            beanDefs.add(registerPostProcessor(registry, def, EVENT_LISTENER_PROCESSOR_BEAN_NAME));
        }

        if (!registry.containsBeanDefinition(EVENT_LISTENER_FACTORY_BEAN_NAME)) {
            RootBeanDefinition def = new RootBeanDefinition(DefaultEventListenerFactory.class);
            def.setSource(source);
            beanDefs.add(registerPostProcessor(registry, def, EVENT_LISTENER_FACTORY_BEAN_NAME));
        }

        return beanDefs;
    }
	
//-------------------------------------------------------------------
	private static BeanDefinitionHolder registerPostProcessor(
			BeanDefinitionRegistry registry, RootBeanDefinition definition, String beanName) {

		definition.setRole(BeanDefinition.ROLE_INFRASTRUCTURE);
        //注册器的入到map里面去
		registry.registerBeanDefinition(beanName, definition);
		return new BeanDefinitionHolder(definition, beanName);
	}


//注册器
org.springframework.beans.factory.support.BeanDefinitionRegistry



org.springframework.beans.factory.config.BeanDefinitionHolder
	private final BeanDefinition beanDefinition;
	private final String beanName;



//DefaultListableBeanFactory实现了BeanDefinitionRegistry（注册器）
org.springframework.beans.factory.support.DefaultListableBeanFactory

org.springframework.beans.factory.support.DefaultListableBeanFactory#registerBeanDefinition
	//通过实现BeanDefinitionRegistry的registerBeanDefinition方法将RootBeanDefinition（是BeanDefinition的一种）
	@Override
	public void registerBeanDefinition(String beanName, BeanDefinition beanDefinition)
			throws BeanDefinitionStoreException {

		Assert.hasText(beanName, "Bean name must not be empty");
		Assert.notNull(beanDefinition, "BeanDefinition must not be null");

		if (beanDefinition instanceof AbstractBeanDefinition) {
			try {
				((AbstractBeanDefinition) beanDefinition).validate();
			}
			catch (BeanDefinitionValidationException ex) {
				throw new BeanDefinitionStoreException(beanDefinition.getResourceDescription(), beanName,
						"Validation of bean definition failed", ex);
			}
		}

		BeanDefinition existingDefinition = this.beanDefinitionMap.get(beanName);
		if (existingDefinition != null) {
			if (!isAllowBeanDefinitionOverriding()) {
				throw new BeanDefinitionOverrideException(beanName, beanDefinition, existingDefinition);
			}
			else if (existingDefinition.getRole() < beanDefinition.getRole()) {
				// e.g. was ROLE_APPLICATION, now overriding with ROLE_SUPPORT or ROLE_INFRASTRUCTURE
				if (logger.isInfoEnabled()) {
					logger.info("Overriding user-defined bean definition for bean '" + beanName +
							"' with a framework-generated bean definition: replacing [" +
							existingDefinition + "] with [" + beanDefinition + "]");
				}
			}
			else if (!beanDefinition.equals(existingDefinition)) {
				if (logger.isDebugEnabled()) {
					logger.debug("Overriding bean definition for bean '" + beanName +
							"' with a different definition: replacing [" + existingDefinition +
							"] with [" + beanDefinition + "]");
				}
			}
			else {
				if (logger.isTraceEnabled()) {
					logger.trace("Overriding bean definition for bean '" + beanName +
							"' with an equivalent definition: replacing [" + existingDefinition +
							"] with [" + beanDefinition + "]");
				}
			}
			this.beanDefinitionMap.put(beanName, beanDefinition);
		}
		else {
			if (hasBeanCreationStarted()) {
				// Cannot modify startup-time collection elements anymore (for stable iteration)
				synchronized (this.beanDefinitionMap) {
					this.beanDefinitionMap.put(beanName, beanDefinition);
					List<String> updatedDefinitions = new ArrayList<>(this.beanDefinitionNames.size() + 1);
					updatedDefinitions.addAll(this.beanDefinitionNames);
					updatedDefinitions.add(beanName);
					this.beanDefinitionNames = updatedDefinitions;
					if (this.manualSingletonNames.contains(beanName)) {
						Set<String> updatedSingletons = new LinkedHashSet<>(this.manualSingletonNames);
						updatedSingletons.remove(beanName);
						this.manualSingletonNames = updatedSingletons;
					}
				}
			}
           //++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
			else {
				// Still in startup registration phase
				this.beanDefinitionMap.put(beanName, beanDefinition);
				this.beanDefinitionNames.add(beanName);
				this.manualSingletonNames.remove(beanName);
			}
           //+++++++++++++++++++++++++++++++++++++++++++++++++++++++++
			this.frozenBeanDefinitionNames = null;
		}

		if (existingDefinition != null || containsSingleton(beanName)) {
			resetBeanDefinition(beanName);
		}
	}


```







```java
Sping AOP源码解析
1、@EnableAspectJAutoProxy
	@Import(AspectJAutoProxyRegistrar.class)给容器中导入AspectJAutoProxyRegistrar
		//利用AspectJAutoProxyRegistrar自定义给容器中注册bean； BeanDefinetion
		internalAutoProxyCreator = AnnotationAwareAspectJAutoProxyCreator
给容器中注册一个AnnotationAwareAspectJAutoProxyCreator


2、AnnotationAwareAspectJAutoProxyCreator(后置处理器)
	->AspectJAwareAdvisorAutoProxyCreator
		->AbstractAdvisorAutoProxyCreator
			->AbstractAutoProxyCreator
				implements SmartInstantiationAwareBeanPostProcessor, BeanFactoryAware
			关注后置处理器（在bean初始化完成前后做的事情）、自动装配BeanFactory

AbstractAutoProxyCreator.setBeanFactory();
AbstractAutoProxyCreator.有后置处理器的逻辑；

AbstractAdvisorAutoProxyCreator.setBeanFactory() -》 initBeanFactory()
AnnotationAwareAspectJAutoProxyCreator.initBeanFactory()   

    
    
    
//流程：
//1.传入主配置类，创建ioc容器
org.springframework.context.annotation.AnnotationConfigApplicationContext
    public AnnotationConfigApplicationContext(Class<?>... annotatedClasses) {
            this();
            register(annotatedClasses);
            refresh();
        }

//2.注册配置类，调用refresh（）刷新容器
	register(annotatedClasses);
	@Override
	public void refresh() throws BeansException, IllegalStateException {
		synchronized (this.startupShutdownMonitor) {
			// Prepare this context for refreshing.
			prepareRefresh();

			// Tell the subclass to refresh the internal bean factory.
			ConfigurableListableBeanFactory beanFactory = obtainFreshBeanFactory();

			// Prepare the bean factory for use in this context.
			prepareBeanFactory(beanFactory);

			try {
				// Allows post-processing of the bean factory in context subclasses.
				postProcessBeanFactory(beanFactory);

				// Invoke factory processors registered as beans in the context.
				invokeBeanFactoryPostProcessors(beanFactory);

				// Register bean processors that intercept bean creation.
				registerBeanPostProcessors(beanFactory);

				// Initialize message source for this context.
				initMessageSource();

				// Initialize event multicaster for this context.
				initApplicationEventMulticaster();

				// Initialize other special beans in specific context subclasses.
				onRefresh();

				// Check for listener beans and register them.
				registerListeners();

				// Instantiate all remaining (non-lazy-init) singletons.
				finishBeanFactoryInitialization(beanFactory);

				// Last step: publish corresponding event.
				finishRefresh();
			}

			catch (BeansException ex) {
				if (logger.isWarnEnabled()) {
					logger.warn("Exception encountered during context initialization - " +
							"cancelling refresh attempt: " + ex);
				}

				// Destroy already created singletons to avoid dangling resources.
				destroyBeans();

				// Reset 'active' flag.
				cancelRefresh(ex);

				// Propagate exception to caller.
				throw ex;
			}

			finally {
				// Reset common introspection caches in Spring's core, since we
				// might not ever need metadata for singleton beans anymore...
				resetCommonCaches();
			}
		}
	}

//3.registerBeanPostProcessors(beanFactory);注册bean的后置处理器来方便拦截bean的创建
	//1.先获取ioc容器的已经定义了的需要创建的对象的所有BeanPostProcessor
org.springframework.context.support.PostProcessorRegistrationDelegate#registerBeanPostProcessors(org.springframework.beans.factory.config.ConfigurableListableBeanFactory, org.springframework.context.support.AbstractApplicationContext)
     String[] postProcessorNames = beanFactory.getBeanNamesForType(BeanPostProcessor.class, true, false);   

	//2.给容器中加了别的BeanPostProcessor
	beanFactory.addBeanPostProcessor(new BeanPostProcessorChecker(beanFactory, beanProcessorTargetCount));

	//3.优先注册实现了PriorityOrdered接口的BeanPostProcessor
	//排序
	sortPostProcessors(priorityOrderedPostProcessors, beanFactory);
	registerBeanPostProcessors(beanFactory, priorityOrderedPostProcessors);
	--> org.springframework.context.support.PostProcessorRegistrationDelegate#registerBeanPostProcessors(org.springframework.beans.factory.config.ConfigurableListableBeanFactory, java.util.List<org.springframework.beans.factory.config.BeanPostProcessor>)
        private static void registerBeanPostProcessors(
			ConfigurableListableBeanFactory beanFactory, List<BeanPostProcessor> postProcessors) {

		for (BeanPostProcessor postProcessor : postProcessors) {
            //+++++++++把BeanPostProcessor添加到beanFactory工厂中
			beanFactory.addBeanPostProcessor(postProcessor);
		}
	}


	//4.再给容器注册实现了Ordered接口的BeanPostProcessor
	sortPostProcessors(orderedPostProcessors, beanFactory);
	registerBeanPostProcessors(beanFactory, orderedPostProcessors);

	//5.注册没有实现优先级接口的BeanPostProcessors；
	registerBeanPostProcessors(beanFactory, nonOrderedPostProcessors);

	//6.注册BeanPostProcessor.实际上就是创建BeanPostProcessor对象，保存在容器中
	//创建internalAutoProxyCreator的BeanPostProcessor[AnnotationAwareAspectJAutoProxyCreator]
		//1.创建Bean实例
org.springframework.context.support.PostProcessorRegistrationDelegate#registerBeanPostProcessors(org.springframework.beans.factory.config.ConfigurableListableBeanFactory, org.springframework.context.support.AbstractApplicationContext)
    BeanPostProcessor pp = beanFactory.getBean(ppName, BeanPostProcessor.class);

org.springframework.beans.factory.support.AbstractBeanFactory#getBean(java.lang.String, java.lang.Class<T>)
	@Override
	public <T> T getBean(String name, Class<T> requiredType) throws BeansException {
		return doGetBean(name, requiredType, null, false);
	}

org.springframework.beans.factory.support.AbstractBeanFactory#doGetBean
	// Create bean instance.
    if (mbd.isSingleton()) {
    //+++++++++++++++++++++++++++
        sharedInstance = getSingleton(beanName, () -> {
            try {
           	//+++++++++++++++++++++++++++++++++++++++如果没有则创建Bean
                return createBean(beanName, mbd, args);
            }
            catch (BeansException ex) {
                // Explicitly remove instance from singleton cache: It might have been put there
                // eagerly by the creation process, to allow for circular reference resolution.
                // Also remove any beans that received a temporary reference to the bean.
                destroySingleton(beanName);
                throw ex;
            }
        });
        bean = getObjectForBeanInstance(sharedInstance, name, beanName, mbd);
    }

org.springframework.beans.factory.support.DefaultSingletonBeanRegistry#getSingleton(java.lang.String, org.springframework.beans.factory.ObjectFactory<?>)
    
    public Object getSingleton(String beanName, ObjectFactory<?> singletonFactory) {
		Assert.notNull(beanName, "Bean name must not be null");
		synchronized (this.singletonObjects) {
			Object singletonObject = this.singletonObjects.get(beanName);
			if (singletonObject == null) {
				if (this.singletonsCurrentlyInDestruction) {
					throw new BeanCreationNotAllowedException(beanName,
							"Singleton bean creation not allowed while singletons of this factory are in destruction " +
							"(Do not request a bean from a BeanFactory in a destroy method implementation!)");
				}
				if (logger.isDebugEnabled()) {
					logger.debug("Creating shared instance of singleton bean '" + beanName + "'");
				}
				beforeSingletonCreation(beanName);
				boolean newSingleton = false;
				boolean recordSuppressedExceptions = (this.suppressedExceptions == null);
				if (recordSuppressedExceptions) {
					this.suppressedExceptions = new LinkedHashSet<>();
				}
				try {
                   	//获取单例对象
                    //++++++++++++++++++++++++++++++++++++++++++++++
					singletonObject = singletonFactory.getObject();
					newSingleton = true;
				}
				catch (IllegalStateException ex) {
					// Has the singleton object implicitly appeared in the meantime ->
					// if yes, proceed with it since the exception indicates that state.
					singletonObject = this.singletonObjects.get(beanName);
					if (singletonObject == null) {
						throw ex;
					}
				}
				catch (BeanCreationException ex) {
					if (recordSuppressedExceptions) {
						for (Exception suppressedException : this.suppressedExceptions) {
							ex.addRelatedCause(suppressedException);
						}
					}
					throw ex;
				}
				finally {
					if (recordSuppressedExceptions) {
						this.suppressedExceptions = null;
					}
					afterSingletonCreation(beanName);
				}
				if (newSingleton) {
					addSingleton(beanName, singletonObject);
				}
			}
			return singletonObject;
		}
	}


org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory#createBean(java.lang.String, org.springframework.beans.factory.support.RootBeanDefinition, java.lang.Object[])
    try {	
        	//++++++++++++++++++++++++++++创建Bean
			Object beanInstance = doCreateBean(beanName, mbdToUse, args);
			if (logger.isTraceEnabled()) {
				logger.trace("Finished creating instance of bean '" + beanName + "'");
			}
			return beanInstance;
		}

org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory#doCreateBean

protected Object doCreateBean(final String beanName, final RootBeanDefinition mbd, final @Nullable Object[] args)
			throws BeanCreationException {

		// Instantiate the bean.
		BeanWrapper instanceWrapper = null;
		if (mbd.isSingleton()) {
            //创建BeanPostProcessor[AnnotationAwareAspectJAutoProxyCreator]
			instanceWrapper = this.factoryBeanInstanceCache.remove(beanName);
		}
		if (instanceWrapper == null) {
			instanceWrapper = createBeanInstance(beanName, mbd, args);
		}
    //+++++++++++++++++++++++实例化bean++++++++++++++++++++++++++++++
		final Object bean = instanceWrapper.getWrappedInstance();
		Class<?> beanType = instanceWrapper.getWrappedClass();
		if (beanType != NullBean.class) {
			mbd.resolvedTargetType = beanType;
		}

		// Allow post-processors to modify the merged bean definition.
		synchronized (mbd.postProcessingLock) {
			if (!mbd.postProcessed) {
				try {
					applyMergedBeanDefinitionPostProcessors(mbd, beanType, beanName);
				}
				catch (Throwable ex) {
					throw new BeanCreationException(mbd.getResourceDescription(), beanName,
							"Post-processing of merged bean definition failed", ex);
				}
				mbd.postProcessed = true;
			}
		}

		// Eagerly cache singletons to be able to resolve circular references
		// even when triggered by lifecycle interfaces like BeanFactoryAware.
		boolean earlySingletonExposure = (mbd.isSingleton() && this.allowCircularReferences &&
				isSingletonCurrentlyInCreation(beanName));
		if (earlySingletonExposure) {
			if (logger.isTraceEnabled()) {
				logger.trace("Eagerly caching bean '" + beanName +
						"' to allow for resolving potential circular references");
			}
			addSingletonFactory(beanName, () -> getEarlyBeanReference(beanName, mbd, bean));
		}

		// Initialize the bean instance.(初始化bean实例)
		Object exposedObject = bean;
		try {
            //++++++++++++++++++++++++++++++++++++给bean的各种属性赋值
			populateBean(beanName, mbd, instanceWrapper);
			exposedObject = initializeBean(beanName, exposedObject, mbd);
		}
		catch (Throwable ex) {
			if (ex instanceof BeanCreationException && beanName.equals(((BeanCreationException) ex).getBeanName())) {
				throw (BeanCreationException) ex;
			}
			else {
				throw new BeanCreationException(
						mbd.getResourceDescription(), beanName, "Initialization of bean failed", ex);
			}
		}

		if (earlySingletonExposure) {
			Object earlySingletonReference = getSingleton(beanName, false);
			if (earlySingletonReference != null) {
				if (exposedObject == bean) {
					exposedObject = earlySingletonReference;
				}
				else if (!this.allowRawInjectionDespiteWrapping && hasDependentBean(beanName)) {
					String[] dependentBeans = getDependentBeans(beanName);
					Set<String> actualDependentBeans = new LinkedHashSet<>(dependentBeans.length);
					for (String dependentBean : dependentBeans) {
						if (!removeSingletonIfCreatedForTypeCheckOnly(dependentBean)) {
							actualDependentBeans.add(dependentBean);
						}
					}
					if (!actualDependentBeans.isEmpty()) {
						throw new BeanCurrentlyInCreationException(beanName,
								"Bean with name '" + beanName + "' has been injected into other beans [" +
								StringUtils.collectionToCommaDelimitedString(actualDependentBeans) +
								"] in its raw version as part of a circular reference, but has eventually been " +
								"wrapped. This means that said other beans do not use the final version of the " +
								"bean. This is often the result of over-eager type matching - consider using " +
								"'getBeanNamesOfType' with the 'allowEagerInit' flag turned off, for example.");
					}
				}
			}
		}

		// Register bean as disposable.
		try {
			registerDisposableBeanIfNecessary(beanName, bean, mbd);
		}
		catch (BeanDefinitionValidationException ex) {
			throw new BeanCreationException(
					mbd.getResourceDescription(), beanName, "Invalid destruction signature", ex);
		}

		return exposedObject;
	}

		//2.populateBean(beanName, mbd, instanceWrapper);给bean的各种属性赋值
		populateBean(beanName, mbd, instanceWrapper);

		//3.初始化Bean,initializeBean(beanName, exposedObject, mbd);
		exposedObject = initializeBean(beanName, exposedObject, mbd);

			//3.1.invokeAwareMethods(beanName, bean); 处理Aware接口的方法的回调
org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory#initializeBean(java.lang.String, java.lang.Object, org.springframework.beans.factory.support.RootBeanDefinition)
	protected Object initializeBean(final String beanName, final Object bean, @Nullable RootBeanDefinition mbd) {
		if (System.getSecurityManager() != null) {
			AccessController.doPrivileged((PrivilegedAction<Object>) () -> {
				invokeAwareMethods(beanName, bean);
				return null;
			}, getAccessControlContext());
		}
		else {
            //+++++++++invokeAwareMethods(beanName, bean); 处理Aware接口的方法的回调++++++
			invokeAwareMethods(beanName, bean);
		}

		Object wrappedBean = bean;
		if (mbd == null || !mbd.isSynthetic()) {
            //++applyBeanPostProcessorsBeforeInitialization（）；应用后置处理器的postProcessBeforeInitialization(result, beanName);++++++++++++++
			wrappedBean = applyBeanPostProcessorsBeforeInitialization(wrappedBean, beanName);
		}

		try {
            //++++++++执行自定义的初始化方法+++++++++++
			invokeInitMethods(beanName, wrappedBean, mbd);
		}
		catch (Throwable ex) {
			throw new BeanCreationException(
					(mbd != null ? mbd.getResourceDescription() : null),
					beanName, "Invocation of init method failed", ex);
		}
		if (mbd == null || !mbd.isSynthetic()) {
            //+++执行后置处理器的postProcessAfterInitialization（）;++++
			wrappedBean = applyBeanPostProcessorsAfterInitialization(wrappedBean, beanName);
		}

		return wrappedBean;
	}



org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory#invokeAwareMethods
	private void invokeAwareMethods(final String beanName, final Object bean) {
	//+++++++++++++++处理Aware接口的方法的回调 +++++++++++++++++
		if (bean instanceof Aware) {
			if (bean instanceof BeanNameAware) {
				((BeanNameAware) bean).setBeanName(beanName);
			}
			if (bean instanceof BeanClassLoaderAware) {
				ClassLoader bcl = getBeanClassLoader();
				if (bcl != null) {
					((BeanClassLoaderAware) bean).setBeanClassLoader(bcl);
				}
			}
			if (bean instanceof BeanFactoryAware) {
				((BeanFactoryAware) bean).setBeanFactory(AbstractAutowireCapableBeanFactory.this);
			}
		}
	}


			//3.2.applyBeanPostProcessorsBeforeInitialization（）；应用后置处理器的postProcessBeforeInitialization(result, beanName);
org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory#applyBeanPostProcessorsBeforeInitialization
	@Override
	public Object applyBeanPostProcessorsBeforeInitialization(Object existingBean, String beanName)
			throws BeansException {

		Object result = existingBean;
		for (BeanPostProcessor processor : getBeanPostProcessors()) {
            
            //++++应用后置处理器的postProcessBeforeInitialization(result, beanName);
			Object current = processor.postProcessBeforeInitialization(result, beanName);
			if (current == null) {
				return result;
			}
			result = current;
		}
		return result;
	}
			//3.3.invokeInitMethods（）；执行自定义的初始化方法

			//3.4.applyBeanPostProcessorsAfterInitialization；执行后置处理器的postProcessAfterInitialization（）;
org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory#applyBeanPostProcessorsAfterInitialization

	@Override
	public Object applyBeanPostProcessorsAfterInitialization(Object existingBean, String beanName)
			throws BeansException {

		Object result = existingBean;
		for (BeanPostProcessor processor : getBeanPostProcessors()) {
            //++++执行后置处理器的postProcessAfterInitialization（）;
			Object current = processor.postProcessAfterInitialization(result, beanName);
			if (current == null) {
				return result;
			}
			result = current;
		}
		return result;
	}



		//4.BeanPostProcessor(AnnotationAwareAspectJAutoProxyCreator)创建成功： --》aspectJAvisorsBuilder
在invokeAwareMethods(beanName, bean); 处理Aware接口的方法的回调
org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory#invokeAwareMethods
	private void invokeAwareMethods(final String beanName, final Object bean) {
	//+++++++++++++++处理Aware接口的方法的回调 +++++++++++++++++
		if (bean instanceof Aware) {
			if (bean instanceof BeanNameAware) {
				((BeanNameAware) bean).setBeanName(beanName);
			}
			if (bean instanceof BeanClassLoaderAware) {
				ClassLoader bcl = getBeanClassLoader();
				if (bcl != null) {
					((BeanClassLoaderAware) bean).setBeanClassLoader(bcl);
				}
			}
			if (bean instanceof BeanFactoryAware) {
				((BeanFactoryAware) bean).setBeanFactory(AbstractAutowireCapableBeanFactory.this);
			}
		}
	}


org.springframework.aop.framework.autoproxy.AbstractAdvisorAutoProxyCreator#setBeanFactory
在AbstractAdvisorAutoProxyCreator extends AbstractAutoProxyCreator 
	@Override
	public void setBeanFactory(BeanFactory beanFactory) {
		super.setBeanFactory(beanFactory);
		if (!(beanFactory instanceof ConfigurableListableBeanFactory)) {
			throw new IllegalArgumentException(
					"AdvisorAutoProxyCreator requires a ConfigurableListableBeanFactory: " + beanFactory);
		}
		initBeanFactory((ConfigurableListableBeanFactory) beanFactory);
	}


org.springframework.aop.aspectj.annotation.AnnotationAwareAspectJAutoProxyCreator#initBeanFactory
在AnnotationAwareAspectJAutoProxyCreator extends AspectJAwareAdvisorAutoProxyCreator
	@Override
	protected void initBeanFactory(ConfigurableListableBeanFactory beanFactory) {
		super.initBeanFactory(beanFactory);
		if (this.aspectJAdvisorFactory == null) {
		//反射的通知工厂
			this.aspectJAdvisorFactory = new ReflectiveAspectJAdvisorFactory(beanFactory);
		}
		//BeanPostProcessor(AnnotationAwareAspectJAutoProxyCreator)创建成功： --》aspectJAvisorsBuilder
		this.aspectJAdvisorsBuilder =
				new BeanFactoryAspectJAdvisorsBuilderAdapter(beanFactory, this.aspectJAdvisorFactory);
	}
	
//7. 把BeanPostProcessor添加到beanFactory工厂中
	beanFactory.addBeanPostProcessor(postProcessor);
	
```



