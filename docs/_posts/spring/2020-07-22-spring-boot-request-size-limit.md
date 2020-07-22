---
layout: post
title: "[Spring Boot] request body size limit 삽질기"
author: soonbee
categories: spring
comments: true
---


## 요구사항

request의 body 사이즈가 일정 크기 이상인 경우 요청을 거절하고 싶다는 요구사항이 들어왔다.

spring request size limit 등의 키워드로 검색해보니 방법이 나온다.

<br/>


## 방법

친절하게도 [spring boot 문서](https://docs.spring.io/spring-boot/docs/current/reference/html/appendix-application-properties.html#server-properties)에 관련있어보이는 옵션이 있었다.

| Key                                   | Default | Description                                                |
| ------------------------------------- | ------- | ---------------------------------------------------------- |
| server.tomcat.max-http-form-post-size | 2MB     | Maximum size of the form content in any HTTP post request. |
| server.tomcat.max-swallow-size        | 2MB     | Maximum amount of request body to swallow.                 |

위 내용대로 application.yaml 파일을 수정하였다. 10MB 등 단위를 같이 적을 수 있어서 편하다.

<br/>


사용하는 Spring Boot version은 2.2.0 이며 2.x 보다 하위 버전은 아래를 참고하면 되겠다.

**Spring Boot 1.3.x and earlier**

* multipart.maxFileSize
* multipart.maxRequestSize

**Spring Boot 1.4.x and 1.5.x**

* spring.http.multipart.maxFileSize
* spring.http.multipart.maxRequestSize

<br/>


방법 적용!

.

.

.

그러나 이렇게 간단하게 끝날 문제였으면 삽질기라며 포스트를 적지도 않았을거다.


<br/>

## 무언가 잘못되었다

작동하지 않는다. 100mb 크기의 데이터도 잘 보내진다

도대체 왜?!

애초에 default value가 2MB인데, 그동안 2MB 이상의 데이터도 잘 주고받았다.

뭔가 이상하다.


<br/>

## 이유

검색의 검색 끝에 원인을 찾아보니 해당 옵션은 form data인 경우에만 적용된다고 한다.

즉, Content-Type이 `multipart/form-data` 혹은 `application/x-www-form-urlencoded` 인 경우에만 정상적으로 작동한다는 것.

아니 분명 문서에 "any HTTP post request" 라고 적혀 있잖아요. any! any!! any!!!!!

spring boot github repository에도 해당 문제로 [설명을 수정해야 한다는 이슈](https://github.com/spring-projects/spring-boot/issues/18521)가 등록되어 있다


<br/>

## 테스트

`Content-Type`을 `multipart/form-data` 로 보냈더니 아래와 같이 exception이 발생하는 것을 확인할 수 있었다

```
org.apache.tomcat.util.http.fileupload.FileUploadBase$SizeLimitExceededException: the request was rejected because its size (30720000) exceeds the configured maximum (10485760)
...
```



 `Content-Type`을 `application/x-www-form-urlencoded` 의 경우 exception이 발생하지는 않고 body가 비어있는 상태로 오는 것을 확인했다. exception이 발생하는 게 맞는 것 같은데, 왜 그런지는 잘 모르겠다. 


<br/>

## 그럼 다른 content에 대한 해결책은 없는가?

`application/json` 등 다른 `Content-Type` 에 대해서는 filter를 통해서 처리할 수 있다.

header의 `Content-Length` 값을 읽어서 특정 limitation 보다 크다면 exception 을 발생시키면 간단하게 해결되겠다.

```java
@Component
static class ApplicationJsonRequestSizeLimitFilter extends OncePerRequestFilter {

    @Override
    protected void doFilterInternal(HttpServletRequest request,
            HttpServletResponse response, FilterChain filterChain)
                    throws ServletException, IOException {
        if (isApplicationJson(request) && request.getContentLengthLong() > 10000) {
            throw new IOException("Request content exceeded limit of 10000 bytes");
        }
        filterChain.doFilter(request, response);
    }

    private boolean isApplicationJson(HttpServletRequest httpRequest) {
        return (MediaType.APPLICATION_JSON.isCompatibleWith(MediaType
                .parseMediaType(httpRequest.getHeader(HttpHeaders.CONTENT_TYPE))));
    }

}
```


<br/>

### 참조

* https://stackoverflow.com/questions/33806863/restrict-the-size-of-post-request-application-json-in-tomcat-7

* https://stackoverflow.com/questions/38607159/spring-boot-embedded-tomcat-application-json-post-request-restriction-to-10kb?answertab=active#tab-top
* https://gist.github.com/jays1204/703297eb0da1facdc454
* https://stackoverflow.com/questions/23118249/whats-the-difference-between-request-payload-vs-form-data-as-seen-in-chrome
* https://server0.tistory.com/41
