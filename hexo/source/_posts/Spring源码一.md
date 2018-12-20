---
title: Spring源码一
date: 2018-10-27 11:22:12
tags: [Spring]
categories: [Spring]
---

Spring源码(1)<!--more-->

#### 源码下载

写的比较匆忙 ，结合Ctrl+F进行寻找关键字进行查询是最妙的。

* 下载源码

  首先从github上下载源码

  ```
  git clonehttps://github.com/spring-projects/spring-framework.git spring-framework
  ```

  你也可以用SourceTree (一个界面化的git工具) 来管理你的git项目

* 下载gradle

  ```
  https://gradle.org/
  ```

  下载完之后需要进行环境变量的配置

  * 在环境变量中配置``GRADLE_HOME=D:\ENV\gradle-4.10.2-bin\gradle-4.10.2``

  * 添加path变量 ``%GRADLE_HOME%\bin``

  * 测试

    ```
    PS C:\Users\andreby\Desktop> gradle -version
    
    ------------------------------------------------------------
    Gradle 4.10.2
    ------------------------------------------------------------
    
    Build time:   2018-09-19 18:10:15 UTC
    Revision:     b4d8d5d170bb4ba516e88d7fe5647e2323d791dd
    
    Kotlin DSL:   1.0-rc-6
    Kotlin:       1.2.61
    Groovy:       2.4.15
    Ant:          Apache Ant(TM) version 1.9.11 compiled on March 23 2018
    JVM:          1.8.0_171 (Oracle Corporation 25.171-b11)
    OS:           Windows 10 10.0 amd64
    ```

* 使用gradle将源码转换为eclipse 工程

  Spring是分多模块的，所以你要查看哪个模块的源码，就进哪个模块的目录下执行以下命令就可以看到了。

  ```
  gradle cleanIdea eclipse 
  ```

  **可能需要梯子**

* 导入工程,即可食用,注意各个工程之间的依赖

* 导入后的问题

  导入后发现一片红色感叹号，看下sts中的problems的选项框，发现是缺少jar包

  ``spring-cglib-repack-3.2.8 `` 和``spring-objenesis-repack-3.0.1.jar ``

   我查了下网上大神们的说法：

  **通过阅读源码发现为了避免第三方 class 的冲突，spring 把最新的 cglib 和 objenesis 给 repack 了**

  参考链接：

  []: https://blog.csdn.net/sekiu/article/details/50624180

#### Spring-Core

正常的spring启动，是以xml配置bean然后进行启动，web项目由web容器来管理spring容器。

我们在applicationContext.xml中配置好bean后，那么启动时候加载这个xml就可以了。

### web容器启动流程

以web项目为例子 springmvc项目 一般使用web.xml进行管理web容器，而spring容器一般交付给web容器进行管理。

* tomcat 启动，加载context监听器。

  首先tomcat的StandardContext类在tomcat启动时候会调用``listenerStart``

  去加载配置在web.xml中的监听器

  webxml中的配置的spring的ContextLoadListener

  ContextLoaderListener 继承自ServletContextListener

  嗯，首先加载WebAppRootListener 这个也是继承自ServletContextListener的

  ```
  ...
  <listener>
  <listener-class>org.springframework.web.util.WebAppRootListener</listener-class>
  </listener>
  <listener>
   <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
  </listener>
  <listener>
      <listener-class>org.springframework.web.util.IntrospectorCleanupListener</listener-class>
  </listener>
  ```

  1.StardandContext中的listenerStart方法：

  ```
     /**
       * Configure the set of instantiated application event listeners
       * for this Context.
       * @return <code>true</code> if all listeners wre
       * initialized successfully, or <code>false</code> otherwise.
       */
      public boolean listenerStart() {
  
          if (log.isDebugEnabled())
              log.debug("Configuring application event listeners");
  
          // Instantiate the required listeners
          String listeners[] = findApplicationListeners();
          Object results[] = new Object[listeners.length];
          boolean ok = true;
          for (int i = 0; i < results.length; i++) {
              if (getLogger().isDebugEnabled())
                  getLogger().debug(" Configuring event listener class '" +
                      listeners[i] + "'");
              try {
                  String listener = listeners[i];
                  results[i] = getInstanceManager().newInstance(listener);
              } catch (Throwable t) {
                  t = ExceptionUtils.unwrapInvocationTargetException(t);
                  ExceptionUtils.handleThrowable(t);
                  getLogger().error(sm.getString(
                          "standardContext.applicationListener", listeners[i]), t);
                  ok = false;
              }
          }
          if (!ok) {
              getLogger().error(sm.getString("standardContext.applicationSkipped"));
              return false;
          }
  
          // Sort listeners in two arrays
          List<Object> eventListeners = new ArrayList<>();
          List<Object> lifecycleListeners = new ArrayList<>();
          for (int i = 0; i < results.length; i++) {
              if ((results[i] instanceof ServletContextAttributeListener)
                  || (results[i] instanceof ServletRequestAttributeListener)
                  || (results[i] instanceof ServletRequestListener)
                  || (results[i] instanceof HttpSessionIdListener)
                  || (results[i] instanceof HttpSessionAttributeListener)) {
                  eventListeners.add(results[i]);
              }
              if ((results[i] instanceof ServletContextListener)
                  || (results[i] instanceof HttpSessionListener)) {
                  lifecycleListeners.add(results[i]);
              }
          }
  
          // Listener instances may have been added directly to this Context by
          // ServletContextInitializers and other code via the pluggability APIs.
          // Put them these listeners after the ones defined in web.xml and/or
          // annotations then overwrite the list of instances with the new, full
          // list.
          for (Object eventListener: getApplicationEventListeners()) {
              eventListeners.add(eventListener);
          }
          setApplicationEventListeners(eventListeners.toArray());
          for (Object lifecycleListener: getApplicationLifecycleListeners()) {
              lifecycleListeners.add(lifecycleListener);
              if (lifecycleListener instanceof ServletContextListener) {
                  noPluggabilityListeners.add(lifecycleListener);
              }
          }
          setApplicationLifecycleListeners(lifecycleListeners.toArray());
  
          // Send application start events
  
          if (getLogger().isDebugEnabled())
              getLogger().debug("Sending application start events");
  
          // Ensure context is not null
          getServletContext();
          context.setNewServletContextListenerAllowed(false);
  
          Object instances[] = getApplicationLifecycleListeners();
          if (instances == null || instances.length == 0) {
              return ok;
          }
  		//Spring中经典的事件驱动， 创建一个event
  		//1.1构建一个事件 内置Context成员变量
          ServletContextEvent event = new ServletContextEvent(getServletContext());
          ServletContextEvent tldEvent = null;
          if (noPluggabilityListeners.size() > 0) {
              noPluggabilityServletContext = new NoPluggabilityServletContext(getServletContext());
              tldEvent = new ServletContextEvent(noPluggabilityServletContext);
          }
          //1.2这里遍历循环 加载所有的继承自ServletContextListener
          for (int i = 0; i < instances.length; i++) {
              if (!(instances[i] instanceof ServletContextListener))
                  continue;
              ServletContextListener listener =
                  (ServletContextListener) instances[i];
              try {
                  fireContainerEvent("beforeContextInitialized", listener);
                  if (noPluggabilityListeners.contains(listener)) {
                  	**//注意这里要加载tldEvent**
                      listener.contextInitialized(tldEvent);
                  } else {
                      listener.contextInitialized(event);
                  }
                  fireContainerEvent("afterContextInitialized", listener);
              } catch (Throwable t) {
                  ExceptionUtils.handleThrowable(t);
                  fireContainerEvent("afterContextInitialized", listener);
                  getLogger().error
                      (sm.getString("standardContext.listenerStart",
                                    instances[i].getClass().getName()), t);
                  ok = false;
              }
          }
          return ok;
  
      }
  
  ```

  2.那首先加载 WebRootContextListener

  ```
  ...
  public class WebAppRootListener implements ServletContextListener {
  
  	@Override
  	public void contextInitialized(ServletContextEvent event) {
  		WebUtils.setWebAppRootSystemProperty(event.getServletContext());
  	}
  ...
  ```

  ```
  	public static void setWebAppRootSystemProperty(ServletContext servletContext) throws IllegalStateException {
  		Assert.notNull(servletContext, "ServletContext must not be null");
  		//获得context的根目录
  		String root = servletContext.getRealPath("/");
  		if (root == null) {
  			throw new IllegalStateException(
  				"Cannot set web app root system property when WAR file is not expanded");
  		}
  		//WEB_APP_ROOT_KEY_PARAM=webAppRootKey 就是在 web.xml文件中定义的webAppRootKey
  		String param = servletContext.getInitParameter(WEB_APP_ROOT_KEY_PARAM);
  		//
  		String key = (param != null ? param : DEFAULT_WEB_APP_ROOT_KEY);
  		String oldValue = System.getProperty(key);
  		if (oldValue != null && !StringUtils.pathEquals(oldValue, root)) {
  			throw new IllegalStateException(
  				"Web app root system property already set to different value: '" +
  				key + "' = [" + oldValue + "] instead of [" + root + "] - " +
  				"Choose unique values for the 'webAppRootKey' context-param in your web.xml files!");
  		}
  		//设置系统环境变量
  		System.setProperty(key, root);
  		servletContext.log("Set web app root system property: '" + key + "' = [" + root + "]");
  	}
  ```

  3.**加载Spring的ContextLoaderListener**

  ```
  public class ContextLoaderListener extends ContextLoader implements ServletContextListener {
  ...
  	@Override
  	public void contextInitialized(ServletContextEvent event) {
  		//3.1初始化web容器对象
  		initWebApplicationContext(event.getServletContext());
  	}
  ...
  
  ```

  3.1跳转到ContextLoader进行加载上下文环境

  ```
  public class ContextLoader {
  ...
  	static {
  		// Load default strategy implementations from properties file.
  		// This is currently strictly internal and not meant to be customized
  		// by application developers.
  		try {
  			ClassPathResource resource = new ClassPathResource(DEFAULT_STRATEGIES_PATH, ContextLoader.class);
  			defaultStrategies = PropertiesLoaderUtils.loadProperties(resource);
  		}
  		catch (IOException ex) {
  			throw new IllegalStateException("Could not load 'ContextLoader.properties': " + ex.getMessage());
  		}
  	}
  	...
  	/**
  	 * Initialize Spring's web application context for the given servlet context,
  	 * using the application context provided at construction time, or creating a new one
  	 * according to the "{@link #CONTEXT_CLASS_PARAM contextClass}" and
  	 * "{@link #CONFIG_LOCATION_PARAM contextConfigLocation}" context-params.
  	 * @param servletContext current servlet context
  	 * @return the new WebApplicationContext
  	 * @see #ContextLoader(WebApplicationContext)
  	 * @see #CONTEXT_CLASS_PARAM
  	 * @see #CONFIG_LOCATION_PARAM
  	 */
  	public WebApplicationContext initWebApplicationContext(ServletContext servletContext) {
  		if (servletContext.getAttribute(WebApplicationContext.ROOT_WEB_APPLICATION_CONTEXT_ATTRIBUTE) != null) {
  			throw new IllegalStateException(
  					"Cannot initialize context because there is already a root application context present - " +
  					"check whether you have multiple ContextLoader* definitions in your web.xml!");
  		}
  
  		Log logger = LogFactory.getLog(ContextLoader.class);
  		servletContext.log("Initializing Spring root WebApplicationContext");
  		if (logger.isInfoEnabled()) {
  			logger.info("Root WebApplicationContext: initialization started");
  		}
  		//记录初始化时间
  		long startTime = System.currentTimeMillis();
  
  		try {
  			// Store context in local instance variable, to guarantee that
  			// it is available on ServletContext shutdown.
  			if (this.context == null) {
  			//3.1.1如果上下文环境对象为null，那么实例化一个
  				this.context = createWebApplicationContext(servletContext);
  			}
  			if (this.context instanceof ConfigurableWebApplicationContext) {
  				ConfigurableWebApplicationContext cwac = (ConfigurableWebApplicationContext) this.context;
  				if (!cwac.isActive()) {
  					// The context has not yet been refreshed -> provide services such as
  					// setting the parent context, setting the application context id, etc
  					if (cwac.getParent() == null) {
  						// The context instance was injected without an explicit parent ->
  						// determine parent for root web application context, if any.
  						//加载父容器
  						ApplicationContext parent = loadParentContext(servletContext);
  						cwac.setParent(parent);
  					}
  					//3.1.2配置并刷新web上下文环境
  					configureAndRefreshWebApplicationContext(cwac, servletContext);
  				}
  			}
  			servletContext.setAttribute(WebApplicationContext.ROOT_WEB_APPLICATION_CONTEXT_ATTRIBUTE, this.context);
  
  			ClassLoader ccl = Thread.currentThread().getContextClassLoader();
  			if (ccl == ContextLoader.class.getClassLoader()) {
  				currentContext = this.context;
  			}
  			else if (ccl != null) {
  				currentContextPerThread.put(ccl, this.context);
  			}
  
  			if (logger.isDebugEnabled()) {
  				logger.debug("Published root WebApplicationContext as ServletContext attribute with name [" +
  						WebApplicationContext.ROOT_WEB_APPLICATION_CONTEXT_ATTRIBUTE + "]");
  			}
  			if (logger.isInfoEnabled()) {
  				long elapsedTime = System.currentTimeMillis() - startTime;
  				logger.info("Root WebApplicationContext: initialization completed in " + elapsedTime + " ms");
  			}
  
  			return this.context;
  		}
  		catch (RuntimeException ex) {
  			logger.error("Context initialization failed", ex);
  			servletContext.setAttribute(WebApplicationContext.ROOT_WEB_APPLICATION_CONTEXT_ATTRIBUTE, ex);
  			throw ex;
  		}
  		catch (Error err) {
  			logger.error("Context initialization failed", err);
  			servletContext.setAttribute(WebApplicationContext.ROOT_WEB_APPLICATION_CONTEXT_ATTRIBUTE, err);
  			throw err;
  		}
  	}
  	
  ```

  3.1.1实例化上下文环境对象

  ```
  protected WebApplicationContext createWebApplicationContext(ServletContext sc) {
  //拿到class对象
  Class<?> contextClass = determineContextClass(sc);
  		if (!ConfigurableWebApplicationContext.class.isAssignableFrom(contextClass)) {
  			throw new ApplicationContextException("Custom context class [" + contextClass.getName() +
  					"] is not of type [" + ConfigurableWebApplicationContext.class.getName() + "]");
  		}
  		//反射生成对象 ConfigurableWebApplicationContext
  		return ( ConfigurableWebApplicationContext) BeanUtils.instantiateClass(contextClass);
  	}
  ```

  加载父类容器

  ```
  public class ContextLoader {
  ...
  	/**
  	 * Template method with default implementation (which may be overridden by a
  	 * subclass), to load or obtain an ApplicationContext instance which will be
  	 * used as the parent context of the root WebApplicationContext. If the
  	 * return value from the method is null, no parent context is set.\
  	 // 注意下面的话 意思是允许加载多web容器
  	 * <p>The main reason to load a parent context here is to allow multiple root
  	 * web application contexts to all be children of a shared EAR context, or
  	 * alternately to also share the same parent context that is visible to
  	 * EJBs. For pure web applications, there is usually no need to worry about
  	 * having a parent context to the root web application context.
  	 * <p>The default implementation uses
  	 * {@link org.springframework.context.access.ContextSingletonBeanFactoryLocator},
  	 * configured via {@link #LOCATOR_FACTORY_SELECTOR_PARAM} and
  	 * {@link #LOCATOR_FACTORY_KEY_PARAM}, to load a parent context
  	 * which will be shared by all other users of ContextsingletonBeanFactoryLocator
  	 * which also use the same configuration parameters.
  	 * @param servletContext current servlet context
  	 * @return the parent application context, or {@code null} if none
  	 * @see org.springframework.context.access.ContextSingletonBeanFactoryLocator
  	 */
  	protected ApplicationContext loadParentContext(ServletContext servletContext) {
  		ApplicationContext parentContext = null;
  		String locatorFactorySelector = servletContext.getInitParameter(LOCATOR_FACTORY_SELECTOR_PARAM);
  		String parentContextKey = servletContext.getInitParameter(LOCATOR_FACTORY_KEY_PARAM);
  
  		if (parentContextKey != null) {
  			// locatorFactorySelector may be null, indicating the default "classpath*:beanRefContext.xml"
  			BeanFactoryLocator locator = ContextSingletonBeanFactoryLocator.getInstance(locatorFactorySelector);
  			Log logger = LogFactory.getLog(ContextLoader.class);
  			if (logger.isDebugEnabled()) {
  				logger.debug("Getting parent context definition: using parent context key of '" +
  						parentContextKey + "' with BeanFactoryLocator");
  			}
  			this.parentContextRef = locator.useBeanFactory(parentContextKey);
  			parentContext = (ApplicationContext) this.parentContextRef.getFactory();
  		}
  
  		return parentContext;
  	}
  ...
  ```

  3.1.2配置并刷新web上下文环境

  ```
  public class ContextLoader {
  ...
  protected void configureAndRefreshWebApplicationContext(ConfigurableWebApplicationContext wac, ServletContext sc) {
  		if (ObjectUtils.identityToString(wac).equals(wac.getId())) {
  			// The application context id is still set to its original default value
  			// -> assign a more useful id based on available information
  			String idParam = sc.getInitParameter(CONTEXT_ID_PARAM);
  			if (idParam != null) {
  				wac.setId(idParam);
  			}
  			else {
  				// Generate default id...
  				wac.setId(ConfigurableWebApplicationContext.APPLICATION_CONTEXT_ID_PREFIX +
  						ObjectUtils.getDisplayString(sc.getContextPath()));
  			}
  		}
  		//设置上下文环境对象
  		wac.setServletContext(sc);
  		//获取初始化变量
  		String configLocationParam = sc.getInitParameter(CONFIG_LOCATION_PARAM);
  		if (configLocationParam != null) {
  		//设置初始化变量到web容器中
  		//设置配置location到web容器这里的configLocationParam 就是web.xml中配置的		     //classpath:xxxxxx.xml
  			wac.setConfigLocation(configLocationParam);
  		}
  
  		// The wac environment's #initPropertySources will be called in any case when the context
  		// is refreshed; do it eagerly here to ensure servlet property sources are in place for
  		// use in any post-processing or initialization that occurs below prior to #refresh
  		//获取环境对象
  		ConfigurableEnvironment env = wac.getEnvironment();
  		//3.1.2.1初始化一些配置properties
      	if (env instanceof ConfigurableWebEnvironment) {
  			((ConfigurableWebEnvironment) env).initPropertySources(sc, null);
  		}
  		//3.1.2.2装配上下文环境对象
  		customizeContext(sc, wac);
  		//3.1.2.3刷新上下文环境
  		wac.refresh();
  	}
  	...
  	}
  ```

  3.1.2.1初始化一些配置properties

  ```
  public class StandardServletEnvironment extends StandardEnvironment implements ConfigurableWebEnvironment {
  ...
  	@Override
  	public void initPropertySources(ServletContext servletContext, ServletConfig servletConfig) {
  		WebApplicationContextUtils.initServletPropertySources(getPropertySources(), servletContext, servletConfig);
  	}
  ...
  }
  public abstract class WebApplicationContextUtils {
  ...
  	/**
  	 * Replace {@code Servlet}-based {@link StubPropertySource stub property sources} with
  	 * actual instances populated with the given {@code servletContext} and
  	 * {@code servletConfig} objects.
  	 * <p>This method is idempotent with respect to the fact it may be called any number
  	 * of times but will perform replacement of stub property sources with their
  	 * corresponding actual property sources once and only once.
  	 * @param propertySources the {@link MutablePropertySources} to initialize (must not
  	 * be {@code null})
  	 * @param servletContext the current {@link ServletContext} (ignored if {@code null}
  	 * or if the {@link StandardServletEnvironment#SERVLET_CONTEXT_PROPERTY_SOURCE_NAME
  	 * servlet context property source} has already been initialized)
  	 * @param servletConfig the current {@link ServletConfig} (ignored if {@code null}
  	 * or if the {@link StandardServletEnvironment#SERVLET_CONFIG_PROPERTY_SOURCE_NAME
  	 * servlet config property source} has already been initialized)
  	 * @see org.springframework.core.env.PropertySource.StubPropertySource
  	 * @see org.springframework.core.env.ConfigurableEnvironment#getPropertySources()
  	 */
  	public static void initServletPropertySources(
  			MutablePropertySources propertySources, ServletContext servletContext, ServletConfig servletConfig) {
  
  		Assert.notNull(propertySources, "'propertySources' must not be null");
  		//propertySources包含这些初始化配置[servletConfigInitParams,servletContextInitParams,jndiProperties,systemProperties,systemEnvironment]
  		//SERVLET_CONTEXT_PROPERTY_SOURCE_NAME=servletContextInitParams
  		//
  		if (servletContext != null && propertySources.contains(StandardServletEnvironment.SERVLET_CONTEXT_PROPERTY_SOURCE_NAME) &&
  				propertySources.get(StandardServletEnvironment.SERVLET_CONTEXT_PROPERTY_SOURCE_NAME) instanceof StubPropertySource) {
  			propertySources.replace(StandardServletEnvironment.SERVLET_CONTEXT_PROPERTY_SOURCE_NAME,
  					new ServletContextPropertySource(StandardServletEnvironment.SERVLET_CONTEXT_PROPERTY_SOURCE_NAME, servletContext));
  		}
  		if (servletConfig != null && propertySources.contains(StandardServletEnvironment.SERVLET_CONFIG_PROPERTY_SOURCE_NAME) &&
  				propertySources.get(StandardServletEnvironment.SERVLET_CONFIG_PROPERTY_SOURCE_NAME) instanceof StubPropertySource) {
  			propertySources.replace(StandardServletEnvironment.SERVLET_CONFIG_PROPERTY_SOURCE_NAME,
  					new ServletConfigPropertySource(StandardServletEnvironment.SERVLET_CONFIG_PROPERTY_SOURCE_NAME, servletConfig));
  		}
  	}
  ...
  }
  ```

  3.1.2.2装配上下文环境对象

  ```
  public class ContextLoader {
  ...
  /**
  	 * Customize the {@link ConfigurableWebApplicationContext} created by this
  	 * ContextLoader after config locations have been supplied to the context
  	 * but before the context is <em>refreshed</em>.
  	 * <p>The default implementation {@linkplain #determineContextInitializerClasses(ServletContext)
  	 * determines} what (if any) context initializer classes have been specified through
  	 * {@linkplain #CONTEXT_INITIALIZER_CLASSES_PARAM context init parameters} and
  	 * {@linkplain ApplicationContextInitializer#initialize invokes each} with the
  	 * given web application context.
  	 * <p>Any {@code ApplicationContextInitializers} implementing
  	 * {@link org.springframework.core.Ordered Ordered} or marked with @{@link
  	 * org.springframework.core.annotation.Order Order} will be sorted appropriately.
  	 * @param sc the current servlet context
  	 * @param wac the newly created application context
  	 * @see #CONTEXT_INITIALIZER_CLASSES_PARAM
  	 * @see ApplicationContextInitializer#initialize(ConfigurableApplicationContext)
  	 */
  	protected void customizeContext(ServletContext sc, ConfigurableWebApplicationContext wac) {
  		List<Class<ApplicationContextInitializer<ConfigurableApplicationContext>>> initializerClasses =
  				determineContextInitializerClasses(sc);
  
  		for (Class<ApplicationContextInitializer<ConfigurableApplicationContext>> initializerClass : initializerClasses) {
  			Class<?> initializerContextClass =
  					GenericTypeResolver.resolveTypeArgument(initializerClass, ApplicationContextInitializer.class);
  			if (initializerContextClass != null && !initializerContextClass.isInstance(wac)) {
  				throw new ApplicationContextException(String.format(
  						"Could not apply context initializer [%s] since its generic parameter [%s] " +
  						"is not assignable from the type of application context used by this " +
  						"context loader: [%s]", initializerClass.getName(), initializerContextClass.getName(),
  						wac.getClass().getName()));
  			}
  			this.contextInitializers.add(BeanUtils.instantiateClass(initializerClass));
  		}
  
  		AnnotationAwareOrderComparator.sort(this.contextInitializers);
  		for (ApplicationContextInitializer<ConfigurableApplicationContext> initializer : this.contextInitializers) {
  			//3.1.2.2.1遍历所有的上下文初始化器，进行初始化容器
  			initializer.initialize(wac);
  		}
  	}
  ...
  }
  ```

  3.1.2.2.1遍历所有的上下文初始化器，进行初始化容器

  ```
  public interface ApplicationContextInitializer<C extends ConfigurableApplicationContext> {
  
  	/**
  	 * Initialize the given application context.
  	 * @param applicationContext the application to configure
  	 */
  	void initialize(C applicationContext);
  
  }
  ```

  **3.1.2.3刷新上下文环境,这个很重要的一个方法,Spring的bean解析就在这里啊**

  ```
  public abstract class AbstractApplicationContext extends DefaultResourceLoader
  		implements ConfigurableApplicationContext, DisposableBean {
  		...
  		@Override
  	public void refresh() throws BeansException, IllegalStateException {
  		synchronized (this.startupShutdownMonitor) {
  			// Prepare this context for refreshing.
  			//3.1.2.3.1预备刷新前的一些准备工作
  			prepareRefresh();
  
  			// Tell the subclass to refresh the internal bean factory.
  			//3.1.2.3.2告诉父类刷新内置bean工厂
  			ConfigurableListableBeanFactory beanFactory = obtainFreshBeanFactory();
  
  			// Prepare the bean factory for use in this context.
  			//3.1.2.3.3创建bean工厂 主要对bean工厂进行特征设置
  			 prepareBeanFactory(beanFactory);
  
  			try {
  				// Allows post-processing of the bean factory in context subclasses.
  				//此方法允许子类在所有的 bean 尚未初始化之前注册 BeanPostProcessor
  				postProcessBeanFactory(beanFactory);
  
  				// Invoke factory processors registered as beans in the context.
  				//BeanFactoryPostProcessor 接口允许我们在 bean 正是初始化之前改变其值。此接口只				//有一个方法
  				invokeBeanFactoryPostProcessors(beanFactory);
  
  				// Register bean processors that intercept bean creation.
  				//BeanPostProcessors注册 在BeanDefinitions 中查找BeanPostProcessor保存一个				//list中
                  registerBeanPostProcessors(beanFactory);
  				//支持国际化
  				// Initialize message source for this context.
  				initMessageSource();
  				//加载事件驱动ApplicationEventMulticaster来管理事件驱动三元素 可以看下具体实现
  				//有addApplicationListener multicastEvent 传播嘛
  				//ApplicationEventPublisher是将事件委托给ApplicationEventMulticaster来处理的
  				// Initialize event multicaster for this context.
  				initApplicationEventMulticaster();
  				//初始化其他的特殊的bean 没有实现方法 默认走的空实现
  				// Initialize other special beans in specific context subclasses.
  				onRefresh();
  
  				// Check for listener beans and register them.
  				//这里注册监听器了 结合上面的事件广播类进行
  				registerListeners();
  				//在工厂中进行单例beans的注册配置
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
  	...
  		}
  ```

  **finishRefresh**

  ```
  	protected void finishRefresh() {
  		// Initialize lifecycle processor for this context.
  		initLifecycleProcessor();
  
  		// Propagate refresh to lifecycle processor first.
  		getLifecycleProcessor().onRefresh();
  
  		// Publish the final event.
  		//最后发布上下文构建成功事件
  		publishEvent(new ContextRefreshedEvent(this));
  		//注册下上下文环境。
  		// Participate in LiveBeansView MBean, if active.
  		LiveBeansView.registerApplicationContext(this);
  	}
  ```

  

  finishBeanFactoryInitialization在工厂中进行单例beans的注册配置**

  * ConversionService bean类型转换的一个服务bean
  * StringValueResolver 用于解析注解值
  * LoadTimeWeaverAware 实现了此接口的 bean 可以得到 LoadTimeWeaver，此处仅仅初始化 
  * preInstantiateSingletons  

  ```
  	/**
  	 * Finish the initialization of this context's bean factory,
  	 * initializing all remaining singleton beans.
  	 */
  	protected void finishBeanFactoryInitialization(ConfigurableListableBeanFactory beanFactory) {
  		// Initialize conversion service for this context.
  		if (beanFactory.containsBean(CONVERSION_SERVICE_BEAN_NAME) &&
  				beanFactory.isTypeMatch(CONVERSION_SERVICE_BEAN_NAME, ConversionService.class)) {
  			beanFactory.setConversionService(
  					beanFactory.getBean(CONVERSION_SERVICE_BEAN_NAME, ConversionService.class));
  		}
  
  		// Register a default embedded value resolver if no bean post-processor
  		// (such as a PropertyPlaceholderConfigurer bean) registered any before:
  		// at this point, primarily for resolution in annotation attribute values.
  		if (!beanFactory.hasEmbeddedValueResolver()) {
  			beanFactory.addEmbeddedValueResolver(new StringValueResolver() {
  				@Override
  				public String resolveStringValue(String strVal) {
  					return getEnvironment().resolvePlaceholders(strVal);
  				}
  			});
  		}
  
  		// Initialize LoadTimeWeaverAware beans early to allow for registering their transformers early.
  		String[] weaverAwareNames = beanFactory.getBeanNamesForType(LoadTimeWeaverAware.class, false, false);
  		for (String weaverAwareName : weaverAwareNames) {
  			getBean(weaverAwareName);
  		}
  
  		// Stop using the temporary ClassLoader for type matching.
  		beanFactory.setTempClassLoader(null);
  
  		// Allow for caching all bean definition metadata, not expecting further changes.
  		beanFactory.freezeConfiguration();
  
  		// Instantiate all remaining (non-lazy-init) singletons.
  		beanFactory.preInstantiateSingletons();
  	}
  ```

  **preInstantiateSingletons  进行单例bean的预初始化**

  首先进行单例类的初始化，其中如果 bean 是 FactoryBean 类型 (注意，只定义了 factory-method 属性的普通 bean 并不是 FactoryBean)，并且还是 SmartFactoryBean 类型，那么需要判断是否需要 eagerInit(isEagerInit 是此接口定义的方法)。 

  ```
  @Override
  	public void preInstantiateSingletons() throws BeansException {
  		if (this.logger.isDebugEnabled()) {
  			this.logger.debug("Pre-instantiating singletons in " + this);
  		}
  
  		// Iterate over a copy to allow for init methods which in turn register new bean definitions.
  		// While this may not be part of the regular factory bootstrap, it does otherwise work fine.
  		List<String> beanNames = new ArrayList<String>(this.beanDefinitionNames);
  
  		// Trigger initialization of all non-lazy singleton beans...
  		for (String beanName : beanNames) {
  			RootBeanDefinition bd = getMergedLocalBeanDefinition(beanName);
  			if (!bd.isAbstract() && bd.isSingleton() && !bd.isLazyInit()) {
  				if (isFactoryBean(beanName)) {
  					final FactoryBean<?> factory = (FactoryBean<?>) getBean(FACTORY_BEAN_PREFIX + beanName);
  					boolean isEagerInit;
  					if (System.getSecurityManager() != null && factory instanceof SmartFactoryBean) {
  						isEagerInit = AccessController.doPrivileged(new PrivilegedAction<Boolean>() {
  							@Override
  							public Boolean run() {
  								return ((SmartFactoryBean<?>) factory).isEagerInit();
  							}
  						}, getAccessControlContext());
  					}
  					else {
  						isEagerInit = (factory instanceof SmartFactoryBean &&
  								((SmartFactoryBean<?>) factory).isEagerInit());
  					}
  					if (isEagerInit) {
  						getBean(beanName);
  					}
  				}
  				else {
  					getBean(beanName);
  				}
  			}
  		}
  
  		// Trigger post-initialization callback for all applicable beans...
  		for (String beanName : beanNames) {
  			Object singletonInstance = getSingleton(beanName);
  			if (singletonInstance instanceof SmartInitializingSingleton) {
  				final SmartInitializingSingleton smartSingleton = (SmartInitializingSingleton) singletonInstance;
  				if (System.getSecurityManager() != null) {
  					AccessController.doPrivileged(new PrivilegedAction<Object>() {
  						@Override
  						public Object run() {
  							smartSingleton.afterSingletonsInstantiated();
  							return null;
  						}
  					}, getAccessControlContext());
  				}
  				else {
  					smartSingleton.afterSingletonsInstantiated();
  				}
  			}
  		}
  	}
  
  ```

  

  **registerListeners，注册监听器**

  需要指出的是监听器列表在ContextLoader类中的``initWebApplicationContext``方法中的``loadParentContext``中的``useBeanFactory``中已经添加进入了

  ````
  public class EventListenerMethodProcessor implements SmartInitializingSingleton, ApplicationContextAware {
  ...
  	protected void processBean(final List<EventListenerFactory> factories, final String beanName, final Class<?> targetType) {
  		if (!this.nonAnnotatedClasses.contains(targetType)) {
  			Map<Method, EventListener> annotatedMethods = null;
  			try {
  				annotatedMethods = MethodIntrospector.selectMethods(targetType,
  						new MethodIntrospector.MetadataLookup<EventListener>() {
  							@Override
  							public EventListener inspect(Method method) {
  								return AnnotatedElementUtils.findMergedAnnotation(method, EventListener.class);
  							}
  						});
  			}
  			catch (Throwable ex) {
  				// An unresolvable type in a method signature, probably from a lazy bean - let's ignore it.
  				if (logger.isDebugEnabled()) {
  					logger.debug("Could not resolve methods for bean with name '" + beanName + "'", ex);
  				}
  			}
  			if (CollectionUtils.isEmpty(annotatedMethods)) {
  				this.nonAnnotatedClasses.add(targetType);
  				if (logger.isTraceEnabled()) {
  					logger.trace("No @EventListener annotations found on bean class: " + targetType.getName());
  				}
  			}
  			else {
  				// Non-empty set of methods
  				for (Method method : annotatedMethods.keySet()) {
  					for (EventListenerFactory factory : factories) {
  						if (factory.supportsMethod(method)) {
  							Method methodToUse = AopUtils.selectInvocableMethod(
  									method, this.applicationContext.getType(beanName));
  							ApplicationListener<?> applicationListener =
  									factory.createApplicationListener(beanName, targetType, methodToUse);
  							if (applicationListener instanceof ApplicationListenerMethodAdapter) {
  								((ApplicationListenerMethodAdapter) applicationListener)
  										.init(this.applicationContext, this.evaluator);
  							}
  							//注意这里添加applicationListener到环境中了							this.applicationContext.addApplicationListener(applicationListener);
  							break;
  						}
  					}
  				}
  				if (logger.isDebugEnabled()) {
  					logger.debug(annotatedMethods.size() + " @EventListener methods processed on bean '" +
  							beanName + "': " + annotatedMethods);
  				}
  			}
  		}
  	}
  ...
  }
  ````

  下面是上下文环境中的监听器注册到工厂

  ```
  	/**
  	 * Add beans that implement ApplicationListener as listeners.
  	 * Doesn't affect other listeners, which can be added without being beans.
  	 */
  	protected void registerListeners() {
  		// Register statically specified listeners first.
  		//获得到之前加载的监听器列表 监听器列表在configureAndRefreshWebApplicationContext的时候就
  		for (ApplicationListener<?> listener : getApplicationListeners()) {
  			getApplicationEventMulticaster().addApplicationListener(listener);
  		}
  
  		// Do not initialize FactoryBeans here: We need to leave all regular beans
  		// uninitialized to let post-processors apply to them!
  		String[] listenerBeanNames = getBeanNamesForType(ApplicationListener.class, true, false);
  		for (String listenerBeanName : listenerBeanNames) {
  			getApplicationEventMulticaster().addApplicationListenerBean(listenerBeanName);
  		}
  
  		// Publish early application events now that we finally have a multicaster...
  		Set<ApplicationEvent> earlyEventsToProcess = this.earlyApplicationEvents;
  		this.earlyApplicationEvents = null;
  		if (earlyEventsToProcess != null) {
  			for (ApplicationEvent earlyEvent : earlyEventsToProcess) {
  				getApplicationEventMulticaster().multicastEvent(earlyEvent);
  			}
  		}
  	}
  ```

  

  **initApplicationEventMulticaster**

  ```
  	/**
  	 * Initialize the ApplicationEventMulticaster.
  	 * Uses SimpleApplicationEventMulticaster if none defined in the context.
  	 * @see org.springframework.context.event.SimpleApplicationEventMulticaster
  	 */
  	protected void initApplicationEventMulticaster() {
  		ConfigurableListableBeanFactory beanFactory = getBeanFactory();
  		//看工厂中有没有ApplicationEventMulticaster这个类 如果有的话 就取出来
  		if (beanFactory.containsLocalBean(APPLICATION_EVENT_MULTICASTER_BEAN_NAME)) {
  			this.applicationEventMulticaster =
  					beanFactory.getBean(APPLICATION_EVENT_MULTICASTER_BEAN_NAME, ApplicationEventMulticaster.class);
  			if (logger.isDebugEnabled()) {
  				logger.debug("Using ApplicationEventMulticaster [" + this.applicationEventMulticaster + "]");
  			}
  		}
  		else {
  		//如果没有这个类 那么就创建一个 然后作为单例对象放置到工厂中。工厂是在上下文环境对象中包含着的
  			this.applicationEventMulticaster = new SimpleApplicationEventMulticaster(beanFactory);
  			beanFactory.registerSingleton(APPLICATION_EVENT_MULTICASTER_BEAN_NAME, this.applicationEventMulticaster);
  			if (logger.isDebugEnabled()) {
  				logger.debug("Unable to locate ApplicationEventMulticaster with name '" +
  						APPLICATION_EVENT_MULTICASTER_BEAN_NAME +
  						"': using default [" + this.applicationEventMulticaster + "]");
  			}
  		}
  	}
  ```

  

  **3.1.2.3.1预备刷新前的一些准备工作**

  ```
  /**
  	 * Prepare this context for refreshing, setting its startup date and
  	 * active flag as well as performing any initialization of property sources.
  	 */
  	protected void prepareRefresh() {
  		this.startupDate = System.currentTimeMillis();
  		this.closed.set(false);
  		this.active.set(true);
  
  		if (logger.isInfoEnabled()) {
  			logger.info("Refreshing " + this);
  		}
  
  		// Initialize any placeholder property sources in the context environment
  		//3.1.2.3.1.1初始化一些配置文件
  		initPropertySources();
  
  		// Validate that all properties marked as required are resolvable
  		// see ConfigurablePropertyResolver#setRequiredProperties
  		//3.1.2.3.2校验必须的配置
  		getEnvironment().validateRequiredProperties();
  
  		// Allow for the collection of early ApplicationEvents,
  		// to be published once the multicaster is available...
  		this.earlyApplicationEvents = new LinkedHashSet<ApplicationEvent>();
  	}
  ```

  **3.1.2.3.2告诉父类刷新内置bean工厂**

  ```
  	/**
  	 * Tell the subclass to refresh the internal bean factory.
  	 * @return the fresh BeanFactory instance
  	 * @see #refreshBeanFactory()
  	 * @see #getBeanFactory()
  	 */
  	protected ConfigurableListableBeanFactory obtainFreshBeanFactory() {
  		//3.1.2.3.2.1在这里refresh bean工厂
  		refreshBeanFactory();
  		ConfigurableListableBeanFactory beanFactory = getBeanFactory();
  		if (logger.isDebugEnabled()) {
  			logger.debug("Bean factory for " + getDisplayName() + ": " + beanFactory);
  		}
  		return beanFactory;
  	}
  ```

  **3.1.2.3.2.1在这里refresh bean工厂**

  ```
  	@Override
  	protected final void refreshBeanFactory() throws BeansException {
  	//如果存在工厂了 就销毁掉 然后关闭工厂 这样就刷新了
  		if (hasBeanFactory()) {
  			destroyBeans();
  			closeBeanFactory();
  		}
  		try {
  			//创建bean可装配到list中的工厂
  			DefaultListableBeanFactory beanFactory = createBeanFactory();
  			beanFactory.setSerializationId(getId());
  			//3.1.2.3.2.1.1给与子类自由定制bean容器的机会
  			customizeBeanFactory(beanFactory);
  			//3.1.2.3.2.1.2核心的bean加载方法。
  			loadBeanDefinitions(beanFactory);
  			synchronized (this.beanFactoryMonitor) {
  				this.beanFactory = beanFactory;
  			}
  		}
  		catch (IOException ex) {
  			throw new ApplicationContextException("I/O error parsing bean definition source for " + getDisplayName(), ex);
  		}
  	}
  ```

  **3.1.2.3.2.1.1给与子类自由定制bean容器的机会**

  ```
  /**
  	 * Customize the internal bean factory used by this context.
  	 * Called for each {@link #refresh()} attempt.
  	 * <p>The default implementation applies this context's
  	 * {@linkplain #setAllowBeanDefinitionOverriding "allowBeanDefinitionOverriding"}
  	 * and {@linkplain #setAllowCircularReferences "allowCircularReferences"} settings,
  	 * if specified. Can be overridden in subclasses to customize any of
  	 * {@link DefaultListableBeanFactory}'s settings.
  	 * @param beanFactory the newly created bean factory for this context
  	 * @see DefaultListableBeanFactory#setAllowBeanDefinitionOverriding
  	 * @see DefaultListableBeanFactory#setAllowCircularReferences
  	 * @see DefaultListableBeanFactory#setAllowRawInjectionDespiteWrapping
  	 * @see DefaultListableBeanFactory#setAllowEagerClassLoading
  	 */
  	protected void customizeBeanFactory(DefaultListableBeanFactory beanFactory) {
  		if (this.allowBeanDefinitionOverriding != null) {
  			//默认false，不允许覆盖beanFactory.setAllowBeanDefinitionOverriding(this.allowBeanDefinitionOverriding);
  		}
  		if (this.allowCircularReferences != null) {
  		//默认false,不允许循环引用
  			beanFactory.setAllowCircularReferences(this.allowCircularReferences);
  		}
  	}
  ```

  **3.1.2.3.2.1.2核心的bean加载方法。这个方法很重要**

  ```
  	/**
  	 * Loads the bean definitions via an XmlBeanDefinitionReader.
  	 * @see org.springframework.beans.factory.xml.XmlBeanDefinitionReader
  	 * @see #initBeanDefinitionReader
  	 * @see #loadBeanDefinitions
  	 */
  	@Override
  	protected void loadBeanDefinitions(DefaultListableBeanFactory beanFactory) throws BeansException, IOException {
  		// Create a new XmlBeanDefinitionReader for the given BeanFactory.
  		//创建一个基于xml的bean声明读取器
  		XmlBeanDefinitionReader beanDefinitionReader = new XmlBeanDefinitionReader(beanFactory);
  
  		// Configure the bean definition reader with this context's
  		// resource loading environment.
  		//设置环境
  		beanDefinitionReader.setEnvironment(getEnvironment());
  		//设置资源加载器
  		beanDefinitionReader.setResourceLoader(this);
  		//设置一个资源entity解析器 这里肯定是xml
  		beanDefinitionReader.setEntityResolver(new ResourceEntityResolver(this));
  
  		// Allow a subclass to provide custom initialization of the reader,
  		// then proceed with actually loading the bean definitions.
  		//3.1.2.3.2.1.2.1初始化bean声明读取器
  		initBeanDefinitionReader(beanDefinitionReader);
  		loadBeanDefinitions(beanDefinitionReader);
  	}
  ```

  **3.1.2.3.2.1.2.1初始化bean声明读取器**

  ```
  	public class XmlWebApplicationContext extends AbstractRefreshableWebApplicationContext {
  	/**
  	 * Load the bean definitions with the given XmlBeanDefinitionReader.
  	 * <p>The lifecycle of the bean factory is handled by the refreshBeanFactory method;
  	 * therefore this method is just supposed to load and/or register bean definitions.
  	 * <p>Delegates to a ResourcePatternResolver for resolving location patterns
  	 * into Resource instances.
  	 * @throws IOException if the required XML document isn't found
  	 * @see #refreshBeanFactory
  	 * @see #getConfigLocations
  	 * @see #getResources
  	 * @see #getResourcePatternResolver
  	 */
  	protected void loadBeanDefinitions(XmlBeanDefinitionReader reader) throws IOException {
  		String[] configLocations = getConfigLocations();
  		if (configLocations != null) {
  			for (String configLocation : configLocations) {
  				//读取具体配置位置然后读取器进行加载跳入下面的方法
  				reader.loadBeanDefinitions(configLocation);
  			}
  		}
  	}
  }	
  	public abstract class AbstractBeanDefinitionReader implements EnvironmentCapable, BeanDefinitionReader {
   ...
  	/**
  	 * Load bean definitions from the specified resource location.
  	 * <p>The location can also be a location pattern, provided that the
  	 * ResourceLoader of this bean definition reader is a ResourcePatternResolver.
  	 * @param location the resource location, to be loaded with the ResourceLoader
  	 * (or ResourcePatternResolver) of this bean definition reader
  	 * @param actualResources a Set to be filled with the actual Resource objects
  	 * that have been resolved during the loading process. May be {@code null}
  	 * to indicate that the caller is not interested in those Resource objects.
  	 * @return the number of bean definitions found
  	 * @throws BeanDefinitionStoreException in case of loading or parsing errors
  	 * @see #getResourceLoader()
  	 * @see #loadBeanDefinitions(org.springframework.core.io.Resource)
  	 * @see #loadBeanDefinitions(org.springframework.core.io.Resource[])
  	 */
  	public int loadBeanDefinitions(String location, @Nullable Set<Resource> actualResources) throws BeanDefinitionStoreException {
  		//开始读取配置了bean的xml文件
  		ResourceLoader resourceLoader = getResourceLoader();
  		//如果资源加载器不存在的话那么就抛出异常
  		if (resourceLoader == null) {
  			throw new BeanDefinitionStoreException(
  					"Cannot import bean definitions from location [" + location + "]: no ResourceLoader available");
  		}
  		//如果是classpath：application*.xml这种pattern的话
  		if (resourceLoader instanceof ResourcePatternResolver) {
  			// Resource pattern matching available.
  			try {
  			//获得资源
  				Resource[] resources = ((ResourcePatternResolver) resourceLoader).getResources(location);
  				//遍历循环 获取所有的资源的总数
  				int loadCount = loadBeanDefinitions(resources);
  				//遍历循环放入actualResources这个set中。
  				if (actualResources != null) {
  					for (Resource resource : resources) {
  						actualResources.add(resource);
  					}
  				}
  				if (logger.isDebugEnabled()) {
  					logger.debug("Loaded " + loadCount + " bean definitions from location pattern [" + location + "]");
  				}
  				return loadCount;
  			}
  			catch (IOException ex) {
  				throw new BeanDefinitionStoreException(
  						"Could not resolve bean definition resource pattern [" + location + "]", ex);
  			}
  		}else {
  			//否则的话只允许绝对路径的资源配置文件，
  			// Can only load single resources by absolute URL.
  			Resource resource = resourceLoader.getResource(location);
  			//获得资源总数
  			int loadCount = loadBeanDefinitions(resource);
  			if (actualResources != null) {
  				//同样加载到set中
  				actualResources.add(resource);
  			}
  			if (logger.isDebugEnabled()) {
  				logger.debug("Loaded " + loadCount + " bean definitions from location [" + location + "]");
  			}
  			return loadCount;
  		}
  	}
  	...
   }
  ```

  **getResources(location)**

  ````
  public class PathMatchingResourcePatternResolver implements ResourcePatternResolver {
  	...
  	@Override
  	public Resource[] getResources(String locationPattern) throws IOException {
  		Assert.notNull(locationPattern, "Location pattern must not be null");
  		//CLASSPATH_ALL_URL_PREFIX=classpath*:
  		//如果是以classpath*:开头的
  		if (locationPattern.startsWith(CLASSPATH_ALL_URL_PREFIX)) {
  			// a class path resource (multiple resources for same name possible)
  			//这里的注释的意思是多个资源的partern的话
  			if (getPathMatcher().isPattern(locationPattern.substring(CLASSPATH_ALL_URL_PREFIX.length()))) {
  				// a class path resource pattern
  				//获取匹配的资源列表 返回资源列表
  				return findPathMatchingResources(locationPattern);
  			}
  			else {
  				// all class path resources with the given name
  				return 
  //获取所有的classpath下的资源			findAllClassPathResources(locationPattern.substring(CLASSPATH_ALL_URL_PREFIX.length()));
  			}
  		}
  		else {
  			// Generally only look for a pattern after a prefix here,
  			// and on Tomcat only after the "*/" separator for its "war:" protocol.
  			int prefixEnd = (locationPattern.startsWith("war:") ? locationPattern.indexOf("*/") + 1 :
  					locationPattern.indexOf(':') + 1);
  			if (getPathMatcher().isPattern(locationPattern.substring(prefixEnd))) {
  				// a file pattern
  				return findPathMatchingResources(locationPattern);
  			}
  			else {
  				// a single resource with the given name
  				return new Resource[] {getResourceLoader().getResource(locationPattern)};
  			}
  		}
  	}
  	...
  	}
  ````

  **findPathMatchingResources,这个找出所有的给定资源匹配模式下的资源列表classpath:***

  ```
  /**
  	 * Find all resources that match the given location pattern via the
  	 * Ant-style PathMatcher. Supports resources in jar files and zip files
  	 * and in the file system.
  	 * @param locationPattern the location pattern to match
  	 * @return the result as Resource array
  	 * @throws IOException in case of I/O errors
  	 * @see #doFindPathMatchingJarResources
  	 * @see #doFindPathMatchingFileResources
  	 * @see org.springframework.util.PathMatcher
  	 */
  	protected Resource[] findPathMatchingResources(String locationPattern) throws IOException {
  		//获取给定location的根目录
  		String rootDirPath = determineRootDir(locationPattern);
  		String subPattern = locationPattern.substring(rootDirPath.length());
  		Resource[] rootDirResources = getResources(rootDirPath);
  		Set<Resource> result = new LinkedHashSet<>(16);
  		for (Resource rootDirResource : rootDirResources) {
  			rootDirResource = resolveRootDirResource(rootDirResource);
  			URL rootDirUrl = rootDirResource.getURL();
  			if (equinoxResolveMethod != null && rootDirUrl.getProtocol().startsWith("bundle")) {
  				URL resolvedUrl = (URL) ReflectionUtils.invokeMethod(equinoxResolveMethod, null, rootDirUrl);
  				if (resolvedUrl != null) {
  					rootDirUrl = resolvedUrl;
  				}
  				rootDirResource = new UrlResource(rootDirUrl);
  			}
  			if (rootDirUrl.getProtocol().startsWith(ResourceUtils.URL_PROTOCOL_VFS)) {
  				result.addAll(VfsResourceMatchingDelegate.findMatchingResources(rootDirUrl, subPattern, getPathMatcher()));
  			}
  			else if (ResourceUtils.isJarURL(rootDirUrl) || isJarResource(rootDirResource)) {
  				result.addAll(doFindPathMatchingJarResources(rootDirResource, rootDirUrl, subPattern));
  			}
  			else {
  				result.addAll(doFindPathMatchingFileResources(rootDirResource, subPattern));
  			}
  		}
  		if (logger.isTraceEnabled()) {
  			logger.trace("Resolved location pattern [" + locationPattern + "] to resources " + result);
  		}
  		return result.toArray(new Resource[0]);
  	}
  
  ```

  **findAllClassPathResources，获取所有classpath下的资源**

  ```
  	/**
  	 * Find all class location resources with the given location via the ClassLoader.
  	 * Delegates to {@link #doFindAllClassPathResources(String)}.
  	 * @param location the absolute path within the classpath
  	 * @return the result as Resource array
  	 * @throws IOException in case of I/O errors
  	 * @see java.lang.ClassLoader#getResources
  	 * @see #convertClassLoaderURL
  	 */
  	protected Resource[] findAllClassPathResources(String location) throws IOException {
  		String path = location;
  		if (path.startsWith("/")) {
  			path = path.substring(1);
  		}
  		Set<Resource> result = doFindAllClassPathResources(path);
  		if (logger.isTraceEnabled()) {
  			logger.trace("Resolved classpath location [" + location + "] to resources " + result);
  		}
  		return result.toArray(new Resource[0]);
  	}
  ```

  

  **3.1.2.3.1.1初始化一些配置文件**

  ```
  public abstract class AbstractRefreshableWebApplicationContext extends AbstractRefreshableConfigApplicationContext
  		implements ConfigurableWebApplicationContext, ThemeSource {
  		
  		...
  			/**
  	 * {@inheritDoc}
  	 * <p>Replace {@code Servlet}-related property sources.
  	 */
  	@Override
  	protected void initPropertySources() {
  	//获得环境对象
  		ConfigurableEnvironment env = getEnvironment();
  		if (env instanceof ConfigurableWebEnvironment) {
  		//加载并初始化配置source
  		//又跳到StandardServletEnvironment 进行3.1.2.1初始化一些配置properties
          ((ConfigurableWebEnvironment) env).initPropertySources(this.servletContext, 		this.servletConfig);
  		}
  	}
  		...
  		}
  ```

   **3.1.2.3.1告诉父类去刷新内置的bean工厂。**

  ```
  	/**
  	 * Tell the subclass to refresh the internal bean factory.
  	 * @return the fresh BeanFactory instance
  	 * @see #refreshBeanFactory()
  	 * @see #getBeanFactory()
  	 */
  	protected ConfigurableListableBeanFactory obtainFreshBeanFactory() {
  		refreshBeanFactory();
  		ConfigurableListableBeanFactory beanFactory = getBeanFactory();
  		if (logger.isDebugEnabled()) {
  			logger.debug("Bean factory for " + getDisplayName() + ": " + beanFactory);
  		}
  		return beanFactory;
  	}
  ```

  **3.1.2.3.3准备bean工厂，包含以下几项:**

  * BeanExpressionResolver(SpringEL解析器)
  * PropertyEditorRegistrar 向spring注册Java的PropertyEditor,定义bean的xml里面都是字符串那么由这个东西来转换为我们需要的类型。只有一个实现ResourceEditorRegistrar 
  * 设置环境注入 addBeanPostProcessor 这样可以将spring内部的一些对象注入到工厂
  * 依赖解析忽略 ignoreDependencyInterface 指定哪些依赖在注入的时候应该被忽略
  * bean 伪装 有些对象不在工厂中，但是我们依然想让他被装配到工厂中 那么 registerResolvableDependency 这个装配关系存贮在一个Map<Class<?>,Object>中

  ```
  /**
  	 * Configure the factory's standard context characteristics,
  	 * such as the context's ClassLoader and post-processors.
  	 * @param beanFactory the BeanFactory to configure
  	 */
  	protected void prepareBeanFactory(ConfigurableListableBeanFactory beanFactory) {
  		// Tell the internal bean factory to use the context's class loader etc.
  		beanFactory.setBeanClassLoader(getClassLoader());
  		beanFactory.setBeanExpressionResolver(new StandardBeanExpressionResolver(beanFactory.getBeanClassLoader()));
  		beanFactory.addPropertyEditorRegistrar(new ResourceEditorRegistrar(this, getEnvironment()));
  
  		// Configure the bean factory with context callbacks.
  		beanFactory.addBeanPostProcessor(new ApplicationContextAwareProcessor(this));
  		beanFactory.ignoreDependencyInterface(EnvironmentAware.class);
  		beanFactory.ignoreDependencyInterface(EmbeddedValueResolverAware.class);
  		beanFactory.ignoreDependencyInterface(ResourceLoaderAware.class);
  		beanFactory.ignoreDependencyInterface(ApplicationEventPublisherAware.class);
  		beanFactory.ignoreDependencyInterface(MessageSourceAware.class);
  		beanFactory.ignoreDependencyInterface(ApplicationContextAware.class);
  
  		// BeanFactory interface not registered as resolvable type in a plain factory.
  		// MessageSource registered (and found for autowiring) as a bean.
  		beanFactory.registerResolvableDependency(BeanFactory.class, beanFactory);
  		beanFactory.registerResolvableDependency(ResourceLoader.class, this);
  		beanFactory.registerResolvableDependency(ApplicationEventPublisher.class, this);
  		beanFactory.registerResolvableDependency(ApplicationContext.class, this);
  
  		// Register early post-processor for detecting inner beans as ApplicationListeners.
  		beanFactory.addBeanPostProcessor(new ApplicationListenerDetector(this));
  
  		// Detect a LoadTimeWeaver and prepare for weaving, if found.
  		if (beanFactory.containsBean(LOAD_TIME_WEAVER_BEAN_NAME)) {
  			beanFactory.addBeanPostProcessor(new LoadTimeWeaverAwareProcessor(beanFactory));
  			// Set a temporary ClassLoader for type matching.
  			beanFactory.setTempClassLoader(new ContextTypeMatchClassLoader(beanFactory.getBeanClassLoader()));
  		}
  
  		// Register default environment beans.
  		//将一些环境变量bean注册为单例对象
  		if (!beanFactory.containsLocalBean(ENVIRONMENT_BEAN_NAME)) {
  			beanFactory.registerSingleton(ENVIRONMENT_BEAN_NAME, getEnvironment());
  		}
  		if (!beanFactory.containsLocalBean(SYSTEM_PROPERTIES_BEAN_NAME)) {
  			beanFactory.registerSingleton(SYSTEM_PROPERTIES_BEAN_NAME, getEnvironment().getSystemProperties());
  		}
  		if (!beanFactory.containsLocalBean(SYSTEM_ENVIRONMENT_BEAN_NAME)) {
  			beanFactory.registerSingleton(SYSTEM_ENVIRONMENT_BEAN_NAME, getEnvironment().getSystemEnvironment());
  		}
  	}
  ```

  **这样的话SpringCore中关于Spring容器的启动（当然我这里直接是Web容器管理Spring容器）就基本完了回头可以画下流程图 ，普通的Spring容器的启动也是一样的**



