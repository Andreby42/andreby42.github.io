---
title: Mybatis源码分析(二)
date: 2018-11-07 18:22:20
tags: Mybatis
categories: Mybatis
---

Mybatis源码分析二<!--more-->

**基于3.4.5版本。与springboot2.0.5 mybatis-spring1.3.2**

* 主要的核心类
  * **Configuration: MyBatis 所有的配置信息都维持在 Configuration 对象之中。**
  * **SqlSession :作为 MyBatis 工作的主要顶层 API，表示和数据库交互的会话，完成必要数据库增删改查功能**
  * **Executor :MyBatis 执行器，是 MyBatis 调度的核心，负责 SQL 语句的生成和查询缓存的维护**
  * **StatementHandler :封装了 JDBC Statement 操作，负责对 JDBC statement 的操作，如设置参数、将 Statement 结果集转换成 List 集合。**
  * **ParameterHandler :负责对用户传递的参数转换成 JDBC Statement 所需要的参数 **
  * **ResultSetHandler :负责将 JDBC 返回的 ResultSet 结果集对象转换成 List 类型的集合**
  * **TypeHandler:负责 java 数据类型和 jdbc 数据类型之间的映射和转换**
  * **MappedStatement :MappedStatement 维护了一条 <select|update|delete|insert> 节点的封装**
  * **SqlSource :负责根据用户传递的 parameterObject，动态地生成 SQL 语句，将信息封装到 BoundSql 对象中并返回**
  * **BoundSql :表示动态生成的 SQL 语句以及相应的参数信息**

* **Mybatis启动过程**

  **这个类是个建造者 会创建sqlsessionFactory,提供了挺多的build方法。**

  上源码

  ```
  public class SqlSessionFactoryBuilder {
  ...
   public SqlSessionFactory build(Reader reader) {
      return build(reader, null, null);
    }
  
    public SqlSessionFactory build(Reader reader, String environment) {
      return build(reader, environment, null);
    }
  
    public SqlSessionFactory build(Reader reader, Properties properties) {
      return build(reader, null, properties);
    }
  
    public SqlSessionFactory build(Reader reader, String environment, Properties properties) {
      try {
        XMLConfigBuilder parser = new XMLConfigBuilder(reader, environment, properties);
        //注意这里的parser.parse();
        return build(parser.parse());
      } catch (Exception e) {
        throw ExceptionFactory.wrapException("Error building SqlSession.", e);
      } finally {
        //重置错误上下文实例
        ErrorContext.instance().reset();
        try {
          reader.close();
        } catch (IOException e) {
          // Intentionally ignore. Prefer previous error.
        }
      }
    }
    //parse方法
    public Configuration parse() {
      if (parsed) {
        throw new BuilderException("Each XMLConfigBuilder can only be used once.");
      }
      parsed = true;
      parseConfiguration(parser.evalNode("/configuration"));
      return configuration;
    }
   //具体的解析
    private void parseConfiguration(XNode root) {
      try {
        //issue #117 read properties first
        // 解析<properties>节点
        propertiesElement(root.evalNode("properties"));
        //解析settings节点
        Properties settings = settingsAsProperties(root.evalNode("settings"));
        //加载自定义的settings
        loadCustomVfs(settings);
        /// 解析<typeAliases>节点
        typeAliasesElement(root.evalNode("typeAliases"));
        // 解析<plugins>节点
        pluginElement(root.evalNode("plugins"));
        // 解析<objectFactory>节点
        objectFactoryElement(root.evalNode("objectFactory"));
        objectWrapperFactoryElement(root.evalNode("objectWrapperFactory"));
        // 解析<reflectorFactory>节点
        reflectorFactoryElement(root.evalNode("reflectorFactory"));
        settingsElement(settings);
         // 解析<environments>节点
        // read it after objectFactory and objectWrapperFactory issue #631
        environmentsElement(root.evalNode("environments"));
        //多数据源的话解析dbid
        databaseIdProviderElement(root.evalNode("databaseIdProvider"));
        typeHandlerElement(root.evalNode("typeHandlers"));
         // 解析<mappers>节点
        mapperElement(root.evalNode("mappers"));
      } catch (Exception e) {
        throw new BuilderException("Error parsing SQL Mapper Configuration. Cause: " + e, e);
      }
    }
  
    public SqlSessionFactory build(InputStream inputStream) {
      return build(inputStream, null, null);
    }
  
    public SqlSessionFactory build(InputStream inputStream, String environment) {
      return build(inputStream, environment, null);
    }
  
    public SqlSessionFactory build(InputStream inputStream, Properties properties) {
      return build(inputStream, null, properties);
    }
  
    public SqlSessionFactory build(InputStream inputStream, String environment, Properties properties) {
      try {
        XMLConfigBuilder parser = new XMLConfigBuilder(inputStream, environment, properties);
        return build(parser.parse());
      } catch (Exception e) {
        throw ExceptionFactory.wrapException("Error building SqlSession.", e);
      } finally {
        ErrorContext.instance().reset();
        try {
          inputStream.close();
        } catch (IOException e) {
          // Intentionally ignore. Prefer previous error.
        }
      }
    }
  ```

  **在项目启动的适合，我这里用的springboot集成的项目当然配置文件是写在application.properties的，springmvc项目一般在xml文件中配置的**

  ```
  ...
  mybatis.type-aliases-package=com.convergence.domain
  mybatis.mapper-locations=classpath:/mapper/*.xml
  mybatis.config-location=
  mybatis.configuration.map-underscore-to-camel-case=true
  mybatis.configuration.multiple-result-sets-enabled=true
  mybatis.configuration.useColumnLabel=true
  ...
  ```

  **创建factory**

  ```
    public class SqlSessionFactoryBuilder {
    ...
    public SqlSessionFactory build(Configuration config) {
      return new DefaultSqlSessionFactory(config);
    }
    ...
    }
  ```

  **Configuration类记录了mybatis的一些配置**

  ```
  /**
   *    Copyright 2009-2017 the original author or authors.
   *
   *    Licensed under the Apache License, Version 2.0 (the "License");
   *    you may not use this file except in compliance with the License.
   *    You may obtain a copy of the License at
   *
   *       http://www.apache.org/licenses/LICENSE-2.0
   *
   *    Unless required by applicable law or agreed to in writing, software
   *    distributed under the License is distributed on an "AS IS" BASIS,
   *    WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
   *    See the License for the specific language governing permissions and
   *    limitations under the License.
   */
  package org.apache.ibatis.session;
  
  import java.util.Arrays;
  import java.util.Collection;
  import java.util.HashMap;
  import java.util.HashSet;
  import java.util.LinkedList;
  import java.util.List;
  import java.util.Map;
  import java.util.Properties;
  import java.util.Set;
  
  import org.apache.ibatis.binding.MapperRegistry;
  import org.apache.ibatis.builder.CacheRefResolver;
  import org.apache.ibatis.builder.ResultMapResolver;
  import org.apache.ibatis.builder.annotation.MethodResolver;
  import org.apache.ibatis.builder.xml.XMLStatementBuilder;
  import org.apache.ibatis.cache.Cache;
  import org.apache.ibatis.cache.decorators.FifoCache;
  import org.apache.ibatis.cache.decorators.LruCache;
  import org.apache.ibatis.cache.decorators.SoftCache;
  import org.apache.ibatis.cache.decorators.WeakCache;
  import org.apache.ibatis.cache.impl.PerpetualCache;
  import org.apache.ibatis.datasource.jndi.JndiDataSourceFactory;
  import org.apache.ibatis.datasource.pooled.PooledDataSourceFactory;
  import org.apache.ibatis.datasource.unpooled.UnpooledDataSourceFactory;
  import org.apache.ibatis.executor.BatchExecutor;
  import org.apache.ibatis.executor.CachingExecutor;
  import org.apache.ibatis.executor.Executor;
  import org.apache.ibatis.executor.ReuseExecutor;
  import org.apache.ibatis.executor.SimpleExecutor;
  import org.apache.ibatis.executor.keygen.KeyGenerator;
  import org.apache.ibatis.executor.loader.ProxyFactory;
  import org.apache.ibatis.executor.loader.cglib.CglibProxyFactory;
  import org.apache.ibatis.executor.loader.javassist.JavassistProxyFactory;
  import org.apache.ibatis.executor.parameter.ParameterHandler;
  import org.apache.ibatis.executor.resultset.DefaultResultSetHandler;
  import org.apache.ibatis.executor.resultset.ResultSetHandler;
  import org.apache.ibatis.executor.statement.RoutingStatementHandler;
  import org.apache.ibatis.executor.statement.StatementHandler;
  import org.apache.ibatis.io.VFS;
  import org.apache.ibatis.logging.Log;
  import org.apache.ibatis.logging.LogFactory;
  import org.apache.ibatis.logging.commons.JakartaCommonsLoggingImpl;
  import org.apache.ibatis.logging.jdk14.Jdk14LoggingImpl;
  import org.apache.ibatis.logging.log4j.Log4jImpl;
  import org.apache.ibatis.logging.log4j2.Log4j2Impl;
  import org.apache.ibatis.logging.nologging.NoLoggingImpl;
  import org.apache.ibatis.logging.slf4j.Slf4jImpl;
  import org.apache.ibatis.logging.stdout.StdOutImpl;
  import org.apache.ibatis.mapping.BoundSql;
  import org.apache.ibatis.mapping.Environment;
  import org.apache.ibatis.mapping.MappedStatement;
  import org.apache.ibatis.mapping.ParameterMap;
  import org.apache.ibatis.mapping.ResultMap;
  import org.apache.ibatis.mapping.VendorDatabaseIdProvider;
  import org.apache.ibatis.parsing.XNode;
  import org.apache.ibatis.plugin.Interceptor;
  import org.apache.ibatis.plugin.InterceptorChain;
  import org.apache.ibatis.reflection.DefaultReflectorFactory;
  import org.apache.ibatis.reflection.MetaObject;
  import org.apache.ibatis.reflection.ReflectorFactory;
  import org.apache.ibatis.reflection.factory.DefaultObjectFactory;
  import org.apache.ibatis.reflection.factory.ObjectFactory;
  import org.apache.ibatis.reflection.wrapper.DefaultObjectWrapperFactory;
  import org.apache.ibatis.reflection.wrapper.ObjectWrapperFactory;
  import org.apache.ibatis.scripting.LanguageDriver;
  import org.apache.ibatis.scripting.LanguageDriverRegistry;
  import org.apache.ibatis.scripting.defaults.RawLanguageDriver;
  import org.apache.ibatis.scripting.xmltags.XMLLanguageDriver;
  import org.apache.ibatis.transaction.Transaction;
  import org.apache.ibatis.transaction.jdbc.JdbcTransactionFactory;
  import org.apache.ibatis.transaction.managed.ManagedTransactionFactory;
  import org.apache.ibatis.type.JdbcType;
  import org.apache.ibatis.type.TypeAliasRegistry;
  import org.apache.ibatis.type.TypeHandler;
  import org.apache.ibatis.type.TypeHandlerRegistry;
  
  /**
   * @author Clinton Begin
   */
  public class Configuration {
    //上下文环境对象
    protected Environment environment;
  
    protected boolean safeRowBoundsEnabled;
    protected boolean safeResultHandlerEnabled = true;
    protected boolean mapUnderscoreToCamelCase;
    //当启用时，有延迟加载属性的对象在被调用时将会完全加载任意属性。否则，每种属性将会按需要加载
    protected boolean aggressiveLazyLoading;
    //允许或不允许多种结果集从一个单独的语句中返回（需要适合的驱动）。
    protected boolean multipleResultSetsEnabled = true;
    //允许 JDBC 支持生成的键。需要适合的驱动。如果设置为 true 则这个设置强制生成的键被使用，尽管一些	//驱动拒绝兼容但仍然有效（比如 Derby
    protected boolean useGeneratedKeys;
    //使用列标签代替列名。不同的驱动在这方便表现不同。参考驱动文档或充分测试两种方法来决定所使用的驱动
    protected boolean useColumnLabel = true;
    //是否全局启用缓存 默认开启
    protected boolean cacheEnabled = true;
    protected boolean callSettersOnNulls;
    protected boolean useActualParamName = true;
    protected boolean returnInstanceForEmptyRow;
  
    protected String logPrefix;
    protected Class <? extends Log> logImpl;
    protected Class <? extends VFS> vfsImpl;
    protected LocalCacheScope localCacheScope = LocalCacheScope.SESSION;
    protected JdbcType jdbcTypeForNull = JdbcType.OTHER;
    protected Set<String> lazyLoadTriggerMethods = new HashSet<String>(Arrays.asList(new String[] { "equals", "clone", "hashCode", "toString" }));
    protected Integer defaultStatementTimeout;
    protected Integer defaultFetchSize;
    protected ExecutorType defaultExecutorType = ExecutorType.SIMPLE;
    //指定 MyBatis 如何自动映射列到字段 / 属性。PARTIAL 只会自动映射简单，没有嵌套的结果。FULL 会自//动映射任意复杂的结果（嵌套的或其他情况）。
    protected AutoMappingBehavior autoMappingBehavior = AutoMappingBehavior.PARTIAL;
    protected AutoMappingUnknownColumnBehavior autoMappingUnknownColumnBehavior = AutoMappingUnknownColumnBehavior.NONE;
  
    protected Properties variables = new Properties();
    protected ReflectorFactory reflectorFactory = new DefaultReflectorFactory();
    protected ObjectFactory objectFactory = new DefaultObjectFactory();
    protected ObjectWrapperFactory objectWrapperFactory = new DefaultObjectWrapperFactory();
    //全局是否启用活禁用懒加载，默认关闭 
    protected boolean lazyLoadingEnabled = false;
    protected ProxyFactory proxyFactory = new JavassistProxyFactory(); // #224 Using internal Javassist instead of OGNL
  
    protected String databaseId;
    /**
     * Configuration factory class.
     * Used to create Configuration for loading deserialized unread properties.
     *
     * @see <a href='https://code.google.com/p/mybatis/issues/detail?id=300'>Issue 300 (google code)</a>
     */
    protected Class<?> configurationFactory;
  
    protected final MapperRegistry mapperRegistry = new MapperRegistry(this);
    protected final InterceptorChain interceptorChain = new InterceptorChain();
    protected final TypeHandlerRegistry typeHandlerRegistry = new TypeHandlerRegistry();
    protected final TypeAliasRegistry typeAliasRegistry = new TypeAliasRegistry();
    protected final LanguageDriverRegistry languageRegistry = new LanguageDriverRegistry();
  
    protected final Map<String, MappedStatement> mappedStatements = new StrictMap<MappedStatement>("Mapped Statements collection");
    protected final Map<String, Cache> caches = new StrictMap<Cache>("Caches collection");
    protected final Map<String, ResultMap> resultMaps = new StrictMap<ResultMap>("Result Maps collection");
    protected final Map<String, ParameterMap> parameterMaps = new StrictMap<ParameterMap>("Parameter Maps collection");
    protected final Map<String, KeyGenerator> keyGenerators = new StrictMap<KeyGenerator>("Key Generators collection");
  
    protected final Set<String> loadedResources = new HashSet<String>();
    protected final Map<String, XNode> sqlFragments = new StrictMap<XNode>("XML fragments parsed from previous mappers");
  
    protected final Collection<XMLStatementBuilder> incompleteStatements = new LinkedList<XMLStatementBuilder>();
    protected final Collection<CacheRefResolver> incompleteCacheRefs = new LinkedList<CacheRefResolver>();
    protected final Collection<ResultMapResolver> incompleteResultMaps = new LinkedList<ResultMapResolver>();
    protected final Collection<MethodResolver> incompleteMethods = new LinkedList<MethodResolver>();
  
    /*
     * A map holds cache-ref relationship. The key is the namespace that
     * references a cache bound to another namespace and the value is the
     * namespace which the actual cache is bound to.
     */
    protected final Map<String, String> cacheRefMap = new HashMap<String, String>();
  
    public Configuration(Environment environment) {
      this();
      this.environment = environment;
    }
  
    public Configuration() {
      typeAliasRegistry.registerAlias("JDBC", JdbcTransactionFactory.class);
      typeAliasRegistry.registerAlias("MANAGED", ManagedTransactionFactory.class);
  
      typeAliasRegistry.registerAlias("JNDI", JndiDataSourceFactory.class);
      typeAliasRegistry.registerAlias("POOLED", PooledDataSourceFactory.class);
      typeAliasRegistry.registerAlias("UNPOOLED", UnpooledDataSourceFactory.class);
  
      typeAliasRegistry.registerAlias("PERPETUAL", PerpetualCache.class);
      typeAliasRegistry.registerAlias("FIFO", FifoCache.class);
      typeAliasRegistry.registerAlias("LRU", LruCache.class);
      typeAliasRegistry.registerAlias("SOFT", SoftCache.class);
      typeAliasRegistry.registerAlias("WEAK", WeakCache.class);
  
      typeAliasRegistry.registerAlias("DB_VENDOR", VendorDatabaseIdProvider.class);
  
      typeAliasRegistry.registerAlias("XML", XMLLanguageDriver.class);
      typeAliasRegistry.registerAlias("RAW", RawLanguageDriver.class);
  
      typeAliasRegistry.registerAlias("SLF4J", Slf4jImpl.class);
      typeAliasRegistry.registerAlias("COMMONS_LOGGING", JakartaCommonsLoggingImpl.class);
      typeAliasRegistry.registerAlias("LOG4J", Log4jImpl.class);
      typeAliasRegistry.registerAlias("LOG4J2", Log4j2Impl.class);
      typeAliasRegistry.registerAlias("JDK_LOGGING", Jdk14LoggingImpl.class);
      typeAliasRegistry.registerAlias("STDOUT_LOGGING", StdOutImpl.class);
      typeAliasRegistry.registerAlias("NO_LOGGING", NoLoggingImpl.class);
  
      typeAliasRegistry.registerAlias("CGLIB", CglibProxyFactory.class);
      typeAliasRegistry.registerAlias("JAVASSIST", JavassistProxyFactory.class);
  
      languageRegistry.setDefaultDriverClass(XMLLanguageDriver.class);
      languageRegistry.register(RawLanguageDriver.class);
    }
  
    public String getLogPrefix() {
      return logPrefix;
    }
  
    public void setLogPrefix(String logPrefix) {
      this.logPrefix = logPrefix;
    }
  
    public Class<? extends Log> getLogImpl() {
      return logImpl;
    }
  
    public void setLogImpl(Class<? extends Log> logImpl) {
      if (logImpl != null) {
        this.logImpl = logImpl;
        LogFactory.useCustomLogging(this.logImpl);
      }
    }
  
    public Class<? extends VFS> getVfsImpl() {
      return this.vfsImpl;
    }
  
    public void setVfsImpl(Class<? extends VFS> vfsImpl) {
      if (vfsImpl != null) {
        this.vfsImpl = vfsImpl;
        VFS.addImplClass(this.vfsImpl);
      }
    }
  
    public boolean isCallSettersOnNulls() {
      return callSettersOnNulls;
    }
  
    public void setCallSettersOnNulls(boolean callSettersOnNulls) {
      this.callSettersOnNulls = callSettersOnNulls;
    }
  
    public boolean isUseActualParamName() {
      return useActualParamName;
    }
  
    public void setUseActualParamName(boolean useActualParamName) {
      this.useActualParamName = useActualParamName;
    }
  
    public boolean isReturnInstanceForEmptyRow() {
      return returnInstanceForEmptyRow;
    }
  
    public void setReturnInstanceForEmptyRow(boolean returnEmptyInstance) {
      this.returnInstanceForEmptyRow = returnEmptyInstance;
    }
  
    public String getDatabaseId() {
      return databaseId;
    }
  
    public void setDatabaseId(String databaseId) {
      this.databaseId = databaseId;
    }
  
    public Class<?> getConfigurationFactory() {
      return configurationFactory;
    }
  
    public void setConfigurationFactory(Class<?> configurationFactory) {
      this.configurationFactory = configurationFactory;
    }
  
    public boolean isSafeResultHandlerEnabled() {
      return safeResultHandlerEnabled;
    }
  
    public void setSafeResultHandlerEnabled(boolean safeResultHandlerEnabled) {
      this.safeResultHandlerEnabled = safeResultHandlerEnabled;
    }
  
    public boolean isSafeRowBoundsEnabled() {
      return safeRowBoundsEnabled;
    }
  
    public void setSafeRowBoundsEnabled(boolean safeRowBoundsEnabled) {
      this.safeRowBoundsEnabled = safeRowBoundsEnabled;
    }
  
    public boolean isMapUnderscoreToCamelCase() {
      return mapUnderscoreToCamelCase;
    }
  
    public void setMapUnderscoreToCamelCase(boolean mapUnderscoreToCamelCase) {
      this.mapUnderscoreToCamelCase = mapUnderscoreToCamelCase;
    }
  
    public void addLoadedResource(String resource) {
      loadedResources.add(resource);
    }
  
    public boolean isResourceLoaded(String resource) {
      return loadedResources.contains(resource);
    }
  
    public Environment getEnvironment() {
      return environment;
    }
  
    public void setEnvironment(Environment environment) {
      this.environment = environment;
    }
  
    public AutoMappingBehavior getAutoMappingBehavior() {
      return autoMappingBehavior;
    }
  
    public void setAutoMappingBehavior(AutoMappingBehavior autoMappingBehavior) {
      this.autoMappingBehavior = autoMappingBehavior;
    }
  
    /**
     * @since 3.4.0
     */
    public AutoMappingUnknownColumnBehavior getAutoMappingUnknownColumnBehavior() {
      return autoMappingUnknownColumnBehavior;
    }
  
    /**
     * @since 3.4.0
     */
    public void setAutoMappingUnknownColumnBehavior(AutoMappingUnknownColumnBehavior autoMappingUnknownColumnBehavior) {
      this.autoMappingUnknownColumnBehavior = autoMappingUnknownColumnBehavior;
    }
  
    public boolean isLazyLoadingEnabled() {
      return lazyLoadingEnabled;
    }
  
    public void setLazyLoadingEnabled(boolean lazyLoadingEnabled) {
      this.lazyLoadingEnabled = lazyLoadingEnabled;
    }
  
    public ProxyFactory getProxyFactory() {
      return proxyFactory;
    }
  
    public void setProxyFactory(ProxyFactory proxyFactory) {
      if (proxyFactory == null) {
        proxyFactory = new JavassistProxyFactory();
      }
      this.proxyFactory = proxyFactory;
    }
  
    public boolean isAggressiveLazyLoading() {
      return aggressiveLazyLoading;
    }
  
    public void setAggressiveLazyLoading(boolean aggressiveLazyLoading) {
      this.aggressiveLazyLoading = aggressiveLazyLoading;
    }
  
    public boolean isMultipleResultSetsEnabled() {
      return multipleResultSetsEnabled;
    }
  
    public void setMultipleResultSetsEnabled(boolean multipleResultSetsEnabled) {
      this.multipleResultSetsEnabled = multipleResultSetsEnabled;
    }
  
    public Set<String> getLazyLoadTriggerMethods() {
      return lazyLoadTriggerMethods;
    }
  
    public void setLazyLoadTriggerMethods(Set<String> lazyLoadTriggerMethods) {
      this.lazyLoadTriggerMethods = lazyLoadTriggerMethods;
    }
  
    public boolean isUseGeneratedKeys() {
      return useGeneratedKeys;
    }
  
    public void setUseGeneratedKeys(boolean useGeneratedKeys) {
      this.useGeneratedKeys = useGeneratedKeys;
    }
  
    public ExecutorType getDefaultExecutorType() {
      return defaultExecutorType;
    }
  
    public void setDefaultExecutorType(ExecutorType defaultExecutorType) {
      this.defaultExecutorType = defaultExecutorType;
    }
  
    public boolean isCacheEnabled() {
      return cacheEnabled;
    }
  
    public void setCacheEnabled(boolean cacheEnabled) {
      this.cacheEnabled = cacheEnabled;
    }
  
    public Integer getDefaultStatementTimeout() {
      return defaultStatementTimeout;
    }
  
    public void setDefaultStatementTimeout(Integer defaultStatementTimeout) {
      this.defaultStatementTimeout = defaultStatementTimeout;
    }
  
    /**
     * @since 3.3.0
     */
    public Integer getDefaultFetchSize() {
      return defaultFetchSize;
    }
  
    /**
     * @since 3.3.0
     */
    public void setDefaultFetchSize(Integer defaultFetchSize) {
      this.defaultFetchSize = defaultFetchSize;
    }
  
    public boolean isUseColumnLabel() {
      return useColumnLabel;
    }
  
    public void setUseColumnLabel(boolean useColumnLabel) {
      this.useColumnLabel = useColumnLabel;
    }
  
    public LocalCacheScope getLocalCacheScope() {
      return localCacheScope;
    }
  
    public void setLocalCacheScope(LocalCacheScope localCacheScope) {
      this.localCacheScope = localCacheScope;
    }
  
    public JdbcType getJdbcTypeForNull() {
      return jdbcTypeForNull;
    }
  
    public void setJdbcTypeForNull(JdbcType jdbcTypeForNull) {
      this.jdbcTypeForNull = jdbcTypeForNull;
    }
  
    public Properties getVariables() {
      return variables;
    }
  
    public void setVariables(Properties variables) {
      this.variables = variables;
    }
  
    public TypeHandlerRegistry getTypeHandlerRegistry() {
      return typeHandlerRegistry;
    }
  
    /**
     * Set a default {@link TypeHandler} class for {@link Enum}.
     * A default {@link TypeHandler} is {@link org.apache.ibatis.type.EnumTypeHandler}.
     * @param typeHandler a type handler class for {@link Enum}
     * @since 3.4.5
     */
    public void setDefaultEnumTypeHandler(Class<? extends TypeHandler> typeHandler) {
      if (typeHandler != null) {
        getTypeHandlerRegistry().setDefaultEnumTypeHandler(typeHandler);
      }
    }
  
    public TypeAliasRegistry getTypeAliasRegistry() {
      return typeAliasRegistry;
    }
  
    /**
     * @since 3.2.2
     */
    public MapperRegistry getMapperRegistry() {
      return mapperRegistry;
    }
  
    public ReflectorFactory getReflectorFactory() {
  	  return reflectorFactory;
    }
  
    public void setReflectorFactory(ReflectorFactory reflectorFactory) {
  	  this.reflectorFactory = reflectorFactory;
    }
  
    public ObjectFactory getObjectFactory() {
      return objectFactory;
    }
  
    public void setObjectFactory(ObjectFactory objectFactory) {
      this.objectFactory = objectFactory;
    }
  
    public ObjectWrapperFactory getObjectWrapperFactory() {
      return objectWrapperFactory;
    }
  
    public void setObjectWrapperFactory(ObjectWrapperFactory objectWrapperFactory) {
      this.objectWrapperFactory = objectWrapperFactory;
    }
  
    /**
     * @since 3.2.2
     */
    public List<Interceptor> getInterceptors() {
      return interceptorChain.getInterceptors();
    }
  
    public LanguageDriverRegistry getLanguageRegistry() {
      return languageRegistry;
    }
  
    public void setDefaultScriptingLanguage(Class<?> driver) {
      if (driver == null) {
        driver = XMLLanguageDriver.class;
      }
      getLanguageRegistry().setDefaultDriverClass(driver);
    }
  
    public LanguageDriver getDefaultScriptingLanguageInstance() {
      return languageRegistry.getDefaultDriver();
    }
  
    /** @deprecated Use {@link #getDefaultScriptingLanguageInstance()} */
    @Deprecated
    public LanguageDriver getDefaultScriptingLanuageInstance() {
      return getDefaultScriptingLanguageInstance();
    }
  
    public MetaObject newMetaObject(Object object) {
      return MetaObject.forObject(object, objectFactory, objectWrapperFactory, reflectorFactory);
    }
  
    public ParameterHandler newParameterHandler(MappedStatement mappedStatement, Object parameterObject, BoundSql boundSql) {
      ParameterHandler parameterHandler = mappedStatement.getLang().createParameterHandler(mappedStatement, parameterObject, boundSql);
      parameterHandler = (ParameterHandler) interceptorChain.pluginAll(parameterHandler);
      return parameterHandler;
    }
  
    public ResultSetHandler newResultSetHandler(Executor executor, MappedStatement mappedStatement, RowBounds rowBounds, ParameterHandler parameterHandler,
        ResultHandler resultHandler, BoundSql boundSql) {
      ResultSetHandler resultSetHandler = new DefaultResultSetHandler(executor, mappedStatement, parameterHandler, resultHandler, boundSql, rowBounds);
      resultSetHandler = (ResultSetHandler) interceptorChain.pluginAll(resultSetHandler);
      return resultSetHandler;
    }
  
    public StatementHandler newStatementHandler(Executor executor, MappedStatement mappedStatement, Object parameterObject, RowBounds rowBounds, ResultHandler resultHandler, BoundSql boundSql) {
      StatementHandler statementHandler = new RoutingStatementHandler(executor, mappedStatement, parameterObject, rowBounds, resultHandler, boundSql);
      statementHandler = (StatementHandler) interceptorChain.pluginAll(statementHandler);
      return statementHandler;
    }
  
    public Executor newExecutor(Transaction transaction) {
      return newExecutor(transaction, defaultExecutorType);
    }
  
    public Executor newExecutor(Transaction transaction, ExecutorType executorType) {
      executorType = executorType == null ? defaultExecutorType : executorType;
      executorType = executorType == null ? ExecutorType.SIMPLE : executorType;
      Executor executor;
      if (ExecutorType.BATCH == executorType) {
        executor = new BatchExecutor(this, transaction);
      } else if (ExecutorType.REUSE == executorType) {
        executor = new ReuseExecutor(this, transaction);
      } else {
        executor = new SimpleExecutor(this, transaction);
      }
      if (cacheEnabled) {
        executor = new CachingExecutor(executor);
      }
      executor = (Executor) interceptorChain.pluginAll(executor);
      return executor;
    }
  
    public void addKeyGenerator(String id, KeyGenerator keyGenerator) {
      keyGenerators.put(id, keyGenerator);
    }
  
    public Collection<String> getKeyGeneratorNames() {
      return keyGenerators.keySet();
    }
  
    public Collection<KeyGenerator> getKeyGenerators() {
      return keyGenerators.values();
    }
  
    public KeyGenerator getKeyGenerator(String id) {
      return keyGenerators.get(id);
    }
  
    public boolean hasKeyGenerator(String id) {
      return keyGenerators.containsKey(id);
    }
  
    public void addCache(Cache cache) {
      caches.put(cache.getId(), cache);
    }
  
    public Collection<String> getCacheNames() {
      return caches.keySet();
    }
  
    public Collection<Cache> getCaches() {
      return caches.values();
    }
  
    public Cache getCache(String id) {
      return caches.get(id);
    }
  
    public boolean hasCache(String id) {
      return caches.containsKey(id);
    }
  
    public void addResultMap(ResultMap rm) {
      resultMaps.put(rm.getId(), rm);
      checkLocallyForDiscriminatedNestedResultMaps(rm);
      checkGloballyForDiscriminatedNestedResultMaps(rm);
    }
  
    public Collection<String> getResultMapNames() {
      return resultMaps.keySet();
    }
  
    public Collection<ResultMap> getResultMaps() {
      return resultMaps.values();
    }
  
    public ResultMap getResultMap(String id) {
      return resultMaps.get(id);
    }
  
    public boolean hasResultMap(String id) {
      return resultMaps.containsKey(id);
    }
  
    public void addParameterMap(ParameterMap pm) {
      parameterMaps.put(pm.getId(), pm);
    }
  
    public Collection<String> getParameterMapNames() {
      return parameterMaps.keySet();
    }
  
    public Collection<ParameterMap> getParameterMaps() {
      return parameterMaps.values();
    }
  
    public ParameterMap getParameterMap(String id) {
      return parameterMaps.get(id);
    }
  
    public boolean hasParameterMap(String id) {
      return parameterMaps.containsKey(id);
    }
  
    public void addMappedStatement(MappedStatement ms) {
      mappedStatements.put(ms.getId(), ms);
    }
  
    public Collection<String> getMappedStatementNames() {
      buildAllStatements();
      return mappedStatements.keySet();
    }
  
    public Collection<MappedStatement> getMappedStatements() {
      buildAllStatements();
      return mappedStatements.values();
    }
  
    public Collection<XMLStatementBuilder> getIncompleteStatements() {
      return incompleteStatements;
    }
  
    public void addIncompleteStatement(XMLStatementBuilder incompleteStatement) {
      incompleteStatements.add(incompleteStatement);
    }
  
    public Collection<CacheRefResolver> getIncompleteCacheRefs() {
      return incompleteCacheRefs;
    }
  
    public void addIncompleteCacheRef(CacheRefResolver incompleteCacheRef) {
      incompleteCacheRefs.add(incompleteCacheRef);
    }
  
    public Collection<ResultMapResolver> getIncompleteResultMaps() {
      return incompleteResultMaps;
    }
  
    public void addIncompleteResultMap(ResultMapResolver resultMapResolver) {
      incompleteResultMaps.add(resultMapResolver);
    }
  
    public void addIncompleteMethod(MethodResolver builder) {
      incompleteMethods.add(builder);
    }
  
    public Collection<MethodResolver> getIncompleteMethods() {
      return incompleteMethods;
    }
  
    public MappedStatement getMappedStatement(String id) {
      return this.getMappedStatement(id, true);
    }
  
    public MappedStatement getMappedStatement(String id, boolean validateIncompleteStatements) {
      if (validateIncompleteStatements) {
        buildAllStatements();
      }
      return mappedStatements.get(id);
    }
  
    public Map<String, XNode> getSqlFragments() {
      return sqlFragments;
    }
  
    public void addInterceptor(Interceptor interceptor) {
      interceptorChain.addInterceptor(interceptor);
    }
  
    public void addMappers(String packageName, Class<?> superType) {
      mapperRegistry.addMappers(packageName, superType);
    }
  
    public void addMappers(String packageName) {
      mapperRegistry.addMappers(packageName);
    }
  
    public <T> void addMapper(Class<T> type) {
      mapperRegistry.addMapper(type);
    }
  
    public <T> T getMapper(Class<T> type, SqlSession sqlSession) {
      return mapperRegistry.getMapper(type, sqlSession);
    }
  
    public boolean hasMapper(Class<?> type) {
      return mapperRegistry.hasMapper(type);
    }
  
    public boolean hasStatement(String statementName) {
      return hasStatement(statementName, true);
    }
  
    public boolean hasStatement(String statementName, boolean validateIncompleteStatements) {
      if (validateIncompleteStatements) {
        buildAllStatements();
      }
      return mappedStatements.containsKey(statementName);
    }
  
    public void addCacheRef(String namespace, String referencedNamespace) {
      cacheRefMap.put(namespace, referencedNamespace);
    }
  
    /*
     * Parses all the unprocessed statement nodes in the cache. It is recommended
     * to call this method once all the mappers are added as it provides fail-fast
     * statement validation.
     */
    protected void buildAllStatements() {
      if (!incompleteResultMaps.isEmpty()) {
        synchronized (incompleteResultMaps) {
          // This always throws a BuilderException.
          incompleteResultMaps.iterator().next().resolve();
        }
      }
      if (!incompleteCacheRefs.isEmpty()) {
        synchronized (incompleteCacheRefs) {
          // This always throws a BuilderException.
          incompleteCacheRefs.iterator().next().resolveCacheRef();
        }
      }
      if (!incompleteStatements.isEmpty()) {
        synchronized (incompleteStatements) {
          // This always throws a BuilderException.
          incompleteStatements.iterator().next().parseStatementNode();
        }
      }
      if (!incompleteMethods.isEmpty()) {
        synchronized (incompleteMethods) {
          // This always throws a BuilderException.
          incompleteMethods.iterator().next().resolve();
        }
      }
    }
  
    /*
     * Extracts namespace from fully qualified statement id.
     *
     * @param statementId
     * @return namespace or null when id does not contain period.
     */
    protected String extractNamespace(String statementId) {
      int lastPeriod = statementId.lastIndexOf('.');
      return lastPeriod > 0 ? statementId.substring(0, lastPeriod) : null;
    }
  
    // Slow but a one time cost. A better solution is welcome.
    protected void checkGloballyForDiscriminatedNestedResultMaps(ResultMap rm) {
      if (rm.hasNestedResultMaps()) {
        for (Map.Entry<String, ResultMap> entry : resultMaps.entrySet()) {
          Object value = entry.getValue();
          if (value instanceof ResultMap) {
            ResultMap entryResultMap = (ResultMap) value;
            if (!entryResultMap.hasNestedResultMaps() && entryResultMap.getDiscriminator() != null) {
              Collection<String> discriminatedResultMapNames = entryResultMap.getDiscriminator().getDiscriminatorMap().values();
              if (discriminatedResultMapNames.contains(rm.getId())) {
                entryResultMap.forceNestedResultMaps();
              }
            }
          }
        }
      }
    }
  
    // Slow but a one time cost. A better solution is welcome.
    protected void checkLocallyForDiscriminatedNestedResultMaps(ResultMap rm) {
      if (!rm.hasNestedResultMaps() && rm.getDiscriminator() != null) {
        for (Map.Entry<String, String> entry : rm.getDiscriminator().getDiscriminatorMap().entrySet()) {
          String discriminatedResultMapName = entry.getValue();
          if (hasResultMap(discriminatedResultMapName)) {
            ResultMap discriminatedResultMap = resultMaps.get(discriminatedResultMapName);
            if (discriminatedResultMap.hasNestedResultMaps()) {
              rm.forceNestedResultMaps();
              break;
            }
          }
        }
      }
    }
  
    protected static class StrictMap<V> extends HashMap<String, V> {
  
      private static final long serialVersionUID = -4950446264854982944L;
      private final String name;
  
      public StrictMap(String name, int initialCapacity, float loadFactor) {
        super(initialCapacity, loadFactor);
        this.name = name;
      }
  
      public StrictMap(String name, int initialCapacity) {
        super(initialCapacity);
        this.name = name;
      }
  
      public StrictMap(String name) {
        super();
        this.name = name;
      }
  
      public StrictMap(String name, Map<String, ? extends V> m) {
        super(m);
        this.name = name;
      }
  
      @SuppressWarnings("unchecked")
      public V put(String key, V value) {
        if (containsKey(key)) {
          throw new IllegalArgumentException(name + " already contains value for " + key);
        }
        if (key.contains(".")) {
          final String shortKey = getShortName(key);
          if (super.get(shortKey) == null) {
            super.put(shortKey, value);
          } else {
            super.put(shortKey, (V) new Ambiguity(shortKey));
          }
        }
        return super.put(key, value);
      }
  
      public V get(Object key) {
        V value = super.get(key);
        if (value == null) {
          throw new IllegalArgumentException(name + " does not contain value for " + key);
        }
        if (value instanceof Ambiguity) {
          throw new IllegalArgumentException(((Ambiguity) value).getSubject() + " is ambiguous in " + name
              + " (try using the full name including the namespace, or rename one of the entries)");
        }
        return value;
      }
  
      private String getShortName(String key) {
        final String[] keyParts = key.split("\\.");
        return keyParts[keyParts.length - 1];
      }
  
      protected static class Ambiguity {
        final private String subject;
  
        public Ambiguity(String subject) {
          this.subject = subject;
        }
  
        public String getSubject() {
          return subject;
        }
      }
    }
  
  }
  
  ```

  **然后代码跳到这里**

  ```
  public class SqlSessionFactoryBean implements FactoryBean<SqlSessionFactory>, InitializingBean, ApplicationListener<ApplicationEvent> {
  ...
  //这里是创建唯一的sqlsessionFactory注意这个注解ConditionalOnMissingBean
    @Bean
    @ConditionalOnMissingBean
    public SqlSessionFactory sqlSessionFactory(DataSource dataSource) throws Exception {
      SqlSessionFactoryBean factory = new SqlSessionFactoryBean();
      factory.setDataSource(dataSource);
      factory.setVfs(SpringBootVFS.class);
      if (StringUtils.hasText(this.properties.getConfigLocation())) {
        factory.setConfigLocation(this.resourceLoader.getResource(this.properties.getConfigLocation()));
      }
      Configuration configuration = this.properties.getConfiguration();
      if (configuration == null && !StringUtils.hasText(this.properties.getConfigLocation())) {
        configuration = new Configuration();
      }
      if (configuration != null && !CollectionUtils.isEmpty(this.configurationCustomizers)) {
        for (ConfigurationCustomizer customizer : this.configurationCustomizers) {
          customizer.customize(configuration);
        }
      }
      factory.setConfiguration(configuration);
      if (this.properties.getConfigurationProperties() != null) {
        factory.setConfigurationProperties(this.properties.getConfigurationProperties());
      }
      if (!ObjectUtils.isEmpty(this.interceptors)) {
        factory.setPlugins(this.interceptors);
      }
      if (this.databaseIdProvider != null) {
        factory.setDatabaseIdProvider(this.databaseIdProvider);
      }
      if (StringUtils.hasLength(this.properties.getTypeAliasesPackage())) {
        factory.setTypeAliasesPackage(this.properties.getTypeAliasesPackage());
      }
      if (StringUtils.hasLength(this.properties.getTypeHandlersPackage())) {
        factory.setTypeHandlersPackage(this.properties.getTypeHandlersPackage());
      }
      if (!ObjectUtils.isEmpty(this.properties.resolveMapperLocations())) {
        factory.setMapperLocations(this.properties.resolveMapperLocations());
      }
  	//创建工厂
      return factory.getObject();
    }
  ...
    /**
     * {@inheritDoc}
     */
     //先走这个方法
    @Override
    public void afterPropertiesSet() throws Exception {
      notNull(dataSource, "Property 'dataSource' is required");
      notNull(sqlSessionFactoryBuilder, "Property 'sqlSessionFactoryBuilder' is required");
      state((configuration == null && configLocation == null) || !(configuration != null && configLocation != null),
                "Property 'configuration' and 'configLocation' can not specified with together");
  	//走这里进行build工厂类
      this.sqlSessionFactory = buildSqlSessionFactory();
    }
    ...
    //注意了具体的配置过程在这里
      protected SqlSessionFactory buildSqlSessionFactory() throws IOException {
  
      Configuration configuration;
  
      XMLConfigBuilder xmlConfigBuilder = null;
      if (this.configuration != null) {
        configuration = this.configuration;
        if (configuration.getVariables() == null) {
          configuration.setVariables(this.configurationProperties);
        } else if (this.configurationProperties != null) {
          configuration.getVariables().putAll(this.configurationProperties);
        }
      } else if (this.configLocation != null) {
        xmlConfigBuilder = new XMLConfigBuilder(this.configLocation.getInputStream(), null, this.configurationProperties);
        configuration = xmlConfigBuilder.getConfiguration();
      } else {
        if (LOGGER.isDebugEnabled()) {
          LOGGER.debug("Property 'configuration' or 'configLocation' not specified, using default MyBatis Configuration");
        }
        configuration = new Configuration();
        if (this.configurationProperties != null) {
          configuration.setVariables(this.configurationProperties);
        }
      }
  
      if (this.objectFactory != null) {
        configuration.setObjectFactory(this.objectFactory);
      }
  
      if (this.objectWrapperFactory != null) {
        configuration.setObjectWrapperFactory(this.objectWrapperFactory);
      }
  
      if (this.vfs != null) {
        configuration.setVfsImpl(this.vfs);
      }
  	//this.typeAliasesPackage=application.properties中的mybatis.type-aliases-package
      if (hasLength(this.typeAliasesPackage)) {
        String[] typeAliasPackageArray = tokenizeToStringArray(this.typeAliasesPackage,
            ConfigurableApplicationContext.CONFIG_LOCATION_DELIMITERS);
        for (String packageToScan : typeAliasPackageArray) {
          configuration.getTypeAliasRegistry().registerAliases(packageToScan,
                  typeAliasesSuperType == null ? Object.class : typeAliasesSuperType);
          if (LOGGER.isDebugEnabled()) {
            LOGGER.debug("Scanned package: '" + packageToScan + "' for aliases");
          }
        }
      }
  	//注册别名类到configuration
      if (!isEmpty(this.typeAliases)) {
        for (Class<?> typeAlias : this.typeAliases) {
          configuration.getTypeAliasRegistry().registerAlias(typeAlias);
          if (LOGGER.isDebugEnabled()) {
            LOGGER.debug("Registered type alias: '" + typeAlias + "'");
          }
        }
      }
  	//看看有没有配置插件有的话加入到拦截器组
      if (!isEmpty(this.plugins)) {
        for (Interceptor plugin : this.plugins) {
          configuration.addInterceptor(plugin);
          if (LOGGER.isDebugEnabled()) {
            LOGGER.debug("Registered plugin: '" + plugin + "'");
          }
        }
      }
  	//看看有没有typeHandlersPackage，有的话注册到cfg类中
      if (hasLength(this.typeHandlersPackage)) {
        String[] typeHandlersPackageArray = tokenizeToStringArray(this.typeHandlersPackage,
            ConfigurableApplicationContext.CONFIG_LOCATION_DELIMITERS);
        for (String packageToScan : typeHandlersPackageArray) {
          configuration.getTypeHandlerRegistry().register(packageToScan);
          if (LOGGER.isDebugEnabled()) {
            LOGGER.debug("Scanned package: '" + packageToScan + "' for type handlers");
          }
        }
      }
  	//看看有没有typeHandlers，有的话注册到cfg类中
      if (!isEmpty(this.typeHandlers)) {
        for (TypeHandler<?> typeHandler : this.typeHandlers) {
          configuration.getTypeHandlerRegistry().register(typeHandler);
          if (LOGGER.isDebugEnabled()) {
            LOGGER.debug("Registered type handler: '" + typeHandler + "'");
          }
        }
      }
  	//看看有没有设置databaseid 有的话配置到cfg类中
      if (this.databaseIdProvider != null) {//fix #64 set databaseId before parse mapper xmls
        try {
          configuration.setDatabaseId(this.databaseIdProvider.getDatabaseId(this.dataSource));
        } catch (SQLException e) {
          throw new NestedIOException("Failed getting a databaseId", e);
        }
      }
  
      if (this.cache != null) {
        configuration.addCache(this.cache);
      }
  
      if (xmlConfigBuilder != null) {
        try {
          xmlConfigBuilder.parse();
  
          if (LOGGER.isDebugEnabled()) {
            LOGGER.debug("Parsed configuration file: '" + this.configLocation + "'");
          }
        } catch (Exception ex) {
          throw new NestedIOException("Failed to parse config resource: " + this.configLocation, ex);
        } finally {
          ErrorContext.instance().reset();
        }
      }
  //如果transactionFactory是空的话 new一个 Spring的事务工厂
      if (this.transactionFactory == null) {
        this.transactionFactory = new SpringManagedTransactionFactory();
      }
  	//这里很重要 将spring的环境对象 和事务工厂还有数据源对象注册到cfg类中
      configuration.setEnvironment(new Environment(this.environment, this.transactionFactory, this.dataSource));
  	//this.mapperLocation=application.properties中配置的mybatis.mapper-locations
  	//拿到mapper文件的配置类
      if (!isEmpty(this.mapperLocations)) {
        for (Resource mapperLocation : this.mapperLocations) {
          if (mapperLocation == null) {
            continue;
          }
  
          try {
          //构建xmlMapperBuilder
            XMLMapperBuilder xmlMapperBuilder = new XMLMapperBuilder(mapperLocation.getInputStream(),
                configuration, mapperLocation.toString(), configuration.getSqlFragments());
                //
            xmlMapperBuilder.parse();
          } catch (Exception e) {
            throw new NestedIOException("Failed to parse mapping resource: '" + mapperLocation + "'", e);
          } finally {
            ErrorContext.instance().reset();
          }
  
          if (LOGGER.isDebugEnabled()) {
            LOGGER.debug("Parsed mapper file: '" + mapperLocation + "'");
          }
        }
      } else {
        if (LOGGER.isDebugEnabled()) {
          LOGGER.debug("Property 'mapperLocations' was not specified or no matching resources found");
        }
      }
  	//最后创建sqlsessionFactory
      return this.sqlSessionFactoryBuilder.build(configuration);
    }
  
  ```

  **configuration.getTypeAliasRegistry().registerAliases的操作**

  ```
   public void registerAliases(String packageName, Class<?> superType){
      ResolverUtil<Class<?>> resolverUtil = new ResolverUtil<Class<?>>();
      resolverUtil.find(new ResolverUtil.IsA(superType), packageName);
      Set<Class<? extends Class<?>>> typeSet = resolverUtil.getClasses();
      for(Class<?> type : typeSet){
        // Ignore inner classes and interfaces (including package-info.java)
        // Skip also inner classes. See issue #6
        if (!type.isAnonymousClass() && !type.isInterface() && !type.isMemberClass()) {
          //注册别名
          registerAlias(type);
        }
      }
    }
    //上面代码调到这里
    public void registerAlias(Class<?> type) {
      String alias = type.getSimpleName();
      //先获得class上的注解 
      Alias aliasAnnotation = type.getAnnotation(Alias.class);
      if (aliasAnnotation != null) {
        //如果有的话拿到注解的值
        alias = aliasAnnotation.value();
      } 
      //
      registerAlias(alias, type);
    }
    //再跳到这里
    public void registerAlias(String alias, Class<?> value) {
      if (alias == null) {
        throw new TypeException("The parameter alias cannot be null");
      }
      // issue #748
      String key = alias.toLowerCase(Locale.ENGLISH);
      if (TYPE_ALIASES.containsKey(key) && TYPE_ALIASES.get(key) != null && !TYPE_ALIASES.get(key).equals(value)) {
        throw new TypeException("The alias '" + alias + "' is already mapped to the value '" + TYPE_ALIASES.get(key).getName() + "'.");
      }
      //这里key=userroledto value=class com.convergence.domain.UserRoleDTO
      TYPE_ALIASES.put(key, value);
    }
  ```

  **进行解析**

  ```
  public void parse() {
      if (!configuration.isResourceLoaded(resource)) {
        configurationElement(parser.evalNode("/mapper"));
        configuration.addLoadedResource(resource);
        //绑定namespace
        bindMapperForNamespace();
      }
  	//解析resultMaps
      parsePendingResultMaps();
      parsePendingCacheRefs();
      parsePendingStatements();
    }
    //解析xml文件
    private void configurationElement(XNode context) {
      try {
        String namespace = context.getStringAttribute("namespace");
        if (namespace == null || namespace.equals("")) {
          throw new BuilderException("Mapper's namespace cannot be empty");
        }
        builderAssistant.setCurrentNamespace(namespace);
        //这里的context就是xml文件内容了
        cacheRefElement(context.evalNode("cache-ref"));
        cacheElement(context.evalNode("cache"));
        parameterMapElement(context.evalNodes("/mapper/parameterMap"));
        resultMapElements(context.evalNodes("/mapper/resultMap"));
        sqlElement(context.evalNodes("/mapper/sql"));
        buildStatementFromContext(context.evalNodes("select|insert|update|delete"));
      } catch (Exception e) {
        throw new BuilderException("Error parsing Mapper XML. Cause: " + e, e);
      }
    }
    //将mapper.java和xml进行绑定
    private void bindMapperForNamespace() {
      String namespace = builderAssistant.getCurrentNamespace();
      if (namespace != null) {
        Class<?> boundType = null;
        try {
          boundType = Resources.classForName(namespace);
        } catch (ClassNotFoundException e) {
          //ignore, bound type is not required
        }
        if (boundType != null) {
          if (!configuration.hasMapper(boundType)) {
            // Spring may not know the real resource name so we set a flag
            // to prevent loading again this resource from the mapper interface
            // look at MapperAnnotationBuilder#loadXmlResource
            configuration.addLoadedResource("namespace:" + namespace);
            configuration.addMapper(boundType);
          }
        }
      }
    }
    //解析resultmap
   private void parsePendingResultMaps() {
      Collection<ResultMapResolver> incompleteResultMaps = configuration.getIncompleteResultMaps();
      synchronized (incompleteResultMaps) {
        Iterator<ResultMapResolver> iter = incompleteResultMaps.iterator();
        while (iter.hasNext()) {
          try {
            iter.next().resolve();
            iter.remove();
          } catch (IncompleteElementException e) {
            // ResultMap is still missing a resource...
          }
        }
      }
    }
    //缓存解析
    private void parsePendingCacheRefs() {
      Collection<CacheRefResolver> incompleteCacheRefs = configuration.getIncompleteCacheRefs();
      synchronized (incompleteCacheRefs) {
        Iterator<CacheRefResolver> iter = incompleteCacheRefs.iterator();
        while (iter.hasNext()) {
          try {
            iter.next().resolveCacheRef();
            iter.remove();
          } catch (IncompleteElementException e) {
            // Cache ref is still missing a resource...
          }
        }
      }
    }
    //statment解析
      private void parsePendingStatements() {
      Collection<XMLStatementBuilder> incompleteStatements = configuration.getIncompleteStatements();
      synchronized (incompleteStatements) {
        Iterator<XMLStatementBuilder> iter = incompleteStatements.iterator();
        while (iter.hasNext()) {
          try {
            iter.next().parseStatementNode();
            iter.remove();
          } catch (IncompleteElementException e) {
            // Statement is still missing a resource...
          }
        }
      }
    }
  ```

  **上面的工厂实例完了 进入sqlsessiontemplate实例**

  ```
    //这里需要上面构建的sqlSessionFactory
    @Bean
    @ConditionalOnMissingBean
    public SqlSessionTemplate sqlSessionTemplate(SqlSessionFactory sqlSessionFactory) {
      ExecutorType executorType = this.properties.getExecutorType();
      if (executorType != null) {
        return new SqlSessionTemplate(sqlSessionFactory, executorType);
      } else {
        return new SqlSessionTemplate(sqlSessionFactory);
      }
    }
  ```

  **构建SqlSessionTemplate**

  ```
   public class SqlSessionTemplate implements SqlSession, DisposableBean {
   ...
   /**
     * Constructs a Spring managed {@code SqlSession} with the given
     * {@code SqlSessionFactory} and {@code ExecutorType}.
     * A custom {@code SQLExceptionTranslator} can be provided as an
     * argument so any {@code PersistenceException} thrown by MyBatis
     * can be custom translated to a {@code RuntimeException}
     * The {@code SQLExceptionTranslator} can also be null and thus no
     * exception translation will be done and MyBatis exceptions will be
     * thrown
     *
     * @param sqlSessionFactory
     * @param executorType
     * @param exceptionTranslator
     */
    public SqlSessionTemplate(SqlSessionFactory sqlSessionFactory, ExecutorType executorType,
        PersistenceExceptionTranslator exceptionTranslator) {
  
      notNull(sqlSessionFactory, "Property 'sqlSessionFactory' is required");
      notNull(executorType, "Property 'executorType' is required");
  
      this.sqlSessionFactory = sqlSessionFactory;
      this.executorType = executorType;
      this.exceptionTranslator = exceptionTranslator;
      //获得一个sqlsession代理类 里面填充了sqlSessionInterceptor
      this.sqlSessionProxy = (SqlSession) newProxyInstance(
          SqlSessionFactory.class.getClassLoader(),
          new Class[] { SqlSession.class },
          new SqlSessionInterceptor());
    }
    ...
    }
  ```

  **这里说下ExecutorType **

  * **simple 默认的 **
  * **reuse 这个类型不做特殊的事情，它只为每个语句创建一个 PreparedStatement。**
  * **batch 批处理语句使用 需要关闭autocommit      **

  ```
  public enum ExecutorType {
    SIMPLE, REUSE, BATCH
  }
  ```

  **在创建sqlSessionTemplate的时候会创建一个SqlSessionInterceptor**

  ```
    private class SqlSessionInterceptor implements InvocationHandler {
  ...
      @Override
      public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
      //首先获取到sqlsession
        SqlSession sqlSession = getSqlSession(
            SqlSessionTemplate.this.sqlSessionFactory,
            SqlSessionTemplate.this.executorType,
            SqlSessionTemplate.this.exceptionTranslator);
        try {
        //进行involke 拿到具体的结果集
          Object result = method.invoke(sqlSession, args);
          //这个if判断：    return (holder != null) && (holder.getSqlSession() == session);
          if (!isSqlSessionTransactional(sqlSession, SqlSessionTemplate.this.sqlSessionFactory)) {
            // force commit even on non-dirty sessions because some databases require
            // a commit/rollback before calling close()
            //然后commit
            sqlSession.commit(true);
          }
          return result;
        } catch (Throwable t) {
          Throwable unwrapped = unwrapThrowable(t);
          if (SqlSessionTemplate.this.exceptionTranslator != null && unwrapped instanceof PersistenceException) {
            // release the connection to avoid a deadlock if the translator is no loaded. See issue #22
            closeSqlSession(sqlSession, SqlSessionTemplate.this.sqlSessionFactory);
            sqlSession = null;
            Throwable translated = SqlSessionTemplate.this.exceptionTranslator.translateExceptionIfPossible((PersistenceException) unwrapped);
            if (translated != null) {
              unwrapped = translated;
            }
          }
          throw unwrapped;
        } finally {
          if (sqlSession != null) {
          //关闭会话
            closeSqlSession(sqlSession, SqlSessionTemplate.this.sqlSessionFactory);
          }
        }
      }
    }
    }
  ```

  **getSqlSession:**

  ```
    public static SqlSession getSqlSession(SqlSessionFactory sessionFactory, ExecutorType executorType, PersistenceExceptionTranslator exceptionTranslator) {
  
      notNull(sessionFactory, NO_SQL_SESSION_FACTORY_SPECIFIED);
      notNull(executorType, NO_EXECUTOR_TYPE_SPECIFIED);
  	//holder
      SqlSessionHolder holder = (SqlSessionHolder) 
      //事务管理器拿到holder
      TransactionSynchronizationManager.getResource(sessionFactory);
  	//获得session
      SqlSession session = sessionHolder(executorType, holder);
      if (session != null) {
        return session;
      }
  
      if (LOGGER.isDebugEnabled()) {
        LOGGER.debug("Creating a new SqlSession");
      }
  	//工厂打开session
      session = sessionFactory.openSession(executorType);
  	//注册session,工厂到holder
      registerSessionHolder(sessionFactory, executorType, exceptionTranslator, session);
  
      return session;
    }
  ```

  **registerSessionHolder:**

  ```
    private static void registerSessionHolder(SqlSessionFactory sessionFactory, ExecutorType executorType,
        PersistenceExceptionTranslator exceptionTranslator, SqlSession session) {
      SqlSessionHolder holder;
      if (TransactionSynchronizationManager.isSynchronizationActive()) {
        Environment environment = sessionFactory.getConfiguration().getEnvironment();
  	  //如果当前的事务工厂是spring的事务管理工厂的实例的话
        if (environment.getTransactionFactory() instanceof SpringManagedTransactionFactory) {
          if (LOGGER.isDebugEnabled()) {
            LOGGER.debug("Registering transaction synchronization for SqlSession [" + session + "]");
          }
  	   //创建holder
          holder = new SqlSessionHolder(session, executorType, exceptionTranslator);
          //底下是事务的一系列管理操作
          TransactionSynchronizationManager.bindResource(sessionFactory, holder);
          TransactionSynchronizationManager.registerSynchronization(new SqlSessionSynchronization(holder, sessionFactory));
          holder.setSynchronizedWithTransaction(true);
          holder.requested();
        } else {
          if (TransactionSynchronizationManager.getResource(environment.getDataSource()) == null) {
            if (LOGGER.isDebugEnabled()) {
              LOGGER.debug("SqlSession [" + session + "] was not registered for synchronization because DataSource is not transactional");
            }
          } else {
            throw new TransientDataAccessResourceException(
                "SqlSessionFactory must be using a SpringManagedTransactionFactory in order to use Spring transaction synchronization");
          }
        }
      } else {
        if (LOGGER.isDebugEnabled()) {
          LOGGER.debug("SqlSession [" + session + "] was not registered for synchronization because synchronization is not active");
        }
      }
  }
  ```

  **具体的sql执行过程**

  

  **mybatis中的每个dao都会继承SqlSessionDaoSupport，在于spring集成的时候，需要依赖spring-mybatis这个jar，而这个SqlSessionDaoSupport类就在这个jar中**

  **这是个抽象类。**

  ```
  package org.mybatis.spring.support;
  
  import static org.springframework.util.Assert.notNull;
  
  import org.apache.ibatis.session.SqlSession;
  import org.apache.ibatis.session.SqlSessionFactory;
  import org.mybatis.spring.SqlSessionTemplate;
  import org.springframework.dao.support.DaoSupport;
  
  /**
   * Convenient super class for MyBatis SqlSession data access objects.
   * It gives you access to the template which can then be used to execute SQL methods.
   * <p>
   * This class needs a SqlSessionTemplate or a SqlSessionFactory.
   * If both are set the SqlSessionFactory will be ignored.
   * <p>
   * {code Autowired} was removed from setSqlSessionTemplate and setSqlSessionFactory
   * in version 1.2.0.
   * 
   * @author Putthibong Boonbong
   *
   * @see #setSqlSessionFactory
   * @see #setSqlSessionTemplate
   * @see SqlSessionTemplate
   */
  public abstract class SqlSessionDaoSupport extends DaoSupport {
  
    private SqlSession sqlSession;
  
    private boolean externalSqlSession;
  
    public void setSqlSessionFactory(SqlSessionFactory sqlSessionFactory) {
      if (!this.externalSqlSession) {
        this.sqlSession = new SqlSessionTemplate(sqlSessionFactory);
      }
    }
  
    public void setSqlSessionTemplate(SqlSessionTemplate sqlSessionTemplate) {
      this.sqlSession = sqlSessionTemplate;
      this.externalSqlSession = true;
    }
  
    /**
     * Users should use this method to get a SqlSession to call its statement methods
     * This is SqlSession is managed by spring. Users should not commit/rollback/close it
     * because it will be automatically done.
     *
     * @return Spring managed thread safe SqlSession
     */
    public SqlSession getSqlSession() {
      return this.sqlSession;
    }
  
    /**
     * {@inheritDoc}
     */
    @Override
    protected void checkDaoConfig() {
      notNull(this.sqlSession, "Property 'sqlSessionFactory' or 'sqlSessionTemplate' are required");
    }
  
  }
  ```

  **那么它的父类DaoSupport什么时候**

  **注意实现了InitializingBean这个是由spring管理的**

  ```
  public abstract class AbstractAutowireCapableBeanFactory extends AbstractBeanFactory
  		implements AutowireCapableBeanFactory {
  		//
  	/**
  	 * Give a bean a chance to react now all its properties are set,
  	 * and a chance to know about its owning bean factory (this object).
  	 * This means checking whether the bean implements InitializingBean or defines
  	 * a custom init method, and invoking the necessary callback(s) if it does.
  	 * @param beanName the bean name in the factory (for debugging purposes)
  	 * @param bean the new bean instance we may need to initialize
  	 * @param mbd the merged bean definition that the bean was created with
  	 * (can also be {@code null}, if given an existing bean instance)
  	 * @throws Throwable if thrown by init methods or by the invocation process
  	 * @see #invokeCustomInitMethod
  	 */
  	 //根据bean的name来反射加载初始方法 当bean DataSourceInitializerInvoker被加载时候会触发事件驱动 去构建sqlsessionFactory
  	 //当bean=MapperFactoryBean的时候 先走父类DaoSupport的afterPropertiesSet
  	protected void invokeInitMethods(String beanName, final Object bean, @Nullable RootBeanDefinition mbd)
  			throws Throwable {
  
  		boolean isInitializingBean = (bean instanceof InitializingBean);
  		if (isInitializingBean && (mbd == null || !mbd.isExternallyManagedInitMethod("afterPropertiesSet"))) {
  			if (logger.isDebugEnabled()) {
  				logger.debug("Invoking afterPropertiesSet() on bean with name '" + beanName + "'");
  			}
  			if (System.getSecurityManager() != null) {
  				try {
  					AccessController.doPrivileged((PrivilegedExceptionAction<Object>) () -> {
  						((InitializingBean) bean).afterPropertiesSet();
  						return null;
  					}, getAccessControlContext());
  				}
  				catch (PrivilegedActionException pae) {
  					throw pae.getException();
  				}
  			}
  			else {
  			//在这里会初始化dao
  				((InitializingBean) bean).afterPropertiesSet();
  			}
  		}
  
  		if (mbd != null && bean.getClass() != NullBean.class) {
  			String initMethodName = mbd.getInitMethodName();
  			if (StringUtils.hasLength(initMethodName) &&
  					!(isInitializingBean && "afterPropertiesSet".equals(initMethodName)) &&	
  					!mbd.isExternallyManagedInitMethod(initMethodName)) {
  				invokeCustomInitMethod(beanName, bean, mbd);
  			}
  		}
  	}
  
  ```

  **当bean=MapperFactoryBean的时候 先走父类DaoSupport的afterPropertiesSet：**

  ```
  public abstract class DaoSupport implements InitializingBean {
  ...
  	@Override
  	public final void afterPropertiesSet() throws IllegalArgumentException, BeanInitializationException {
  		// Let abstract subclasses check their configuration.
  		//检查dao的cfg
  		checkDaoConfig();
  
  		// Let concrete implementations initialize themselves.
  		try {
  		//初始化加载dao 父类空实现
  			initDao();
  		}
  		catch (Exception ex) {
  			throw new BeanInitializationException("Initialization of DAO failed", ex);
  		}
  	}
  ```

  

  **具体实现MapperFactoryBean：**

  ```
  package org.mybatis.spring.mapper;
  
  import static org.springframework.util.Assert.notNull;
  
  import org.apache.ibatis.executor.ErrorContext;
  import org.apache.ibatis.session.Configuration;
  import org.mybatis.spring.SqlSessionTemplate;
  import org.mybatis.spring.support.SqlSessionDaoSupport;
  import org.springframework.beans.factory.FactoryBean;
  
  /**
   * BeanFactory that enables injection of MyBatis mapper interfaces. It can be set up with a
   * SqlSessionFactory or a pre-configured SqlSessionTemplate.
   * <p>
   * Sample configuration:
   *
   * <pre class="code">
   * {@code
   *   <bean id="baseMapper" class="org.mybatis.spring.mapper.MapperFactoryBean" abstract="true" lazy-init="true">
   *     <property name="sqlSessionFactory" ref="sqlSessionFactory" />
   *   </bean>
   *
   *   <bean id="oneMapper" parent="baseMapper">
   *     <property name="mapperInterface" value="my.package.MyMapperInterface" />
   *   </bean>
   *
   *   <bean id="anotherMapper" parent="baseMapper">
   *     <property name="mapperInterface" value="my.package.MyAnotherMapperInterface" />
   *   </bean>
   * }
   * </pre>
   * <p>
   * Note that this factory can only inject <em>interfaces</em>, not concrete classes.
   *
   * @author Eduardo Macarron
   *
   * @see SqlSessionTemplate
   */
  public class MapperFactoryBean<T> extends SqlSessionDaoSupport implements FactoryBean<T> {
  
    private Class<T> mapperInterface;
  
    private boolean addToConfig = true;
  
    public MapperFactoryBean() {
      //intentionally empty 
    }
    
    public MapperFactoryBean(Class<T> mapperInterface) {
      this.mapperInterface = mapperInterface;
    }
  
    /**
     * {@inheritDoc}
     */
    @Override
    protected void checkDaoConfig() {
    //进入父类DaoSupport的checkDaoConfig方法
      super.checkDaoConfig();
  
      notNull(this.mapperInterface, "Property 'mapperInterface' is required");
  	//获取到mybatis的configuration
      Configuration configuration = getSqlSession().getConfiguration();
      //如果配置中没有当前执行sql的dao
      if (this.addToConfig && !configuration.hasMapper(this.mapperInterface)) {
        try {
        //加入该dao到配置类中
          configuration.addMapper(this.mapperInterface);
        } catch (Exception e) {
          logger.error("Error while adding the mapper '" + this.mapperInterface + "' to configuration.", e);
          throw new IllegalArgumentException(e);
        } finally {
          ErrorContext.instance().reset();
        }
      }
    }
  
    /**
     * {@inheritDoc}
     */
    @Override
    public T getObject() throws Exception {
      return getSqlSession().getMapper(this.mapperInterface);
    }
  
    /**
     * {@inheritDoc}
     */
    @Override
    public Class<T> getObjectType() {
      return this.mapperInterface;
    }
  
    /**
     * {@inheritDoc}
     */
    @Override
    public boolean isSingleton() {
      return true;
    }
  
    //------------- mutators --------------
  
    /**
     * Sets the mapper interface of the MyBatis mapper
     *
     * @param mapperInterface class of the interface
     */
    public void setMapperInterface(Class<T> mapperInterface) {
      this.mapperInterface = mapperInterface;
    }
  
    /**
     * Return the mapper interface of the MyBatis mapper
     *
     * @return class of the interface
     */
    public Class<T> getMapperInterface() {
      return mapperInterface;
    }
  
    /**
     * If addToConfig is false the mapper will not be added to MyBatis. This means
     * it must have been included in mybatis-config.xml.
     * <p/>
     * If it is true, the mapper will be added to MyBatis in the case it is not already
     * registered.
     * <p/>
     * By default addToCofig is true.
     *
     * @param addToConfig
     */
    public void setAddToConfig(boolean addToConfig) {
      this.addToConfig = addToConfig;
    }
  
    /**
     * Return the flag for addition into MyBatis config.
     *
     * @return true if the mapper will be added to MyBatis in the case it is not already
     * registered.
     */
    public boolean isAddToConfig() {
      return addToConfig;
    }
  }
  
  ```

  **走完上面的代码，console打印日志：**

  ```
  2018-11-07 23:51:11.500 INFO [restartedMain][Jdk14Logger.java:99] - Bean 'resourceDao' of type [org.mybatis.spring.mapper.MapperFactoryBean] is not eligible for getting processed by all BeanPostProcessors (for example: not eligible for auto-proxying)
  ```

  **那么底下会创建bean **

  ```
  public abstract class AbstractAutowireCapableBeanFactory extends AbstractBeanFactory
  		implements AutowireCapableBeanFactory {
  	...
  	@Override
  	protected Object createBean(String beanName, RootBeanDefinition mbd, @Nullable Object[] args)
  			throws BeanCreationException {
  
  		if (logger.isDebugEnabled()) {
  			logger.debug("Creating instance of bean '" + beanName + "'");
  		}
  		RootBeanDefinition mbdToUse = mbd;
  
  		// Make sure bean class is actually resolved at this point, and
  		// clone the bean definition in case of a dynamically resolved Class
  		// which cannot be stored in the shared merged bean definition.
  		Class<?> resolvedClass = resolveBeanClass(mbd, beanName);
  		if (resolvedClass != null && !mbd.hasBeanClass() && mbd.getBeanClassName() != null) {
  			mbdToUse = new RootBeanDefinition(mbd);
  			mbdToUse.setBeanClass(resolvedClass);
  		}
  
  		// Prepare method overrides.
  		try {
  			mbdToUse.prepareMethodOverrides();
  		}
  		catch (BeanDefinitionValidationException ex) {
  			throw new BeanDefinitionStoreException(mbdToUse.getResourceDescription(),
  					beanName, "Validation of method overrides failed", ex);
  		}
  
  		try {
  			// Give BeanPostProcessors a chance to return a proxy instead of the target bean instance.
  			Object bean = resolveBeforeInstantiation(beanName, mbdToUse);
  			if (bean != null) {
  				return bean;
  			}
  		}
  		catch (Throwable ex) {
  			throw new BeanCreationException(mbdToUse.getResourceDescription(), beanName,
  					"BeanPostProcessor before instantiation of bean failed", ex);
  		}
  
  		try {
  		//创建bean
  			Object beanInstance = doCreateBean(beanName, mbdToUse, args);
  			if (logger.isDebugEnabled()) {
  				logger.debug("Finished creating instance of bean '" + beanName + "'");
  			}
  			return beanInstance;
  		}
  		catch (BeanCreationException | ImplicitlyAppearedSingletonException ex) {
  			// A previously detected exception with proper bean creation context already,
  			// or illegal singleton state to be communicated up to DefaultSingletonBeanRegistry.
  			throw ex;
  		}
  		catch (Throwable ex) {
  			throw new BeanCreationException(
  					mbdToUse.getResourceDescription(), beanName, "Unexpected exception during bean creation", ex);
  		}
  	}
  	...
  	//doCreateBean 创建bean
  		protected Object doCreateBean(final String beanName, final RootBeanDefinition mbd, final @Nullable Object[] args)
  			throws BeanCreationException {
  
  		// Instantiate the bean.
  		BeanWrapper instanceWrapper = null;
  		if (mbd.isSingleton()) {
  			instanceWrapper = this.factoryBeanInstanceCache.remove(beanName);
  		}
  		if (instanceWrapper == null) {
  			instanceWrapper = createBeanInstance(beanName, mbd, args);
  		}
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
  			if (logger.isDebugEnabled()) {
  				logger.debug("Eagerly caching bean '" + beanName +
  						"' to allow for resolving potential circular references");
  			}
  			addSingletonFactory(beanName, () -> getEarlyBeanReference(beanName, mbd, bean));
  		}
  
  		// Initialize the bean instance.
  		Object exposedObject = bean;
  		try {
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
  
  		// Register bean as disposable 
  		//将这个dao注册为一次性的bean
  		try {
  			registerDisposableBeanIfNecessary(beanName, bean, mbd);
  		}
  		catch (BeanDefinitionValidationException ex) {
  			throw new BeanCreationException(
  					mbd.getResourceDescription(), beanName, "Invalid destruction signature", ex);
  		}
  
  		return exposedObject;
  	}
  
  	}
  	
  ```

  **//注册为单例**

  ```
  	public abstract class AbstractBeanFactory extends FactoryBeanRegistrySupport implements ConfigurableBeanFactory {
  ...
  	/**
  	 * Add the given bean to the list of disposable beans in this factory,
  	 * registering its DisposableBean interface and/or the given destroy method
  	 * to be called on factory shutdown (if applicable). Only applies to singletons.
  	 * @param beanName the name of the bean
  	 * @param bean the bean instance
  	 * @param mbd the bean definition for the bean
  	 * @see RootBeanDefinition#isSingleton
  	 * @see RootBeanDefinition#getDependsOn
  	 * @see #registerDisposableBean
  	 * @see #registerDependentBean
  	 */
  	protected void registerDisposableBeanIfNecessary(String beanName, Object bean, RootBeanDefinition mbd) {
  		AccessControlContext acc = (System.getSecurityManager() != null ? getAccessControlContext() : null);
  		if (!mbd.isPrototype() && requiresDestruction(bean, mbd)) {
  			if (mbd.isSingleton()) {
  				// Register a DisposableBean implementation that performs all destruction
  				// work for the given bean: DestructionAwareBeanPostProcessors,
  				// DisposableBean interface, custom destroy method.
  				registerDisposableBean(beanName,
  						new DisposableBeanAdapter(bean, beanName, mbd, getBeanPostProcessors(), acc));
  			}
  			else {
  				// A bean with a custom scope...
  				Scope scope = this.scopes.get(mbd.getScope());
  				if (scope == null) {
  					throw new IllegalStateException("No Scope registered for scope name '" + mbd.getScope() + "'");
  				}
  				scope.registerDestructionCallback(beanName,
  						new DisposableBeanAdapter(bean, beanName, mbd, getBeanPostProcessors(), acc));
  			}
  		}
  	}
  	...
  	}
  ```

  **当要执行dao层的sql操作时候**

  ```
  org.apache.ibatis.binding.MapperProxy@14e19901
  ```

  **这个是一个代理类**

  ```
  /**
   *    Copyright 2009-2017 the original author or authors.
   *
   *    Licensed under the Apache License, Version 2.0 (the "License");
   *    you may not use this file except in compliance with the License.
   *    You may obtain a copy of the License at
   *
   *       http://www.apache.org/licenses/LICENSE-2.0
   *
   *    Unless required by applicable law or agreed to in writing, software
   *    distributed under the License is distributed on an "AS IS" BASIS,
   *    WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
   *    See the License for the specific language governing permissions and
   *    limitations under the License.
   */
  package org.apache.ibatis.binding;
  
  import java.io.Serializable;
  import java.lang.invoke.MethodHandles;
  import java.lang.reflect.Constructor;
  import java.lang.reflect.InvocationHandler;
  import java.lang.reflect.Method;
  import java.lang.reflect.Modifier;
  import java.util.Map;
  
  import org.apache.ibatis.lang.UsesJava7;
  import org.apache.ibatis.reflection.ExceptionUtil;
  import org.apache.ibatis.session.SqlSession;
  
  /**
   * @author Clinton Begin
   * @author Eduardo Macarron
   */
  public class MapperProxy<T> implements InvocationHandler, Serializable {
  
    private static final long serialVersionUID = -6424540398559729838L;
    private final SqlSession sqlSession;
    private final Class<T> mapperInterface;
    private final Map<Method, MapperMethod> methodCache;
  
    public MapperProxy(SqlSession sqlSession, Class<T> mapperInterface, Map<Method, MapperMethod> methodCache) {
      this.sqlSession = sqlSession;
      this.mapperInterface = mapperInterface;
      this.methodCache = methodCache;
    }
    //具体的sql操作会走这里
    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
      try {
      //判断是否是生命的类
        if (Object.class.equals(method.getDeclaringClass())) {
          return method.invoke(this, args);
          //是否是原生方法
        } else if (isDefaultMethod(method)) {
          return invokeDefaultMethod(proxy, method, args);
        }
      } catch (Throwable t) {
        throw ExceptionUtil.unwrapThrowable(t);
      }
      //将具体的sql操作缓存并返回
      final MapperMethod mapperMethod = cachedMapperMethod(method);
      //进行执行
      return mapperMethod.execute(sqlSession, args);
    }
   //将具体的sql操作method缓存
    private MapperMethod cachedMapperMethod(Method method) {
      MapperMethod mapperMethod = methodCache.get(method);
      if (mapperMethod == null) {
        mapperMethod = new MapperMethod(mapperInterface, method, sqlSession.getConfiguration());
        methodCache.put(method, mapperMethod);
      }
      return mapperMethod;
    }
  
    @UsesJava7
    private Object invokeDefaultMethod(Object proxy, Method method, Object[] args)
        throws Throwable {
      final Constructor<MethodHandles.Lookup> constructor = MethodHandles.Lookup.class
          .getDeclaredConstructor(Class.class, int.class);
      if (!constructor.isAccessible()) {
        constructor.setAccessible(true);
      }
      final Class<?> declaringClass = method.getDeclaringClass();
      return constructor
          .newInstance(declaringClass,
              MethodHandles.Lookup.PRIVATE | MethodHandles.Lookup.PROTECTED
                  | MethodHandles.Lookup.PACKAGE | MethodHandles.Lookup.PUBLIC)
          .unreflectSpecial(method, declaringClass).bindTo(proxy).invokeWithArguments(args);
    }
  
    /**
     * Backport of java.lang.reflect.Method#isDefault()
     */
    private boolean isDefaultMethod(Method method) {
      return ((method.getModifiers()
          & (Modifier.ABSTRACT | Modifier.PUBLIC | Modifier.STATIC)) == Modifier.PUBLIC)
          && method.getDeclaringClass().isInterface();
    }
  }
  
  ```

  **MapperMethod:**

  ```
  public class MapperMethod {
  
    private final SqlCommand command;
    private final MethodSignature method;
  
    public MapperMethod(Class<?> mapperInterface, Method method, Configuration config) {
      this.command = new SqlCommand(config, mapperInterface, method);
      this.method = new MethodSignature(config, mapperInterface, method);
    }
   //这里进行执行sql
    public Object execute(SqlSession sqlSession, Object[] args) {
      Object result;
      //判断crud操作
      switch (command.getType()) {
        case INSERT: {
      	Object param = method.convertArgsToSqlCommandParam(args);
          result = rowCountResult(sqlSession.insert(command.getName(), param));
          break;
        }
        case UPDATE: {
          Object param = method.convertArgsToSqlCommandParam(args);
          result = rowCountResult(sqlSession.update(command.getName(), param));
          break;
        }
        case DELETE: {
          Object param = method.convertArgsToSqlCommandParam(args);
          result = rowCountResult(sqlSession.delete(command.getName(), param));
          break;
        }
        case SELECT:
          if (method.returnsVoid() && method.hasResultHandler()) {
            executeWithResultHandler(sqlSession, args);
            result = null;
          } else if (method.returnsMany()) {
          //这里是一个many的result跳到这里
            result = executeForMany(sqlSession, args);
          } else if (method.returnsMap()) {
            result = executeForMap(sqlSession, args);
          } else if (method.returnsCursor()) {
            result = executeForCursor(sqlSession, args);
          } else {
            Object param = method.convertArgsToSqlCommandParam(args);
            result = sqlSession.selectOne(command.getName(), param);
          }
          break;
        case FLUSH:
          result = sqlSession.flushStatements();
          break;
        default:
          throw new BindingException("Unknown execution method for: " + command.getName());
      }
      if (result == null && method.getReturnType().isPrimitive() && !method.returnsVoid()) {
        throw new BindingException("Mapper method '" + command.getName() 
            + " attempted to return null from a method with a primitive return type (" + method.getReturnType() + ").");
      }
      return result;
    }
  
    private Object rowCountResult(int rowCount) {
      final Object result;
      if (method.returnsVoid()) {
        result = null;
      } else if (Integer.class.equals(method.getReturnType()) || Integer.TYPE.equals(method.getReturnType())) {
        result = rowCount;
      } else if (Long.class.equals(method.getReturnType()) || Long.TYPE.equals(method.getReturnType())) {
        result = (long)rowCount;
      } else if (Boolean.class.equals(method.getReturnType()) || Boolean.TYPE.equals(method.getReturnType())) {
        result = rowCount > 0;
      } else {
        throw new BindingException("Mapper method '" + command.getName() + "' has an unsupported return type: " + method.getReturnType());
      }
      return result;
    }
  
    private void executeWithResultHandler(SqlSession sqlSession, Object[] args) {
      MappedStatement ms = sqlSession.getConfiguration().getMappedStatement(command.getName());
      if (void.class.equals(ms.getResultMaps().get(0).getType())) {
        throw new BindingException("method " + command.getName() 
            + " needs either a @ResultMap annotation, a @ResultType annotation," 
            + " or a resultType attribute in XML so a ResultHandler can be used as a parameter.");
      }
      Object param = method.convertArgsToSqlCommandParam(args);
      if (method.hasRowBounds()) {
        RowBounds rowBounds = method.extractRowBounds(args);
        sqlSession.select(command.getName(), param, rowBounds, method.extractResultHandler(args));
      } else {
        sqlSession.select(command.getName(), param, method.extractResultHandler(args));
      }
    }
  	//多结果集的跳到这里
    private <E> Object executeForMany(SqlSession sqlSession, Object[] args) {
      List<E> result;
      Object param = method.convertArgsToSqlCommandParam(args);
      if (method.hasRowBounds()) {
        RowBounds rowBounds = method.extractRowBounds(args);
        //重点 交给sqlSession进行处理
        result = sqlSession.<E>selectList(command.getName(), param, rowBounds);
      } else {
        result = sqlSession.<E>selectList(command.getName(), param);
      }
      // issue #510 Collections & arrays support
      if (!method.getReturnType().isAssignableFrom(result.getClass())) {
        if (method.getReturnType().isArray()) {
          return convertToArray(result);
        } else {
          return convertToDeclaredCollection(sqlSession.getConfiguration(), result);
        }
      }
      return result;
    }
  
    private <T> Cursor<T> executeForCursor(SqlSession sqlSession, Object[] args) {
      Cursor<T> result;
      Object param = method.convertArgsToSqlCommandParam(args);
      if (method.hasRowBounds()) {
        RowBounds rowBounds = method.extractRowBounds(args);
        result = sqlSession.<T>selectCursor(command.getName(), param, rowBounds);
      } else {
        result = sqlSession.<T>selectCursor(command.getName(), param);
      }
      return result;
    }
  
    private <E> Object convertToDeclaredCollection(Configuration config, List<E> list) {
      Object collection = config.getObjectFactory().create(method.getReturnType());
      MetaObject metaObject = config.newMetaObject(collection);
      metaObject.addAll(list);
      return collection;
    }
  
    @SuppressWarnings("unchecked")
    private <E> Object convertToArray(List<E> list) {
      Class<?> arrayComponentType = method.getReturnType().getComponentType();
      Object array = Array.newInstance(arrayComponentType, list.size());
      if (arrayComponentType.isPrimitive()) {
        for (int i = 0; i < list.size(); i++) {
          Array.set(array, i, list.get(i));
        }
        return array;
      } else {
        return list.toArray((E[])array);
      }
    }
  
    private <K, V> Map<K, V> executeForMap(SqlSession sqlSession, Object[] args) {
      Map<K, V> result;
      Object param = method.convertArgsToSqlCommandParam(args);
      if (method.hasRowBounds()) {
        RowBounds rowBounds = method.extractRowBounds(args);
        result = sqlSession.<K, V>selectMap(command.getName(), param, method.getMapKey(), rowBounds);
      } else {
        result = sqlSession.<K, V>selectMap(command.getName(), param, method.getMapKey());
      }
      return result;
    }
  
    public static class ParamMap<V> extends HashMap<String, V> {
  
      private static final long serialVersionUID = -2212268410512043556L;
  
      @Override
      public V get(Object key) {
        if (!super.containsKey(key)) {
          throw new BindingException("Parameter '" + key + "' not found. Available parameters are " + keySet());
        }
        return super.get(key);
      }
  
    }
  
    public static class SqlCommand {
  
      private final String name;
      private final SqlCommandType type;
  
      public SqlCommand(Configuration configuration, Class<?> mapperInterface, Method method) {
        final String methodName = method.getName();
        final Class<?> declaringClass = method.getDeclaringClass();
        MappedStatement ms = resolveMappedStatement(mapperInterface, methodName, declaringClass,
            configuration);
        if (ms == null) {
          if (method.getAnnotation(Flush.class) != null) {
            name = null;
            type = SqlCommandType.FLUSH;
          } else {
            throw new BindingException("Invalid bound statement (not found): "
                + mapperInterface.getName() + "." + methodName);
          }
        } else {
          name = ms.getId();
          type = ms.getSqlCommandType();
          if (type == SqlCommandType.UNKNOWN) {
            throw new BindingException("Unknown execution method for: " + name);
          }
        }
      }
  
      public String getName() {
        return name;
      }
  
      public SqlCommandType getType() {
        return type;
      }
  
      private MappedStatement resolveMappedStatement(Class<?> mapperInterface, String methodName,
          Class<?> declaringClass, Configuration configuration) {
        String statementId = mapperInterface.getName() + "." + methodName;
        if (configuration.hasStatement(statementId)) {
          return configuration.getMappedStatement(statementId);
        } else if (mapperInterface.equals(declaringClass)) {
          return null;
        }
        for (Class<?> superInterface : mapperInterface.getInterfaces()) {
          if (declaringClass.isAssignableFrom(superInterface)) {
            MappedStatement ms = resolveMappedStatement(superInterface, methodName,
                declaringClass, configuration);
            if (ms != null) {
              return ms;
            }
          }
        }
        return null;
      }
    }
  
    public static class MethodSignature {
  
      private final boolean returnsMany;
      private final boolean returnsMap;
      private final boolean returnsVoid;
      private final boolean returnsCursor;
      private final Class<?> returnType;
      private final String mapKey;
      private final Integer resultHandlerIndex;
      private final Integer rowBoundsIndex;
      private final ParamNameResolver paramNameResolver;
  
      public MethodSignature(Configuration configuration, Class<?> mapperInterface, Method method) {
        Type resolvedReturnType = TypeParameterResolver.resolveReturnType(method, mapperInterface);
        if (resolvedReturnType instanceof Class<?>) {
          this.returnType = (Class<?>) resolvedReturnType;
        } else if (resolvedReturnType instanceof ParameterizedType) {
          this.returnType = (Class<?>) ((ParameterizedType) resolvedReturnType).getRawType();
        } else {
          this.returnType = method.getReturnType();
        }
        this.returnsVoid = void.class.equals(this.returnType);
        this.returnsMany = (configuration.getObjectFactory().isCollection(this.returnType) || this.returnType.isArray());
        this.returnsCursor = Cursor.class.equals(this.returnType);
        this.mapKey = getMapKey(method);
        this.returnsMap = (this.mapKey != null);
        this.rowBoundsIndex = getUniqueParamIndex(method, RowBounds.class);
        this.resultHandlerIndex = getUniqueParamIndex(method, ResultHandler.class);
        this.paramNameResolver = new ParamNameResolver(configuration, method);
      }
  
      public Object convertArgsToSqlCommandParam(Object[] args) {
        return paramNameResolver.getNamedParams(args);
      }
  
      public boolean hasRowBounds() {
        return rowBoundsIndex != null;
      }
  
      public RowBounds extractRowBounds(Object[] args) {
        return hasRowBounds() ? (RowBounds) args[rowBoundsIndex] : null;
      }
  
      public boolean hasResultHandler() {
        return resultHandlerIndex != null;
      }
  
      public ResultHandler extractResultHandler(Object[] args) {
        return hasResultHandler() ? (ResultHandler) args[resultHandlerIndex] : null;
      }
  
      public String getMapKey() {
        return mapKey;
      }
  
      public Class<?> getReturnType() {
        return returnType;
      }
  
      public boolean returnsMany() {
        return returnsMany;
      }
  
      public boolean returnsMap() {
        return returnsMap;
      }
  
      public boolean returnsVoid() {
        return returnsVoid;
      }
  
      public boolean returnsCursor() {
        return returnsCursor;
      }
  
      private Integer getUniqueParamIndex(Method method, Class<?> paramType) {
        Integer index = null;
        final Class<?>[] argTypes = method.getParameterTypes();
        for (int i = 0; i < argTypes.length; i++) {
          if (paramType.isAssignableFrom(argTypes[i])) {
            if (index == null) {
              index = i;
            } else {
              throw new BindingException(method.getName() + " cannot have multiple " + paramType.getSimpleName() + " parameters");
            }
          }
        }
        return index;
      }
  
      private String getMapKey(Method method) {
        String mapKey = null;
        if (Map.class.isAssignableFrom(method.getReturnType())) {
          final MapKey mapKeyAnnotation = method.getAnnotation(MapKey.class);
          if (mapKeyAnnotation != null) {
            mapKey = mapKeyAnnotation.value();
          }
        }
        return mapKey;
      }
    }
  
  }
  
  ```

  **上述的sqlsession的操作跳到 DefaultSqlSession **

  ```
  public class DefaultSqlSession implements SqlSession {
  ...
    @Override
    public <E> List<E> selectList(String statement, Object parameter) {
      return this.selectList(statement, parameter, RowBounds.DEFAULT);
    }
    @Override
    public <E> List<E> selectList(String statement, Object parameter, RowBounds rowBounds) {
      try {
      //根据语句statement拿到MappedStatement
        MappedStatement ms = configuration.getMappedStatement(statement);
        //重点来了 executor执行器来执行具体的sql
        return ,executor.query(ms, wrapCollection(parameter), rowBounds, Executor.NO_RESULT_HANDLER);
      } catch (Exception e) {
        throw ExceptionFactory.wrapException("Error querying database.  Cause: " + e, e);
      } finally {
        ErrorContext.instance().reset();
      }
    }
    ...
    }
  ```

  **executor有两个实现：BaseExecutor 和CachingExecutor**

  **这里走CachingExecutor：**

  ```
  public class CachingExecutor implements Executor {
  ...
    @Override
    public <E> List<E> query(MappedStatement ms, Object parameterObject, RowBounds rowBounds, ResultHandler resultHandler) throws SQLException {
    //拿到sql boundSql里面存储有要执行的sql
      BoundSql boundSql = ms.getBoundSql(parameterObject);
      CacheKey key = createCacheKey(ms, parameterObject, rowBounds, boundSql);
      //返回具体的query方法的执行结果集
      return query(ms, parameterObject, rowBounds, resultHandler, key, boundSql);
    }
    //缓存sql
      @Override
    public CacheKey createCacheKey(MappedStatement ms, Object parameterObject, RowBounds rowBounds, BoundSql boundSql) {
        //没有的话走BaseExecutor
      return delegate.createCacheKey(ms, parameterObject, rowBounds, boundSql);
    }
    //具体的query方法
      @Override
    public <E> List<E> query(MappedStatement ms, Object parameterObject, RowBounds rowBounds, ResultHandler resultHandler, CacheKey key, BoundSql boundSql)
        throws SQLException {
      Cache cache = ms.getCache();
      //这里用到缓存 看之前是否用过sql查询 有的话酒从缓存拿了
      if (cache != null) {
        flushCacheIfRequired(ms);
        if (ms.isUseCache() && resultHandler == null) {
          ensureNoOutParams(ms, parameterObject, boundSql);
          @SuppressWarnings("unchecked")
          List<E> list = (List<E>) tcm.getObject(cache, key);
          if (list == null) {
            list = delegate.<E> query(ms, parameterObject, rowBounds, resultHandler, key, boundSql);
            tcm.putObject(cache, key, list); // issue #578 and #116
          }
          return list;
        }
      }
      //没有的话走BaseExecutor
      return delegate.<E> query(ms, parameterObject, rowBounds, resultHandler, key, boundSql);
    }
  
  ```

  **BaseExecutor：**

  ```
  //如果没有缓存走这里的query
  @SuppressWarnings("unchecked")
    @Override
    public <E> List<E> query(MappedStatement ms, Object parameter, RowBounds rowBounds, ResultHandler resultHandler, CacheKey key, BoundSql boundSql) throws SQLException {
      ErrorContext.instance().resource(ms.getResource()).activity("executing a query").object(ms.getId());
      if (closed) {
        throw new ExecutorException("Executor was closed.");
      }
      if (queryStack == 0 && ms.isFlushCacheRequired()) {
        clearLocalCache();
      }
      List<E> list;
      try {
        queryStack++;
        list = resultHandler == null ? (List<E>) localCache.getObject(key) : null;
        if (list != null) {
          handleLocallyCachedOutputParameters(ms, key, parameter, boundSql);
        } else {
          list = queryFromDatabase(ms, parameter, rowBounds, resultHandler, key, boundSql);
        }
      } finally {
        queryStack--;
      }
      if (queryStack == 0) {
        for (DeferredLoad deferredLoad : deferredLoads) {
          deferredLoad.load();
        }
        // issue #601
        deferredLoads.clear();
        if (configuration.getLocalCacheScope() == LocalCacheScope.STATEMENT) {
          // issue #482
          clearLocalCache();
        }
      }
      return list;
    }
  
    //再往里的queryFromDatabase
    private <E> List<E> queryFromDatabase(MappedStatement ms, Object parameter, RowBounds rowBounds, ResultHandler resultHandler, CacheKey key, BoundSql boundSql) throws SQLException {
      List<E> list;
      localCache.putObject(key, EXECUTION_PLACEHOLDER);
      try {
      //这里再次跳转 走SimpleExecutor
        list = doQuery(ms, parameter, rowBounds, resultHandler, boundSql);
      } finally {
        localCache.removeObject(key);
      }
      localCache.putObject(key, list);
      if (ms.getStatementType() == StatementType.CALLABLE) {
        localOutputParameterCache.putObject(key, parameter);
      }
      return list;
    }
   ...
   }
  ```

  **SimpleExecutor:类似的类还有ReuseExecutor，ClosedExecutor，BatchExecutor 都是BaseExecutor的子类**

  **值得一提的是再openSessionFromConnection或openSessionFromConnection的时候就会创建一个新的executor并制定类型:**

  ```
    public Executor newExecutor(Transaction transaction, ExecutorType executorType) {
      executorType = executorType == null ? defaultExecutorType : executorType;
      executorType = executorType == null ? ExecutorType.SIMPLE : executorType;
      Executor executor;
      if (ExecutorType.BATCH == executorType) {
        executor = new BatchExecutor(this, transaction);
      } else if (ExecutorType.REUSE == executorType) {
        executor = new ReuseExecutor(this, transaction);
      } else {
        executor = new SimpleExecutor(this, transaction);
      }
      if (cacheEnabled) {
        executor = new CachingExecutor(executor);
      }
      executor = (Executor) interceptorChain.pluginAll(executor);
      return executor;
    }
  ```

  ```
    @Override
    public <E> List<E> doQuery(MappedStatement ms, Object parameter, RowBounds rowBounds, ResultHandler resultHandler, BoundSql boundSql) throws SQLException {
      Statement stmt = null;
      try {
        Configuration configuration = ms.getConfiguration();
       //获得一个StatementHandler
        StatementHandler handler = configuration.newStatementHandler(wrapper, ms, parameter, rowBounds, resultHandler, boundSql);
       //进行预编译sql
        stmt = prepareStatement(handler, ms.getStatementLog());
        //交给handler去执行
        return handler.<E>query(stmt, resultHandler);
      } finally {
        closeStatement(stmt);
      }
    }
          //进行预编译sql
      private Statement prepareStatement(StatementHandler handler, Log statementLog) throws SQLException {
      Statement stmt;
      Connection connection = getConnection(statementLog);
      stmt = handler.prepare(connection, transaction.getTimeout());
      handler.parameterize(stmt);
      return stmt;
    }
  ```

  **交给StatementHandler去执行了,有几个实现类：这里走的PreparedStatementHandler**

  ```
   public class PreparedStatementHandler extends BaseStatementHandler {
  ...
   @Override
    public <E> List<E> query(Statement statement, ResultHandler resultHandler) throws SQLException {
      PreparedStatement ps = (PreparedStatement) statement;
      ps.execute();
      return resultSetHandler.<E> handleResultSets(ps);
    }
    ...
    }
  ```

  **最后交给DefaultResultSetHandler来做**

  ```
  public class DefaultResultSetHandler implements ResultSetHandler {
  
    @Override
    public List<Object> handleResultSets(Statement stmt) throws SQLException {
      ErrorContext.instance().activity("handling results").object(mappedStatement.getId());
  
      final List<Object> multipleResults = new ArrayList<Object>();
  
      int resultSetCount = 0;
      ResultSetWrapper rsw = getFirstResultSet(stmt);
  
      List<ResultMap> resultMaps = mappedStatement.getResultMaps();
      int resultMapCount = resultMaps.size();
      validateResultMapsCount(rsw, resultMapCount);
      while (rsw != null && resultMapCount > resultSetCount) {
        ResultMap resultMap = resultMaps.get(resultSetCount);
        handleResultSet(rsw, resultMap, multipleResults, null);
        rsw = getNextResultSet(stmt);
        cleanUpAfterHandlingResultSet();
        resultSetCount++;
      }
  
      String[] resultSets = mappedStatement.getResultSets();
      if (resultSets != null) {
        while (rsw != null && resultSetCount < resultSets.length) {
          ResultMapping parentMapping = nextResultMaps.get(resultSets[resultSetCount]);
          if (parentMapping != null) {
            String nestedResultMapId = parentMapping.getNestedResultMapId();
            ResultMap resultMap = configuration.getResultMap(nestedResultMapId);
            handleResultSet(rsw, resultMap, null, parentMapping);
          }
          rsw = getNextResultSet(stmt);
          cleanUpAfterHandlingResultSet();
          resultSetCount++;
        }
      }
  
      return collapseSingleResultList(multipleResults);
    }
      @SuppressWarnings("unchecked")
    private List<Object> collapseSingleResultList(List<Object> multipleResults) {
      return multipleResults.size() == 1 ? (List<Object>) multipleResults.get(0) : multipleResults;
    }
  
  
  ```

  **整体流程**

  **Mapper.selectOne()-->MapperProxy.invoke-->mapperMethod-->SqlSession.selectOne-->Executor.query()-->SimpleExecutor.doQuery-->PreparedStatementHandler.query-->DefaultResultSetHandler.query**

  