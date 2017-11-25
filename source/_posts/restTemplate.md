title: Spring RestTemplate关联HttpClient4.5的配置HttpClient和自身的BUG
thumbnail: https://jokers-1252021562.cosgz.myqcloud.com/images/technology/timg.jpeg
date: 2017/06/07 16:21:00
tags: 
    - BUG
    - HttpClient4.5
    - spring RestTemplate
categories:
    - spring
---

  如题 ,本博客将解决RestTemplate 的配置问题 ,同时告知其存在的BUG

写作背景:

我们知道HttpClient要想使用PATCH, PUT等请求 配置将相当麻烦, 当结合RestTemplate后就会变得十分简单. 那么如何进行结合?

<本文基于SpringBoot框架, 相关Spring也可以进行参考设计>

现在进行示例说明:

1.配置方法:

```java
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.http.client.ClientHttpRequestFactory;
import org.springframework.http.client.HttpComponentsClientHttpRequestFactory;
import org.springframework.web.client.RestTemplate;

/**
 * @author yuton
 * @version 1.0
 * @description
 * @since 2017/5/30 13:31
 */
@Configuration
public class HttpClientRestConfig {
    @Bean
    public ClientHttpRequestFactory clientHttpRequestFactory() {
        HttpComponentsClientHttpRequestFactory clientHttpRequestFactory = new HttpComponentsClientHttpRequestFactory();
        clientHttpRequestFactory.setHttpClient(HttpsClientPoolThread.builder().createSSLClientDefault());
        //这里是使用了自定义的一个HttpsClientPoolThread线程池单例 以后有机会会单独写文章展示其配置内容, 大家可以先使用默认的HttpClients.createDefault()进行配置,或自定义线程池;
        clientHttpRequestFactory.setConnectTimeout(10000);
        clientHttpRequestFactory.setReadTimeout(10000);
        clientHttpRequestFactory.setConnectionRequestTimeout(200);
        return clientHttpRequestFactory;
    }

    @Bean
    public RestTemplate restTemplate() {
        return new RestTemplate(clientHttpRequestFactory());
    }
}

```

2.使用方法:



```java

    @Resource
    private CopyWriteUI copyWriteUI;
    @Resource
    private I18nMessageUI i18nMessageUI;
    @Resource
    private RestTemplate restTemplate;
    @Override
    public IResult callPolice(String imei, String onOff) {
        String url = copyWriteUI.getSocketUrl() + APITable.CALL_POLICE_WATCH
                .replace("{imei}", imei)
                .replace("{onOff}", onOff);
        IResult result = restTemplate.patchForObject(url, null, Result.class);
        if (null == result) {
            return CommonTools.errorResult(ResultMessage.ERROR_PROMPT, i18nMessageUI.getNetworkAnomaly());
        }
        if (result.isSuccessful()) {
            return CommonTools.successResult(ResultMessage.STATUS_SUCCESS);
        }
        return CommonTools.errorResult(ResultMessage.ERROR_PROMPT, i18nMessageUI.getNetworkAnomaly());
    }
    
        @Override
    public IResult setWatchStepTime(String imei, String times) {
        String url = copyWriteUI.getSocketUrl() + APITable.SET_WATCH_STEP_TIME.replace("{imei}", imei);
        MultiValueMap<String, String> multiValueMap = new LinkedMultiValueMap<>();
        multiValueMap.add("times", times);
        IResult result = restTemplate.postForObject(url, multiValueMap, Result.class);
        if (null == result) {
            return CommonTools.errorResult(ResultMessage.ERROR_PROMPT, i18nMessageUI.getNetworkAnomaly());
        }
        if (result.isSuccessful()) {
            return CommonTools.successResult(ResultMessage.STATUS_SUCCESS);
        }
        return CommonTools.errorResult(ResultMessage.ERROR_PROMPT, i18nMessageUI.getNetworkAnomaly());
    }

```

在这里 要说明下:

1. post put patch等请求 参数必须使用MultiValueMap进行接收和传递,否则 参数会为空!
2. get请求,如果需要使用Map传递参数,那么该Map一定不能是MultiValueMap! 否则, 传递的参数会附带上'[]'!


