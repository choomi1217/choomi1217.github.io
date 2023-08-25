---
title:  "[개발 기록] Spring Servlet"

categories:
  - log
tags:
  - [log, spring-log]

toc: true
toc_sticky: true

breadcrumbs: true

date: 2023-08-25
last_modified_at: 2023-08-25
---

이직을 준비 하면서
**Spring의 핵심 기본기 다지기**

[CGI와 서블릿, JSP의 연관관계 알아보기](https://velog.io/@suhongkim98/CGI와-서블릿-JSP의-연관관계-알아보기)

[[10분 테코톡] 🌻타미의 Servlet vs Spring](https://www.youtube.com/watch?v=2pBsXI01J6M)

# 🗒️ 용어 정리

### CGI

Common Gateway Interface

정적 리소스만 제공하던 웹서버가 사용자의 요청을 동적으로 처리해서 결과물을 보여줄 수 있게 합니다.

하지만 이 CGI는 요청당 하나의 프로세스를 만들어 메모리 낭비가 심하다는 단점이 있습니다.

이를 보완해서 요청당 스레드를 분리한 서블릿이 나오게 됩니다.

또한 이때 스레드당 따로 생성되는 CGI를 Single tone을 이용해 처리하게 됩니다.

### 서블릿

http request를 받고 response를 돌려주는 역할을 하는 자바 클래스

**`javax.servlet.http.HttpServlet`** 클래스를 상속받아 구현하며, 클라이언트의 요청 유형에 따라**`doGet()`**, **`doPost()`**, **`doPut()`**, **`doDelete()`** 등의 메서드를 오버라이드하여 구현합니다.

스프링은 **`DispatcherServlet`**이라는 서블릿을 제공합니다.

**`@GetMapping`**, **`@PostMapping`** 등의 어노테이션은 각 메소드가 처리할 HTTP 요청의 유형을 명시합니다.

**`DispatcherServlet`**은 이런 어노테이션을 바탕으로 요청을 적절한 메소드로 라우팅합니다.

### 서블릿 컨테이너

Tomcat, Jetty 등등..

**서블릿의 생명주기를 관리**

**서블릿 간 공유 리소스를 제공**

**웹 서버와 통신하여 HTTP 요청을 받고 응답을 반환하는 역할**

1. 클라이언트로부터 HTTP 요청이 들어오면, Tomcat의 웹 서버 컴포넌트인 Connector가 이 요청을 받습니다.
2. Connector는 요청을 분석하여 해당하는 서블릿을 찾습니다.
3. Connector는 요청에 해당하는 서블릿이 찾아지면, **새로운 Java Thread를 생성**하고 그 Thread에서 서블릿의 service() 메소드를 호출합니다. service() 메소드는 요청의 HTTP 메소드(GET, POST 등)에 따라 적절한 메소드(doGet(), doPost() 등)를 호출합니다.
4. 서블릿이 로직을 처리하고 HTTP 응답을 생성합니다.
5. 생성된 HTTP 응답은 Connector를 통해 다시 클라이언트에게 전달됩니다.

# 직접 보기

*org.apache.catalina.servlet*

Servlet 클래스에 적힌 내용

> **모든 서블릿이 구현해야 하는 메소드를 정의합니다.
서블릿은 웹 서버 내에서 실행되는 작은 Java 프로그램입니다. 서블릿은 일반적으로 HTTP(HyperText Transfer Protocol)를 통해 웹 클라이언트의 요청을 수신하고 이에 응답합니다.
이 인터페이스를 구현하기 위해 jakarta.servlet.GenericServlet 을 확장하는 일반 서블릿 또는 jakarta.servlet.http.HttpServlet 을 확장하는 HTTP 서블릿을 작성할 수 있습니다.
이 인터페이스는 서블릿을 초기화하고 요청을 처리하며 서버에서 서블릿을 제거하는 방법을 정의합니다. 이들은 수명 주기 메서드로 알려져 있으며 다음 순서로 호출됩니다.
1. 서블릿이 생성된 다음 init 메서드로 초기화됩니다.
2. service 메서드에 대한 클라이언트의 모든 호출이 처리됩니다.
3. 서블릿은 서비스에서 제외되고 destroy 메소드로 파괴된 다음 가비지 수집 및 종료됩니다.**
>

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/6232dedf-f9eb-4436-be6e-2bc3d729465c/Untitled.png)

# init
> 서블릿 컨테이너는 서블릿을 인스턴스화한 후 정확히 한 번 init 메서드를 호출합니다.
init 메소드는 서블릿이 요청을 수신하기 전에 성공적으로 완료되어야 합니다.
>

```java
//추상 HttpServlet
@Override
    public void init(ServletConfig config) throws ServletException {
        super.init(config);
        cachedUseLegacyDoHead = Boolean.parseBoolean(config.getInitParameter(LEGACY_DO_HEAD));
    }

//구현 CGIServlet
@Override
    public void init(ServletConfig config) throws ServletException {

        super.init(config);

        // Set our properties from the initialization parameters
        cgiPathPrefix = getServletConfig().getInitParameter("cgiPathPrefix");
        boolean passShellEnvironment =
            Boolean.parseBoolean(getServletConfig().getInitParameter("passShellEnvironment"));

        if (passShellEnvironment) {
            shellEnv.putAll(System.getenv());
        }

        Enumeration<String> e = config.getInitParameterNames();
        while(e.hasMoreElements()) {
            String initParamName = e.nextElement();
            if (initParamName.startsWith("environment-variable-")) {
                if (initParamName.length() == 21) {
                    throw new ServletException(sm.getString("cgiServlet.emptyEnvVarName"));
                }
                shellEnv.put(initParamName.substring(21), config.getInitParameter(initParamName));
            }
        }

        if (getServletConfig().getInitParameter("executable") != null) {
            cgiExecutable = getServletConfig().getInitParameter("executable");
        }

        if (getServletConfig().getInitParameter("executable-arg-1") != null) {
            List<String> args = new ArrayList<>();
            for (int i = 1;; i++) {
                String arg = getServletConfig().getInitParameter(
                        "executable-arg-" + i);
                if (arg == null) {
                    break;
                }
                args.add(arg);
            }
            cgiExecutableArgs = args;
        }

        if (getServletConfig().getInitParameter("parameterEncoding") != null) {
            parameterEncoding = getServletConfig().getInitParameter("parameterEncoding");
        }

        if (getServletConfig().getInitParameter("stderrTimeout") != null) {
            stderrTimeout = Long.parseLong(getServletConfig().getInitParameter(
                    "stderrTimeout"));
        }

        if (getServletConfig().getInitParameter("envHttpHeaders") != null) {
            envHttpHeadersPattern =
                    Pattern.compile(getServletConfig().getInitParameter("envHttpHeaders"));
        }

        if (getServletConfig().getInitParameter("enableCmdLineArguments") != null) {
            enableCmdLineArguments =
                    Boolean.parseBoolean(config.getInitParameter("enableCmdLineArguments"));
        }

        if (getServletConfig().getInitParameter("cgiMethods") != null) {
            String paramValue = getServletConfig().getInitParameter("cgiMethods");
            paramValue = paramValue.trim();
            if ("*".equals(paramValue)) {
                cgiMethodsAll = true;
            } else {
                String[] methods = paramValue.split(",");
                for (String method : methods) {
                    String trimmedMethod = method.trim();
                    cgiMethods.add(trimmedMethod);
                }
            }
        } else {
            cgiMethods.add("GET");
            cgiMethods.add("POST");
        }

        if (getServletConfig().getInitParameter("cmdLineArgumentsEncoded") != null) {
            cmdLineArgumentsEncodedPattern =
                    Pattern.compile(getServletConfig().getInitParameter("cmdLineArgumentsEncoded"));
        }

        String value = getServletConfig().getInitParameter("cmdLineArgumentsDecoded");
        if (ALLOW_ANY_PATTERN.equals(value)) {
            // Optimisation for case where anything is allowed
            cmdLineArgumentsDecodedPattern = null;
        } else if (value != null) {
            cmdLineArgumentsDecodedPattern = Pattern.compile(value);
        }
    }
```

- cgiPathPrefix
- passShellEnvironment
- environment-variable-***
- parameterEncoding
- stderrTimeout
- envHttpHeaders
- enableCmdLineArguments
- cgiMethods

# service
> 서블릿이 요청에 응답할 수 있도록 서블릿 컨테이너에서 호출합니다.
서블릿은 일반적으로 여러 요청을 동시에 처리할 수 있는 다중 스레드 서블릿 컨테이너 내에서 실행됩니다. 개발자는 파일, 네트워크 연결, 서블릿의 클래스 및 인스턴스 변수와 같은 공유 리소스에 대한 액세스를 동기화해야 합니다.
Params:
req – 클라이언트의 요청을 포함하는 ServletRequest 객체
res – 서블릿의 응답을 포함하는 ServletResponse 객체
Throws:
ServletException – 서블릿의 정상 작동을 방해하는 예외가 발생하는 경우
IOException – 입력 또는 출력 예외가 발생한 경우
>

```java
// ServletRequest를 받아 오버라이드한 메소드
@Override
    public void service(ServletRequest req, ServletResponse res) throws ServletException, IOException {

        HttpServletRequest request;
        HttpServletResponse response;

        try {
            request = (HttpServletRequest) req;
            response = (HttpServletResponse) res;
        } catch (ClassCastException e) {
            throw new ServletException(lStrings.getString("http.non_http"));
        }
        service(request, response);
    }

// HttpServletRequest를 받은 메소드
protected void service(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {

        String method = req.getMethod();

        if (method.equals(METHOD_GET)) {
            long lastModified = getLastModified(req);
            if (lastModified == -1) {
                // servlet doesn't support if-modified-since, no reason
                // to go through further expensive logic
                doGet(req, resp);
            } else {
                long ifModifiedSince;
                try {
                    ifModifiedSince = req.getDateHeader(HEADER_IFMODSINCE);
                } catch (IllegalArgumentException iae) {
                    // Invalid date header - proceed as if none was set
                    ifModifiedSince = -1;
                }
                if (ifModifiedSince < (lastModified / 1000 * 1000)) {
                    // If the servlet mod time is later, call doGet()
                    // Round down to the nearest second for a proper compare
                    // A ifModifiedSince of -1 will always be less
                    maybeSetLastModified(resp, lastModified);
                    doGet(req, resp);
                } else {
                    resp.setStatus(HttpServletResponse.SC_NOT_MODIFIED);
                }
            }

        } else if (method.equals(METHOD_HEAD)) {
            long lastModified = getLastModified(req);
            maybeSetLastModified(resp, lastModified);
            doHead(req, resp);

        } else if (method.equals(METHOD_POST)) {
            doPost(req, resp);

        } else if (method.equals(METHOD_PUT)) {
            doPut(req, resp);

        } else if (method.equals(METHOD_DELETE)) {
            doDelete(req, resp);

        } else if (method.equals(METHOD_OPTIONS)) {
            doOptions(req, resp);

        } else if (method.equals(METHOD_TRACE)) {
            doTrace(req, resp);

        } else {
            //
            // Note that this means NO servlet supports whatever
            // method was requested, anywhere on this server.
            //

            String errMsg = lStrings.getString("http.method_not_implemented");
            Object[] errArgs = new Object[1];
            errArgs[0] = method;
            errMsg = MessageFormat.format(errMsg, errArgs);

            resp.sendError(HttpServletResponse.SC_NOT_IMPLEMENTED, errMsg);
        }
    }
```

# destroy
> 서블릿이 요청에 응답할 수 있도록 서블릿 컨테이너에서 호출합니다.
서블릿은 일반적으로 여러 요청을 동시에 처리할 수 있는 다중 스레드 서블릿 컨테이너 내에서 실행됩니다. 개발자는 파일, 네트워크 연결, 서블릿의 클래스 및 인스턴스 변수와 같은 공유 리소스에 대한 액세스를 동기화해야 합니다.
Params:
req – 클라이언트의 요청을 포함하는 ServletRequest 객체
res – 서블릿의 응답을 포함하는 ServletResponse 객체
Throws:
ServletException – 서블릿의 정상 작동을 방해하는 예외가 발생하는 경우
IOException – 입력 또는 출력 예외가 발생한 경우
>

```java
// ServletRequest를 받아 오버라이드한 메소드
@Override
    public void service(ServletRequest req, ServletResponse res) throws ServletException, IOException {

        HttpServletRequest request;
        HttpServletResponse response;

        try {
            request = (HttpServletRequest) req;
            response = (HttpServletResponse) res;
        } catch (ClassCastException e) {
            throw new ServletException(lStrings.getString("http.non_http"));
        }
        service(request, response);
    }

// HttpServletRequest를 받은 메소드
protected void service(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {

        String method = req.getMethod();

        if (method.equals(METHOD_GET)) {
            long lastModified = getLastModified(req);
            if (lastModified == -1) {
                // servlet doesn't support if-modified-since, no reason
                // to go through further expensive logic
                doGet(req, resp);
            } else {
                long ifModifiedSince;
                try {
                    ifModifiedSince = req.getDateHeader(HEADER_IFMODSINCE);
                } catch (IllegalArgumentException iae) {
                    // Invalid date header - proceed as if none was set
                    ifModifiedSince = -1;
                }
                if (ifModifiedSince < (lastModified / 1000 * 1000)) {
                    // If the servlet mod time is later, call doGet()
                    // Round down to the nearest second for a proper compare
                    // A ifModifiedSince of -1 will always be less
                    maybeSetLastModified(resp, lastModified);
                    doGet(req, resp);
                } else {
                    resp.setStatus(HttpServletResponse.SC_NOT_MODIFIED);
                }
            }

        } else if (method.equals(METHOD_HEAD)) {
            long lastModified = getLastModified(req);
            maybeSetLastModified(resp, lastModified);
            doHead(req, resp);

        } else if (method.equals(METHOD_POST)) {
            doPost(req, resp);

        } else if (method.equals(METHOD_PUT)) {
            doPut(req, resp);

        } else if (method.equals(METHOD_DELETE)) {
            doDelete(req, resp);

        } else if (method.equals(METHOD_OPTIONS)) {
            doOptions(req, resp);

        } else if (method.equals(METHOD_TRACE)) {
            doTrace(req, resp);

        } else {
            //
            // Note that this means NO servlet supports whatever
            // method was requested, anywhere on this server.
            //

            String errMsg = lStrings.getString("http.method_not_implemented");
            Object[] errArgs = new Object[1];
            errArgs[0] = method;
            errMsg = MessageFormat.format(errMsg, errArgs);

            resp.sendError(HttpServletResponse.SC_NOT_IMPLEMENTED, errMsg);
        }
    }
```
# debug
1. gradlew clean build
2. java -jar build/libs/spring-servlet.jar
3. SpringApplication.java
  1. init
  2. run

![Untitled](../image/servlet-debug.png)

1. 요청

```java
###
GET http://localhost:8080/hello

###
POST http://localhost:8080/hello
Content-Type: application/json

{
  "name": "pony"
}

@Controller
public class HelloWorld {
    @GetMapping("/hello")
    public String hello() {
        return "hello";
    }

    @PostMapping("/hello")
    public String helloPost(HelloRequest helloRequest) {
        return "반갑습니다! " + helloRequest.name;
    }
}
```

1. TomcatWebServer
2. LifeCycleBase