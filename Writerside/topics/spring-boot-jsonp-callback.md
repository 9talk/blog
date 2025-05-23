#  jsonp CallBack配置

引用文章: <a href="https://blog.csdn.net/feinifi/article/details/116228582">springboot实现jsonp</a>

JSONP的最基本的原理是：动态添加一个`<script>`标签，而`script`标签的src属性是没有跨域的限制的。

JSONP仅支持GET请求，不支持POST等其他类型的HTTP请求。

jsonp是ajax跨域解决方案的一种办法，就是借助标签`<script></script>`可以实现不同域之间数据请求的一种方式，类似iframe，不受跨域限制，它请求返回之后，会以一种回调的形式调起挂在window对象上的全局方法callback，这里的callback就是我们在url请求中指定的回调函数，参数就是我们请求服务端包装在callback之内的数据。

jsonp的实现，需要前后端配合，不是说前端可以实现，后端也可以实现，他们需要配合才能实现一个完整的jsonp请求。

springboot1.x对jsonp还有一个AbstractJsonpResponseBodyAdvice的抽象类可以去实现jsonp，而无需改动普通的GET请求，但是在springboot2.x版本里面已经没有这个类了，所以又回到了springmvc阶段对jsonp的实现。

## Spring Boot 1.x

```java
// 内部静态类
@ControllerAdvice
static class JsonpAdvice extends AbstractJsonpResponseBodyAdvice {
    public JsonpAdvice() {
        super("callback");
    }
}
```

## Spring Boot 2.x

```java
@ControllerAdvice
@Slf4j
public class JsonpCallBackAdvice implements ResponseBodyAdvice {
    
    private ObjectMapper OM = new ObjectMapper();
    
    @Override
    public boolean supports(MethodParameter returnType, Class converterType) {
        return true;
    }
    
    @Override
    public Object beforeBodyWrite(Object body, MethodParameter returnType, MediaType selectedContentType, Class selectedConverterType, ServerHttpRequest request, ServerHttpResponse response) {
        HttpServletRequest servletRequest = ((ServletServerHttpRequest) request).getServletRequest();
        HttpServletResponse servletResponse = ((ServletServerHttpResponse) response).getServletResponse();
        String value = servletRequest.getParameter("callback");
        if (value != null) {
            try {
                byte[] bytes = (value + "(" + OM.writeValueAsString(body) + ")").getBytes();
                ServletOutputStream outputStream = servletResponse.getOutputStream();
                outputStream.write(bytes); 
                outputStream.flush();
                outputStream.close();
            } catch (JsonProcessingException ignore) {
            } catch (IOException ignore) {
            }
            if (log.isDebugEnabled()) {
                log.debug("Ignoring invalid jsonp parameter value: " + value);
            }
        }
        return body;
    }
}
```
