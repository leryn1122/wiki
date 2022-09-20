<a name="QFCuu"></a>
# HTTP 浏览器相关知识
HTTP 和浏览器有关的知识都在这里

<a name="e5GGs"></a>
## 浏览器安全策略

浏览器的安全机制包括 **网页安全模型** 和 **沙箱模型**.
<a name="KGQe8"></a>
### CORS 跨源资源共享

- [CORS - MDN](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/CORS)
<a name="xMbil"></a>
#### 原因

产生原因: 跨域产生的原因是由于前端地址与后台接口不是同源, 从而导致 Ajax 不能发送<br />非同源产生的问题

- Cookie, LocalStorage 和 IndexDB 无法获取
- DOM 无法获得
- Ajax 请求不能发送



**同源条件**

**协议**, **端口**, **主机**三者相同即为同源, 包括 IP 或者域名不同也算<br />反之, 其中只要某一个不一样则为不同源.<br />前后端分离的项目必然会出现跨域问题, 例如后端在 [http://localhost:8080/api/](http://localhost:8080/api/)** 而前端在 **[**http://localhost:3000/**](http://localhost:3000/) . 这样前端发请求给后端的时候就会报出跨域问题.

> 跨源资源共享 (CORS) 或通俗地译为跨域资源共享, 是一种基于 HTTP 头的机制, 该机制通过允许服务器标示除了它自己以外的其它 origin 这样浏览器可以访问加载这些资源. 跨源资源共享还通过一种机制来检查服务器是否会允许要发送的真实请求, 该机制通过浏览器发起一个到服务器托管的跨源资源的"预检 (Preflight) "请求. 在预检中, 浏览器发送的头中标示有 HTTP 方法和真实请求中会用到的头.


所以只要在请求中增加 CORS 的请求头就可以开启跨域支持. 此外还需要放行所有的 OPTIONS 预检请求.<br />当然不是所有请求都会发出预检请求. 这类请求称为简单请求, 满足以下条件的请求就是简单请求:<br />使用下列方法之一:

- GET
- HEAD
- POST

请求头只能包含以下几个:

- Accept
- Accept-Language
- Content-Language

Content-Type 的值仅限于下列三者之一:

- text/plain
- multipart/form-data
- application/x-www-form-urlencoded
<a name="h1UeY"></a>
#### 解决方式


**本地开发跨域**<br />本地开发一般使用下面 3 种方式进行处理:

- vite 的 proxy 进行代理
- 后台开启 CORS, 前端不需要做任何改动
- 使用 nginx 转发请求



**生产环境跨域**<br />生产环境一般使用下面 2 种方式进行处理:

- 后台开启 CORS, 前端不需要做任何改动
- 使用 nginx 转发请求

注意只需要一个地方增加跨域支持就可以, 重复增加会导致浏览器报重复的 CORS 请求头的错误.

<a name="cbggy"></a>
#### 解决方式: Java 后端

后端的解决方法其实很多, 这里只需要在后端增加几个请求头就 OK:

```java
@Override
public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain)
  throws IOException, ServletException {
  HttpServletRequest httpServletRequest = (HttpServletRequest) servletRequest;
  HttpServletResponse httpServletResponse = (HttpServletResponse) servletResponse;
  httpServletResponse.addHeader(HttpHeaders.ACCESS_CONTROL_ALLOW_ORIGIN, httpServletRequest.getHeader(HttpHeaders.ORIGIN));
  httpServletResponse.addHeader(HttpHeaders.ACCESS_CONTROL_ALLOW_HEADERS, getHeaders(httpServletRequest));
  httpServletResponse.setHeader(HttpHeaders.ACCESS_CONTROL_ALLOW_CREDENTIALS, "true");
  httpServletResponse.setHeader(HttpHeaders.ACCESS_CONTROL_ALLOW_METHODS, "GET,POST,PUT,DELETE,OPTIONS");
  chain.doFilter(requestWrapper, response);
}
```

当然 Springboot 也提供了对应的解决方法:

```java
@Configuration
public class GlobalCorsConfiguration {
  @Bean
  public CorsFilter corsFilter() {
    CorsConfiguration corsConfiguration = new CorsConfiguration();
    corsConfiguration.addAllowedOriginPattern("*");
    corsConfiguration.setAllowCredentials(true);
    corsConfiguration.addAllowedHeader("*");
    corsConfiguration.setAllowedMethods(Arrays.asList("GET","POST","PUT","DELETE","OPTIONS"));
    corsConfiguration.setExposedHeaders(Arrays.asList(
      HttpHeaders.ACCESS_CONTROL_REQUEST_METHOD,
      HttpHeaders.ACCESS_CONTROL_REQUEST_HEADERS,
      HttpHeaders.AUTHORIZATION,
      "X-Requested-With",
      "X-Access-Token"));
    UrlBasedCorsConfigurationSource source = new UrlBasedCorsConfigurationSource();
    source.registerCorsConfiguration("/**", corsConfiguration);
    return new CorsFilter(source);
  }
}
```

<a name="ox1jC"></a>
#### 解决方式: nginx 转发请求

既然同源策略是浏览器安全策略, 那么直接让 nginx 代理来发起请求就不会产生跨域问题了.

```nginx
server {
  listen       8080;
  server_name  localhost;
  
  # 接口代理, 用于解决跨域问题
  location ^~ /api {
        
    # 跨域请求头
    add_header Access-Control-Allow-Credentials  true;
    add_header Access-Control-Allow-Origin       $host;
    add_header Access-Control-Allow-Headers      X-Requested-With;
    add_header Access-Control-Allow-Methods      GET,POST,PUT,DELETE,OPTIONS;
    
    proxy_set_header Host             $host;
    proxy_set_header X-Real-IP        $remote_addr;
    proxy_set_header X-Forwarded-For  $proxy_add_x_forwarded_for;
    
    # 后台接口地址
    proxy_pass http://backend/;
    proxy_redirect default;
  }
}
```

<a name="ERwZB"></a>
## XSS

// TODO
<a name="lzZbj"></a>
## CSP 内容安全策略

参考文档:

- [CSP - MDN](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/CSP)

注意:

- CSP 定义的是网页自身能够访问的某些域和资源
- CORS 定义的是一个网页如何才能访问被同源策略禁止的跨域资源, 规定两者交互的协议和方式

CSP 指定了可信任有效域, 让浏览器信任白名单中可执行脚本的来源, 并忽略所有其他脚本. 

CSP 是基于 HTTP 头的安全策略:

```http
Content-Security-Policy: default-src 'self' *.trusted.com
```

也可以 (但不推荐) 在 HTML 的 `<meta>` 元素中配置该策略:

```html
<meta http-equiv="Content-Security-Policy" content="default-src 'self'; img-src https://*; child-src 'none';">
```

实际开发中, 我们更倾向于在前端 pod 对应的 ingress (k8s 环境下) 中或对应的 nginx 中 (非 docker 环境) 添加, 这不会对前端原有代码产生侵入式改动.<br />添加不同资源类型:

```http
Content-Security-Policy: default-src 'self' 'unsafe-inline' 'unsafe-eval' *.leryn.top *.qq.com *.baidu.com; script-src 'self' 'unsafe-inline' 'unsafe-eval' *.leryn.top *.qq.com *.baidu.com; img-src 'self' *.leryn.top data: *.qq.com; frame-src 'self' *.leryn.top webcompt: data: *.qq.com; connect-src 'self' *.leryn.top *.leryn.top:30333; font-src 'self' *.alicdn.com data:;
```

如果有安全策略, 那么他会报错:

```
Refused to load the font 'http://cdn.jsdelivr.net/npm/vue@2.6.11/dist/vue.min.js' because it violates the following Content Security Policy directive: "script-src 'self' 'unsafe-inline' 'unsafe-eval' *.leryn.top *.qq.com *.baidu.com".
```
