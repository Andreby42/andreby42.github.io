---
title: SpringBoot开启https
date: 2018-09-16 11:57:30
tags: [Java]
categories: [SpringBoot]
---

SpringBoot2.x开启Https,了解下<!--more-->

#### SSL证书生成

 * 使用java的自带的keytool生成ssl证书

   ```
   
   keytool -genkeypair -alias skyeyes  -keyalg RSA -keysize 4096 -keypass nopassword -sigalg SHA384withRSA -dname "cn=Wong,ou=skyeye,o=skyeye,l=Beijing,st=Beijing,c=CN" -validity 3650  -keystore skyeye.jks -storetype JKS -storepass nopassword
   ```

   * -storetype 指定密钥仓库类型  

   *  -keyalg 生证书的算法名称 

   * -keysize 证书大小 

   * -keystore 生成的证书文件的存储路径 

   * -validity 证书的有效期

   * -keypass nopassword 私钥的密码 最好与 storepass 一致。

   * -sigalg SHA384withRSA此处”SHA384withRSA“为签名算法  用sha3

   * -dname “cn=www.mydomain.com,ou=xxx,o=xxx,l=Beijing,st=Beijing,c=CN” 在此填写证书信息。”CN = 名字与姓氏 / 域名, OU = 组织单位名称, O = 组织名称, L = 城市或区域名称, ST = 州或省份名称, C = 单位的两字母国家代码” 

   * -storetype JKS 此处”JKS “为证书库类型。可用的证书库类型为：JKS、PKCS12 等。jdk9 以前，默认为 JKS。自 jdk9 开始，默认为 PKCS12

   * -storepass nopassword此处”nopassword“为证书库密码 (私钥的密码)。最好与 keypass 一致 

   * PS:   **上述命令，需要将 -dname 参数替换（尤其时域名要写对）、密码更改即可，其它可保持不变。** 

     使用everything搜索生成的www.spacexplore.com_keystore.jks 找到这个文件，

* 项目中配置证书

   * 将上述文件拷贝到项目的resources 底下,windows放在项目根目录下。

   * 配置文件中加入ssl证书配置

     ```
     server.ssl.key-store= classpath:skyeye.jks
     server.ssl.key-store-password=nopassword
     server.ssl.keyStoreType=JKS
     server.ssl.keyAlias:skyeyes
     ```

* 配置代码

   **注意这里的HTTPS端口就是server.port的端口，两个要一致** 

   ```
   @Configuration
   public class TomcatConfig {
   	private Connector createHTTPConnector() {
   		Connector connector = new Connector("org.apache.coyote.http11.Http11NioProtocol");
   		// 同时启用 http（8080）、https（8082）两个端口
   		connector.setScheme("http");
   		connector.setSecure(false);
   		//设置http端口8080 
   		connector.setPort(8080);
   		//设置https端口为8082
   		connector.setRedirectPort(8082);
   		return connector;
   	}
   
   	/**
   	 * 這是2.x的配置方法 1.x與2，x不一樣
   	 * @Bean
   	 * @return
   	 */
   	public ServletWebServerFactory servletContainer() {
   		TomcatServletWebServerFactory tomcat = new TomcatServletWebServerFactory();
   		tomcat.addAdditionalTomcatConnectors(createHTTPConnector());
   		return tomcat;
   	}
   }
   ```

   

* 启动报错了。。。

   ```
   Caused by: java.io.IOException: DerInputStream.getLength(): lengthTag=109, too big.
   	at sun.security.util.DerInputStream.getLength(DerInputStream.java:599) ~[?:1.8.0_171]
   	at sun.security.util.DerValue.init(DerValue.java:391) ~[?:1.8.0_171]
   	at sun.security.util.DerValue.<init>(DerValue.java:332) ~[?:1.8.0_171]
   	at sun.security.util.DerValue.<init>(DerValue.java:345) ~[?:1.8.0_171]
   	at sun.security.pkcs12.PKCS12KeyStore.engineLoad(PKCS12KeyStore.java:1938) ~[?:1.8.0_171]
   	at java.security.KeyStore.load(KeyStore.java:1445) ~[?:1.8.0_171]
   	at org.apache.tomcat.util.net.SSLUtilBase.getStore(SSLUtilBase.java:139) ~[tomcat-embed-core-8.5.31.jar:8.5.31]
   	at org.apache.tomcat.util.net.SSLHostConfigCertificate.getCertificateKeystore(SSLHostConfigCertificate.java:204) ~[tomcat-embed-core-8.5.31.jar:8.5.31]
   	at org.apache.tomcat.util.net.jsse.JSSEUtil.getKeyManagers(JSSEUtil.java:184) ~[tomcat-embed-core-8.5.31.jar:8.5.31]
   	at org.apache.tomcat.util.net.AbstractJsseEndpoint.createSSLContext(AbstractJsseEndpoint.java:114) ~[tomcat-embed-core-8.5.31.jar:8.5.31]
   	at org.apache.tomcat.util.net.AbstractJsseEndpoint.initialiseSsl(AbstractJsseEndpoint.java:87) ~[tomcat-embed-core-8.5.31.jar:8.5.31]
   	at org.apache.tomcat.util.net.NioEndpoint.bind(NioEndpoint.java:225) ~[tomcat-embed-core-8.5.31.jar:8.5.31]
   	at org.apache.tomcat.util.net.AbstractEndpoint.start(AbstractEndpoint.java:1150) ~[tomcat-embed-core-8.5.31.jar:8.5.31]
   	at org.apache.coyote.AbstractProtocol.start(AbstractProtocol.java:591) ~[tomcat-embed-core-8.5.31.jar:8.5.31]
   	at org.apache.catalina.connector.Connector.startInternal(Connector.java:1018) ~[tomcat-embed-core-8.5.31.jar:8.5.31]
   	at org.apache.catalina.util.LifecycleBase.start(LifecycleBase.java:150) ~[tomcat-embed-core-8.5.31.jar:8.5.31]
   ```

   看起来像是读取文件导致的问题。

   **解决办法:**

   ```
    <plugin>
                   <groupId>org.apache.maven.plugins</groupId>
                   <artifactId>maven-resources-plugin</artifactId>
                   <configuration>
                       <encoding>UTF-8</encoding>
                       <!-- 过滤后缀为 pem、pfx 的证书文件 -->
                       <nonFilteredFileExtensions>
                           <nonFilteredFileExtension>pem</nonFilteredFileExtension>
                           <nonFilteredFileExtension>pfx</nonFilteredFileExtension>
                           <nonFilteredFileExtension>p12</nonFilteredFileExtension>
                       </nonFilteredFileExtensions>
                   </configuration>
               </plugin>
   ```

   同时生成的密钥文件类型要与配置中的相同,(**如果需要进行密钥格式转换，那么就转换 这里就不写了**)，如密钥文件为.jks,那么配置这么写：(**如果server.ssl.key-store= skyeye.jks这么写就得将密钥文件放置在项目的根目录下，而这么写server.ssl.key-store=classpath: skyeye.jks,放到resources就可以了**)

   ```
   server.ssl.key-store=classpath:skyeye.jks
   server.ssl.keyStoreType=JKS
   ```

* 进行测试访问

   ![SpringBoot开启https](SpringBoot开启https/HTTPS1.png)

* http与https共存时候，强制转换为https，做法就是开启过滤器强制转换为https

   ```
   /**
    * 将http访问请求转为https的过滤器 http与https共存时
    * 
    * @author andreby
    *
    */
   @Configuration
   @WebFilter(urlPatterns = "/*", filterName = "http2HttpsFilter")
   public class Http2HttpsFilter extends OncePerRequestFilter {
   
   	@Override
   	protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response, FilterChain filterChain)
   			throws ServletException, IOException {
   		String requestURL = request.getRequestURL().toString();
   		String protocol = requestURL.split("://")[0];
   		if ("http".equals(protocol)) {
   			// 将http请求uri替换为https,将http端口请求替换为https端口 8081端口被nexus占用 换8082
   			requestURL = requestURL.replace("http", "https").replace("8080", "8082");
   			response.sendRedirect(requestURL);
   		}
   		filterChain.doFilter(request, response);
   	}
   
   }
   
   ```

   

* 不需要共存且需要将http转为https,去掉Http2HttpsFilter，并修改TomcatConfig类为如下。

   ```
   @Configuration
   public class TomcatConfig {
       @Bean
   	public Connector httpConnector() {
   		Connector connector = new Connector("org.apache.coyote.http11.Http11NioProtocol");
   		connector.setScheme("http");
   		connector.setSecure(false);
   		// 设置http8080端口 和 重定向到https的端口8082 8081端口被nexus占用 换8082
   		// 这个8080端口为http端口 如果这里设置了http端口那么server.port是不是就不用设置了？
   		connector.setPort(8080);
   		// 这个8082端口为https端口 8081端口被nexus占用 换8082
   		connector.setRedirectPort(8082);
   		return connector;
   	}
   
   	/**
   	 * 這是2.x的配置方法 1.x與2，x不一樣
   	 * 
   	 * @Bean
   	 * @return
   	 */
   	@Bean
   	public ServletWebServerFactory servletContainer() {
   		// http与https共存时候，需要强转http为https时候用注释代码
   		/*
   		 * TomcatServletWebServerFactory tomcat = new TomcatServletWebServerFactory();
   		 * tomcat.addAdditionalTomcatConnectors(createHTTPConnector()); return tomcat;
   		 */
   		TomcatServletWebServerFactory tomcat = new TomcatServletWebServerFactory() {
   			@Override
   			protected void postProcessContext(Context context) {
   				SecurityConstraint constraint = new SecurityConstraint();
   				constraint.setUserConstraint("CONFIDENTIAL");
   				SecurityCollection collection = new SecurityCollection();
   				collection.addPattern("/*");
   				constraint.addCollection(collection);
   				context.addConstraint(constraint);
   			}
   		};
   		tomcat.addAdditionalTomcatConnectors(httpConnector());
   		return tomcat;
   
   	}
   }
   ```

   

* 一些其他的配置

   * 如果你有一个代理服务器，你需要设置`spring.devtools.remote.proxy.host`和`spring.devtools.remote.proxy.port`这两个属性。 （属于devtools这个东西）

   * springboot2.0.5官档关于运维管理端ssl的说明

     When configured to use a custom port, the management server can also be configured with its own SSL by using the various management.server.ssl.* properties. For example, doing so lets a management server be available over HTTP while the main application uses HTTPS, as shown in the following property settings:

     ```
     server.port=8443
     server.ssl.enabled=true
     server.ssl.key-store=classpath:store.jks
     server.ssl.key-password=secret
     management.server.port=8083
     management.server.ssl.enabled=false
     ```

     Alternatively, both the main server and the management server can use SSL but with different key stores, as follows: 

     ```
     server.port=8443
     server.ssl.enabled=true
     server.ssl.key-store=classpath:main.jks
     server.ssl.key-password=secret
     management.server.port=8083
     management.server.ssl.enabled=true
     management.server.ssl.key-store=classpath:management.jks
     management.server.ssl.key-password=secret
     ```

     也就是可以配置多个证书。 这个 management.server.port 就是管理端的端口，其实是运维管理端口。配合actuator 这个类库来用。这样的话我们就可以将前台应用端口和后端运维管理隔离开，比如trace

     直接可以 http://127.0.0.1:8083/trace   来获得trace信息。而管理端的https也是可以配置的。

   * **SpringBoot2.0.5关于ssl的说明**

     SSL can be configured declaratively by setting the various `server.ssl.*` properties, typically in `application.properties` or `application.yml`. The following example shows setting SSL properties in `application.properties`: 

     ```
     server.port=8443
     server.ssl.key-store=classpath:keystore.jks
     server.ssl.key-store-password=secret
     server.ssl.key-password=another-secret
     ```

     下面这个类是ssl的配置类注意看注释

     ```
     /*
      * Copyright 2012-2017 the original author or authors.
      *
      * Licensed under the Apache License, Version 2.0 (the "License");
      * you may not use this file except in compliance with the License.
      * You may obtain a copy of the License at
      *
      *      http://www.apache.org/licenses/LICENSE-2.0
      *
      * Unless required by applicable law or agreed to in writing, software
      * distributed under the License is distributed on an "AS IS" BASIS,
      * WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
      * See the License for the specific language governing permissions and
      * limitations under the License.
      */
     
     package org.springframework.boot.web.server;
     
     /**
      * Simple server-independent abstraction for SSL configuration.
      *
      * @author Andy Wilkinson
      * @author Vladimir Tsanev
      * @since 2.0.0
      */
     public class Ssl {
     
     	/**
     	 * Whether to enable SSL support.
     	 */
     	private boolean enabled = true;
     
     	/**
     	 * Whether client authentication is wanted ("want") or needed ("need"). Requires a
     	 * trust store.
     	 */
     	private ClientAuth clientAuth;
     
     	/**
     	 * Supported SSL ciphers.
     	 */
     	private String[] ciphers;
     
     	/**
     	 * Enabled SSL protocols.
     	 */
     	private String[] enabledProtocols;
     
     	/**
     	 * Alias that identifies the key in the key store.
     	 */
     	private String keyAlias;
     
     	/**
     	 * Password used to access the key in the key store.
     	 */
     	private String keyPassword;
     
     	/**
     	 * Path to the key store that holds the SSL certificate (typically a jks file).
     	 */
     	private String keyStore;
     
     	/**
     	 * Password used to access the key store.
     	 */
     	private String keyStorePassword;
     
     	/**
     	 * Type of the key store.
     	 */
     	private String keyStoreType;
     
     	/**
     	 * Provider for the key store.
     	 */
     	private String keyStoreProvider;
     
     	/**
     	 * Trust store that holds SSL certificates.
     	 */
     	private String trustStore;
     
     	/**
     	 * Password used to access the trust store.
     	 */
     	private String trustStorePassword;
     
     	/**
     	 * Type of the trust store.
     	 */
     	private String trustStoreType;
     
     	/**
     	 * Provider for the trust store.
     	 */
     	private String trustStoreProvider;
     
     	/**
     	 * SSL protocol to use.
     	 */
     	private String protocol = "TLS";
     
     	public boolean isEnabled() {
     		return this.enabled;
     	}
     
     	public void setEnabled(boolean enabled) {
     		this.enabled = enabled;
     	}
     
     	public ClientAuth getClientAuth() {
     		return this.clientAuth;
     	}
     
     	public void setClientAuth(ClientAuth clientAuth) {
     		this.clientAuth = clientAuth;
     	}
     
     	public String[] getCiphers() {
     		return this.ciphers;
     	}
     
     	public void setCiphers(String[] ciphers) {
     		this.ciphers = ciphers;
     	}
     
     	public String getKeyAlias() {
     		return this.keyAlias;
     	}
     
     	public void setKeyAlias(String keyAlias) {
     		this.keyAlias = keyAlias;
     	}
     
     	public String getKeyPassword() {
     		return this.keyPassword;
     	}
     
     	public void setKeyPassword(String keyPassword) {
     		this.keyPassword = keyPassword;
     	}
     
     	public String getKeyStore() {
     		return this.keyStore;
     	}
     
     	public void setKeyStore(String keyStore) {
     		this.keyStore = keyStore;
     	}
     
     	public String getKeyStorePassword() {
     		return this.keyStorePassword;
     	}
     
     	public void setKeyStorePassword(String keyStorePassword) {
     		this.keyStorePassword = keyStorePassword;
     	}
     
     	public String getKeyStoreType() {
     		return this.keyStoreType;
     	}
     
     	public void setKeyStoreType(String keyStoreType) {
     		this.keyStoreType = keyStoreType;
     	}
     
     	public String getKeyStoreProvider() {
     		return this.keyStoreProvider;
     	}
     
     	public void setKeyStoreProvider(String keyStoreProvider) {
     		this.keyStoreProvider = keyStoreProvider;
     	}
     
     	public String[] getEnabledProtocols() {
     		return this.enabledProtocols;
     	}
     
     	public void setEnabledProtocols(String[] enabledProtocols) {
     		this.enabledProtocols = enabledProtocols;
     	}
     
     	public String getTrustStore() {
     		return this.trustStore;
     	}
     
     	public void setTrustStore(String trustStore) {
     		this.trustStore = trustStore;
     	}
     
     	public String getTrustStorePassword() {
     		return this.trustStorePassword;
     	}
     
     	public void setTrustStorePassword(String trustStorePassword) {
     		this.trustStorePassword = trustStorePassword;
     	}
     
     	public String getTrustStoreType() {
     		return this.trustStoreType;
     	}
     
     	public void setTrustStoreType(String trustStoreType) {
     		this.trustStoreType = trustStoreType;
     	}
     
     	public String getTrustStoreProvider() {
     		return this.trustStoreProvider;
     	}
     
     	public void setTrustStoreProvider(String trustStoreProvider) {
     		this.trustStoreProvider = trustStoreProvider;
     	}
     
     	public String getProtocol() {
     		return this.protocol;
     	}
     
     	public void setProtocol(String protocol) {
     		this.protocol = protocol;
     	}
     
     	/**
     	 * Client authentication types.
     	 */
     	public enum ClientAuth {
     
     		/**
     		 * Client authentication is wanted but not mandatory.
     		 */
     		WANT,
     
     		/**
     		 * Client authentication is needed and mandatory.
     		 */
     		NEED
     
     	}
     
     }
     ```

     **Using configuration such as the preceding example means the application no longer supports a plain HTTP connector at port 8080. Spring Boot does not support the configuration of both an HTTP connector and an HTTPS connector through `application.properties`. If you want to have both, you need to configure one of them programmatically. We recommend using `application.properties` to configure HTTPS, as the HTTP connector is the easier of the two to configure programmatically. See the [`spring-boot-sample-tomcat-multi-connectors`](https://github.com/spring-projects/spring-boot/tree/v2.0.5.RELEASE/spring-boot-samples/spring-boot-sample-tomcat-multi-connectors) sample project for an example. **

     这段话的意思是，如果用了上面的配置，那么就不支持内置tomcat的默认8080端口了，如果你两个都想保留，那么可以用代码的方式实现，同时我们保留了使用application.properties去配置https，下面请看tomcat配置多连接器的代码示例，嗯 ，代码示例我看了 ，就是最上面，我用的那种代码配置多连接器。

   * **application.properties中的ssl配置详解**

     ```
     server.ssl.ciphers   #是否支持SSL ciphers.
     server.ssl.client-auth   #设定client authentication是wanted 还是 needed.
     server.ssl.enabled   #是否开启ssl，默认: true
     server.ssl.key-alias   #设定key store中key的别名.
     server.ssl.key-password   #访问key store中key的密码.
     server.ssl.key-store   #设定持有SSL certificate的key store的路径，通常是一个.jks文件.
     server.ssl.key-store-password   #设定访问key store的密码.
     server.ssl.key-store-provider   #设定key store的提供者.
     server.ssl.key-store-type   #设定key store的类型.
     server.ssl.protocol   #使用的SSL协议，默认: TLS
     server.ssl.trust-store   #持有SSL certificates的Trust store.
     server.ssl.trust-store-password   #访问trust store的密码.
     server.ssl.trust-store-provider   #设定trust store的提供者.
     server.ssl.trust-store-type   #指定trust store的类型.
     ```

   * 开启http2

     在application.properties中添加

     ```
     ##开启HTTP2
     server.http2.enabled=true  
     ```

* **正常开发中，会在nginx端做ssl证书配置，这是后话了**