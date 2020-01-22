---
title: CORS
date: 2019-09-07 00:17:40
tags: [JAVA]
categories: [WEB]
---

JavaWeb中的跨域<!--more-->



web开发中经常遇到跨域问题 一般分两种

比如现在有一个前后端分离的项目，前端域名为front.github.com

后端接口服务工程域名 为  backgroud.baidu.com

当前端登录时候需要调用后端服务工程的一个接口 backgroud.baidu.com/sso/login

* 后端代码添加跨域过滤器

  * 代码跨域过滤器

    ````
    public class TokenFilter extends OncePerRequestFilter {
        @Override
        protected void doFilterInternal(HttpServlet Requestrequest, HttpServletResponse response, FilterChain filterChain) throws ServletException, IOException {
            // do something
            cors(request, response);
            // do something
        }
      
        private void cors(HttpServletRequest request, HttpServletResponse response) {
            String allowOrigin = request.getHeader("Origin");
            String allowMethods = "GET,PUT, POST, DELETE";
            String allowHeaders = "Origin,No-Cache, X-Requested-With, If-Modified-Since, Pragma, Last-Modified,Cache-Control, Expires, Content-Type, X-E4M-With";
            response.addHeader("Access-Control-Allow-Credentials", "true");
            response.addHeader("Access-Control-Allow-Headers", allowHeaders);
            response.addHeader("Access-Control-Allow-Methods", allowMethods);
            response.addHeader("Access-Control-Allow-Origin", allowOrigin);
        }
    }
    ````

    ````
    // /* 表示全部拦截
    @WebFilter(filterName = "loginFilter",urlPatterns = "/*")
    public class LoginFilter implements Filter {
        //这里面 填写不需要 被拦截的地址
        private static final Set<String> ALLOWED_PATHS = Collections.unmodifiableSet(
                new HashSet<String>( Arrays.asList("/login","/isLogin","/findCategory") )
        );
     
        //初始化调用的方法
        //当服务器 被启动的时候，调用
        public void init(FilterConfig filterConfig) throws ServletException { }
     
        //拦截的方法
        public void doFilter(ServletRequest req, ServletResponse res, FilterChain filterChain) throws IOException, ServletException {
     
     
            HttpServletRequest request = (HttpServletRequest) req;
            HttpServletResponse response = (HttpServletResponse) res;
     
            //解决跨域的问题 http://localhost:3030 为非同域名下的请求方
            response.setHeader("Access-Control-Allow-Origin","http://localhost:3030");
            response.setHeader("Access-Control-Allow-Credentials","true");
            response.setHeader("Access-Control-Allow-Headers", "Content-Type,Content-Length, Authorization, Accept,X-Requested-With,X-App-Id, X-Token");
            response.setHeader("Access-Control-Allow-Methods","PUT,POST,GET,DELETE,OPTIONS");
            response.setHeader("Access-Control-Max-Age", "3600");
     
     
            String path = request.getRequestURI().substring(request.getContextPath().length()).replaceAll("[/]+$","");
            boolean allowePath =  ALLOWED_PATHS.contains(path);
     
            if (!allowePath) {  //需要拦截的方法
                Object aa = request.getSession().getAttribute("user");
                if (aa != null) {
                    filterChain.doFilter(request,response);
            }
                else {
                    response.getWriter().write("noLogin");
                }
            } else {  //不需要被拦截的方法
                //直接放行
                filterChain.doFilter(request,response);
            }
     
     
     
        }
     
        //销毁时候调用的方法
        public void destroy() { }
    }
    
    ````

  * 注解跨域过滤器

    spring版本4.2以上 注解源码如下

    ````
    @Target({ ElementType.METHOD, ElementType.TYPE })
    @Retention(RetentionPolicy.RUNTIME)
    @Documented
    public @interface CrossOrigin {
    
        String[] DEFAULT_ORIGINS = { "*" };
    
        String[] DEFAULT_ALLOWED_HEADERS = { "*" };
    
        boolean DEFAULT_ALLOW_CREDENTIALS = true;
    
        long DEFAULT_MAX_AGE = 1800;
    
    
        /**
         * 同origins属性一样
         */
        @AliasFor("origins")
        String[] value() default {};
    
        /**
         * 所有支持域的集合，例如"http://domain1.com"。
         * <p>这些值都显示在请求头中的Access-Control-Allow-Origin
         * "*"代表所有域的请求都支持
         * <p>如果没有定义，所有请求的域都支持
         * @see #value
         */
        @AliasFor("value")
        String[] origins() default {};
    
        /**
         * 允许请求头重的header，默认都支持
         */
        String[] allowedHeaders() default {};
    
        /**
         * 响应头中允许访问的header，默认为空
         */
        String[] exposedHeaders() default {};
    
        /**
         * 请求支持的方法，例如"{RequestMethod.GET, RequestMethod.POST}"}。
         * 默认支持RequestMapping中设置的方法
         */
        RequestMethod[] methods() default {};
    
        /**
         * 是否允许cookie随请求发送，使用时必须指定具体的域
         */
        String allowCredentials() default "";
    
        /**
         * 预请求的结果的有效期，默认30分钟
         */
        long maxAge() default -1;
    
    }
    ````

    

    实现

    在controller上添加该注解 表示 该注解的方法都接收origin跨域请求方的请求

    ```
    @CrossOrigin(origins = "http://domain2.com", maxAge = 3600)
    @RestController
    @RequestMapping("/account")
    public class AccountController {
    
        @RequestMapping("/{id}")
        public Account retrieve(@PathVariable Long id) {
            // ...
        }
    
        @RequestMapping(method = RequestMethod.DELETE, path = "/{id}")
        public void remove(@PathVariable Long id) {
            // ...
        }
    }
    ```

    在方法上使用 表示单个方法允许跨域请求

    ```
    @CrossOrigin(maxAge = 3600)
    @RestController
    @RequestMapping("/account")
    public class AccountController {
    
        @CrossOrigin("http://domain2.com")
        @RequestMapping("/{id}")
        public Account retrieve(@PathVariable Long id) {
            // ...
        }
    
        @RequestMapping(method = RequestMethod.DELETE, path = "/{id}")
        public void remove(@PathVariable Long id) {
            // ...
        }
    }
    ```

    全局的跨域请求配置

    ````
    @Configuration
    @EnableWebMvc
    public class WebConfig extends WebMvcConfigurerAdapter {
    
        @Override
        public void addCorsMappings(CorsRegistry registry) {
            registry.addMapping("/**");
        }
    }
    ````

    特定路径的CORS

    ````
    @Configuration
    @EnableWebMvc
    public class WebConfig extends WebMvcConfigurerAdapter {
    
        @Override
        public void addCorsMappings(CorsRegistry registry) {
            registry.addMapping("/api/**")
                .allowedOrigins("http://domain2.com")
                .allowedMethods("PUT", "DELETE")
                .allowedHeaders("header1", "header2", "header3")
                .exposedHeaders("header1", "header2")
                .allowCredentials(false).maxAge(3600);
        }
    }
    ````

    

  * springBoot配置跨域

    配置起来比较罗嗦

    ```
    @Configuration
    public class CorsConfig {
        @Bean
        public CorsFilter corsFilter() {
            //1.添加CORS配置信息
            CorsConfiguration config = new CorsConfiguration();
            //放行哪些原始域
            config.addAllowedOrigin("*");
            //是否发送Cookie信息
            config.setAllowCredentials(true);
            //放行哪些原始域(请求方式)
            config.addAllowedMethod("*");
            //放行哪些原始域(头部信息)
            config.addAllowedHeader("*");
    
            //2.添加映射路径
            UrlBasedCorsConfigurationSource configSource = new UrlBasedCorsConfigurationSource();
            configSource.registerCorsConfiguration("/**", config);
    
            //3.返回新的CorsFilter.
            return new CorsFilter(configSource);
        }
    }
    ```

    

* nginx配置反向代理解决

  主要利用proxy_pass来解决

  修改前端nginx的配置文件

  ```
  server {
          listen       80;
          server_name  front.github.com;
          access_log  logs/test.access.log;
          # 匹配以sso开头的请求
          location /sso {
              proxy_pass http://background.baidu.com/;  #注意域名后有一个/
          }
          location / {
              root html/a;
              index index.html index.htm;
          }
          #
          error_page   500 502 503 504  /50x.html;
          location = /50x.html {
              root   html;
          }
      }
  ```

  加上/的话 访问 backgroud.baidu.com/sso/login.html 获得预期的 backgroud.baidu.com/login.html

  不加/的话 访问 backgroud.baidu.com/sso/login.html 得到的是 backgroud.baidu.com/sso/login.html 

  那么前端访问请求接口地址应该改为front.github.com/sso/login

  **需要注意的是当请求方需要携带cookies时候（Access-Control-Allow-Credentials", "true"）进行访问跨域方时候 Access-Control-Allow-Origin 不能为***

