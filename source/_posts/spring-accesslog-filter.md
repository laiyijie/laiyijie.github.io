---
title: 在SpringMVC中打印完整的AccessLog
date: 2017-07-13 16:50:51
categories: 
- Spring
tags:
- learning
---
转载请注明来源 [赖赖的博客](http://laiyijie.me)

## 导语
> 总要造一下轮子才知道别人的轮子有多厉害

最近笔者遇到一个问题：在SpringMVC框架下，没有直接的方式可以打印完整的AccessLog，因为request body 和response body是通过inputstream和outputstream来封装的，在使用过很多人提供的方式后，发现效果总是不尽人意，所以自己摸索了一下，完成了这个功能

**请慎重在生产环境使用这个功能，因为打印完整的AccessLog是一个很大的消耗**

<!-- more -->


### 项目工程目录结构和代码获取地址

https://github.com/laiyijie/spring-access-log-filter

## 详解核心类**AccessLogFilter**

完整代码如下：

	public class AccessLogFilter extends OncePerRequestFilter {
        private static final Logger logger = LogManager.getLogger(AccessLogFilter.class);
    
        private String usernameKey = "username";
        private Integer payloadMaxLength = 1024;
    
        @Override
        protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response,
                                        FilterChain filterChain) throws ServletException, IOException {
    
            Long startTime = System.currentTimeMillis();
            ContentCachingRequestWrapper requestWrapper = new ContentCachingRequestWrapper(request);
            ContentCachingResponseWrapper responseWrapper = new ContentCachingResponseWrapper(response);
    
            filterChain.doFilter(requestWrapper, responseWrapper);
    
            String requestPayload = getPayLoad(requestWrapper.getContentAsByteArray(),
                    request.getCharacterEncoding());
            String responsePayload = getPayLoad(responseWrapper.getContentAsByteArray(),
                    response.getCharacterEncoding());
            responseWrapper.copyBodyToResponse();
            BLog.accessJsonLogBuilder()
                .addRequestPayLoad(requestPayload)
                .addResponsePayLoad(responsePayload)
                .put(request, usernameKey)
                .put(response)
                .put("_COST_", System.currentTimeMillis() - startTime)
                .log();
        }
    
        private String getPayLoad(byte[] buf, String characterEncoding) {
            String payload = "";
            if (buf == null) return payload;
            if (buf.length > 0) {
                int length = Math.min(buf.length, getPayloadMaxLength());
                try {
                    payload = new String(buf, 0, length, characterEncoding);
                } catch (UnsupportedEncodingException ex) {
                    payload = "[unknown]";
                }
            }
            return payload;
        }
    
    
        public String getUsernameKey() {
            return usernameKey;
        }
    
        public void setUsernameKey(String usernameKey) {
            this.usernameKey = usernameKey;
        }
    
        public Integer getPayloadMaxLength() {
            return payloadMaxLength;
        }
    
        public void setPayloadMaxLength(Integer payloadMaxLength) {
            this.payloadMaxLength = payloadMaxLength;
        }
    }


其核心思想是实现了**OncePerRequestFilter**这个虚基类，而这个虚基类是实现了**Filter**接口，并且要基于SpringMVC框架的，因此这个方法只能适用于SpringMVC工程

### 核心方法是**doFilterInternal**

    protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response,
                                    FilterChain filterChain) throws ServletException, IOException {

        Long startTime = System.currentTimeMillis();
        ContentCachingRequestWrapper requestWrapper = new ContentCachingRequestWrapper(request);
        ContentCachingResponseWrapper responseWrapper = new ContentCachingResponseWrapper(response);

        filterChain.doFilter(requestWrapper, responseWrapper);

        String requestPayload = getPayLoad(requestWrapper.getContentAsByteArray(),
                request.getCharacterEncoding());
        String responsePayload = getPayLoad(responseWrapper.getContentAsByteArray(),
                response.getCharacterEncoding());
        responseWrapper.copyBodyToResponse();
        BLog.accessJsonLogBuilder()
            .addRequestPayLoad(requestPayload)
            .addResponsePayLoad(responsePayload)
            .put(request, usernameKey)
            .put(response)
            .put("_COST_", System.currentTimeMillis() - startTime)
            .log();
    }

首先通过

    ContentCachingRequestWrapper requestWrapper = new ContentCachingRequestWrapper(request);
    ContentCachingResponseWrapper responseWrapper = new ContentCachingResponseWrapper(response);
这两个方式对请求和返回进行包装（让body可以被缓存），这样来解决inputstream不能多次读取的问题

调用

	filterChain.doFilter(requestWrapper, responseWrapper);
来执行其他的filter之后

可以通过

        String requestPayload = getPayLoad(requestWrapper.getContentAsByteArray(),
                request.getCharacterEncoding());
        String responsePayload = getPayLoad(responseWrapper.getContentAsByteArray(),
                response.getCharacterEncoding());

这种方式取出request和response的payload，不要忘记

        responseWrapper.copyBodyToResponse();
重新写入response

最后打印出整个log
