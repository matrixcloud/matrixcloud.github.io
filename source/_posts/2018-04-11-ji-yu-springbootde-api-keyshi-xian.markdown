---
layout: post
title: "基于SpringBoot的API Key实现"
date: 2018-04-11 17:02:21 +0800
comments: true
categories: Java
---

由于项目需要，需要开放接口并提供SDK给第三方的应用使用，该微服务（以下简称`open-api`），需要实现的功能如下：

- 进行签名认证，防止第三方数据被篡改
- 记录API调用频次，对API调用进行限制
- 提供RESTFUL的API接口

## 表的设计

提供一张`open_key`表作为记录，其中重要的字段为：

- `acccess_key`：在redis中记录API的调用频次（相当于用户名），唯一
- `acccess_secret`：签名算法中使用（相当于用户密码），随机字符串

![](https://ws1.sinaimg.cn/large/006tKfTcgy1fq8tzzbzr3j31580duaan.jpg)

## 签名算法

在网络安全中，永远不要以明文的形式发送密码，以防被恶意截取。笔者的签名算法以较为简单的计算函数实现：`md5(accessKey:acccessSecret:cnonce:body)`。其中，`cnonce`为客户端生成的随机字符串，`body`为`request body`。客户端生成签名后，以下列的形式发送HTTP请求头：

```
Authorization: accessKey=?;cnonce=?;sign=?
```

这样子，在服务端，获取到头部信息后便可采用同样的签名算法进行校验了。

> 为什么签名算法会有效？
> 主要是因为accessSecret没有在网络进行传输，监听者获取不了。所以当监听者试图篡改内容的时候，无法计算出正确的签名，而导致请求失败。
> 可以参考：《HTTP权威指南》的摘要认证章节

<!--more-->

## ApiFilter

在SpringBoot中，通过`FilterRegistrationBean`注册`Filter`，其中`ApiFilter`的核心代码如下：

```
/**
 * Created by atom on 2018/4/9.
 */
@Component
public class ApiFilter implements Filter {
    @Autowired
    private OpenKeyRepository openKeyRepository;

    @Override
    public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) throws IOException, ServletException {
        HttpServletRequest servletRequest = (HttpServletRequest)request;
        String uri = servletRequest.getRequestURI();

        if (uri.startsWith("/api/open/")) {
            CHttpServletRequestWrapper wrapperRequest = new CHttpServletRequestWrapper(servletRequest);
            String authorization = servletRequest.getHeader("Authorization");
            if (StringUtils.isBlank(authorization)) {
                writeError(response, ExceptionCode.NeedAuthorization.value(), "Need authorization header");
                return;
            }
            try {
                String[] array = authorization.split(";");
                Map<String, String> map = new HashMap<>();
                for (String s : array) {
                    String[] kv = s.split("=");
                    map.put(kv[0], kv[1]);
                }
                String accessKey = map.get("accessKey");
                OpenKey openKey = openKeyRepository.findByAccessKey(accessKey);
                if (openKey == null) {
                    writeError(response, ExceptionCode.InvalidAccessKey.value(), "Invalid accessKey");
                    return;
                }
                String accessSecret = openKey.getAccessSecret();
                String cnonce = map.get("cnonce");
                String sign = map.get("sign");

                String body = wrapperRequest.getBody();
                boolean checked = ToolKits.checkSign(accessKey, accessSecret, cnonce, body, sign);
                if (!checked) {
                    writeError(response, ExceptionCode.InvalidSign.value(), "Invalid sign");
                } else {
                    wrapperRequest.addHeader("x-access-key", accessKey);
                    chain.doFilter(wrapperRequest, response);
                }
            } catch (ArrayIndexOutOfBoundsException e) {
                writeError(response, ExceptionCode.InvalidAuthorizationFormat.value(), "Invalid authorization format");
            }
        } else {
            chain.doFilter(servletRequest, response);
        }
    }

    private void writeError(ServletResponse response, int code, String message) {
        try {
            response.setContentType("application/json");
            PrintWriter writer = response.getWriter();
            Map<String, Object> map = new HashMap<>();
            map.put("errorCode", code);
            map.put("message", message);
            writer.print(GsonUtil.toJson(map));
            writer.close();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

## 调用频次计数

笔者在`ApiFilter`中自定义了`x-access-key`头部，这样子，在`Controller`中可以轻松的获取到`accessKey`了，再利用redis.incr方法为`acccessKey`计数，当`accessKey`的频次超出了`call_threshold`直接给客户端抛出异常。

## 后记

服务发布至线上后，可以通过`Nginx`的反向代理配置合理的请求路径。