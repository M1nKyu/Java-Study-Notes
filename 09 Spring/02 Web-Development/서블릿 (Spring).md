---
created: 2025-02-14 17:19
category:
  - spring
tags:
  - TIL
last_modified: 2025-02-14T18:46:00
---
## ⭐ Spring 서블릿(Servlet)이란?
### 🍪 개요
- 자바를 사용하여 웹페이지를 동적으로 생성하는 서버측 프로그램.
- 클라이언트의 요청을 처리하고 결과를 반환하는 자바 웹 프로그래밍 기술의 기초. 
- Spring MVC 아키텍처의 중심 요소로, 클라이언트의 HTTP 요청을 적절한 컨트롤러로 전달하고, 응답을 생성하는 역할을 수행한다.
---
### 🍪서블릿 특징
 >- 일반적으로 웹 서버는 정적 페이지만을 제공한다. 동적 페이지를 제공하기 위해서 웹서버는 다른 곳에 도움을 요청하여 동적 페이지를 작성해야 한다.
> - 여기서 웹 서버가 동적 페이지를 제공할 수 있도록 도와주는 애플리케이션이 서블릿이며, 동적인 페이지를 생성하는 애플리케이션이 CGI이다.
#### 동적 웹 페이지 처리
- HTML, JSON, XML 등 다양한 형식의 응답을 생성할 수 있다.
#### Java Thread를 이용하여 동작
- 서블릿은 멀티스레드 환경에서 동작한다.
- 서블릿 컨테이너(ex: Apache Tomcat)는 각 요청을 별도의 스레드에서 처리한다.
#### MVC 패턴에서 Controller로 이용
- 서블릿은 클라이언트의 요청을 분석하고, 비즈니스 로직 실햏ㅇ 후 결과를 View로 전달한다.
#### HTTP 프로토콜 서비스 지원
- HTTP 프로토콜 서비스를 지원하는 `javax.servlet.http.HttpServlet` 클래스를 상속받는다.
- HTTP 메서드에 따라 `doGet()`, `doPost()` 등의 메서드를 오버라이드하여 요청을 처리한다.
#### UDP보다 처리속도가 느림
- TCP/IP 기반의 HTTP 프로토콜을 사용한다.
#### HTML 변경 시 서블릿을 재컴파일해야 하는 단점
- 이를 보완하기 위해 JSP나 템플릿 엔진(ex: Thymeleaf) 등이 사용된다.
---
### 🍪 웹 서버와 서블릿의 관계
#### 웹 서버의 역할
- 웹 서버 (Apache, Nginx)는 정적 콘텐츠를 클라이언트에게 제공한다.
- 동적 콘텐츠를 제공할 수 없다.
#### 동적 페이지를 제공하기 위한 방법 -> 서블릿과 CGI
- 웹 서버는 동적 페이지를 제공하기 위해 외부 애플리케이션의 도움을 받아야 한다.
- 이때 서블릿과 CGI(Common Gateway Interface)를 사용할 수 있다.
#### 서블릿 vs CGI
- 서블릿
	- Java 기반으로 동작, 서블릿 컨테이너(ex: Tomcat)에서 실행된다.
	- 멀티스레드 환경에서 동작한다 -> CGI보다 빠르고 효율적이다.
	- 서블릿은 메모리에 상주한다 -> CGI보다 성능이 우수하다.
- CGI
	- 웹 서버가 외부 프로그램을 호출하여 동적 페이지를 생성한다.
	- 각 요청마다 새로운 프로세스를 생성한다 -> 서블릿보다 느리고 리소스 사용이 많다.
	- Perl, Python, PHP 등 다양한 언어로 구현할 수 있다.
---
### 🍪 서블릿 컨테이너
- 서블릿 컨테이너는 말그대로 서블릿을 담고 관리해주는 컨테이너.
- 서블릿을 관리하며, 클라이언트에서 요청을 하면 HttpServletRequest, HttpServletResponse 두 객체를 생성하여 POST, GET 여부에 따라 동적인 페이지를 생성하여 응답한다.

#### 서블릿 컨테이너의 주요 기능
1. 서블릿 생명주기 관리
	- 서블릿의 생명주기를 관리한다.
2. 웹서버와의 통신 지원
	- 서블릿과 웹서버가 손쉽게 통신할 수 있게 해준다.
	- 클라이언트의 Request를 받고 Response를 보낼 수 있도록 웹 서버와 소켓을 만들어 통신을 해준다.
	- 일반적으로 통신은 소켓 생성 -> 특정 포트 리스닝 -> 연결 요청 시 스트림 생성 후 요청을 받음 
		- 이 과정을 서블릿 컨테이너가 대신해준다.
		- 개발자로서 비즈니스 로직에 집중할 수 있게 해준다.
3. 멀티스레딩 관리
	- 서블릿 컨테이너는 서블릿 요청이 들어오면 스레드를 생성해서 작업을 수행한다.
	- 그래서 동시에 여러 요청이 들어와도 멀티스레딩 환경으로 작업을 관리할 수 있다.
4. 선언적인 보안관리
5. 서블릿 컨테이너는 보안 기능을 지원하므로, 서블릿 또는 자바 클래스에 보안 관련된 메서드를 직접 구현하지 않아도 된다.
---
### 🍪 서블릿 동작 방식
- 서블릿은 클라이언트의 요청을 받아 처리하고, 그 결과를 응답으로 반환하는 역할을 한다. 서블릿 컨테이너는 서블릿의 생명 주기를 관리한다.
- 서블릿은 요청이 들어올 때마다 생성되는 것이 아닌, 초기화된 후 여러 요청을 처리할 수 있다. (메모리 사용 최적화, 성능 향상에 기여)
	- 서블릿은 요청당 프로세스 생성 대신, 요청당 스레드를 생성하기 때문이다.
- 서블릿의 동작은 
	- 클라이언트 요청 -> 서블릿 컨테이너가 해당 서블릿 호출 -> 요청을 처리한 후 응답 반환
#### 동작 과정
- ![과정|500x250](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Ft1.daumcdn.net%2Fcfile%2Ftistory%2F993A7F335A04179D20)
	- [이미지 출처](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Ft1.daumcdn.net%2Fcfile%2Ftistory%2F993A7F335A04179D20)
1. Servlet Request, Servlet Response 객체를 생성한다.
2. 설정 파일을 참고하여 매핑할 Servlet을 확인한다.
3. 해당 서블릿 인스턴스 존재의 유무를 확인 -> 없으면 `init()` 메서드를 호출하여 생성한다.
4. Servlet Container에 스레드를 생성하고 service를 실행한다.
5. 응답을 처리하였으면 `destroy()` 메서드를 실행하여 Servlet Request, Servlet Response 객체를 소멸시킨다.
---
### 🍪 서블릿의 생명주기
> - 서블릿 객체가 생성되고, 요청을 처리하며, 소멸될 때까지의 과정
> - 생명주기는 서블릿 컨테이너가 자동으로 관리한다.
1. 서블릿 객체 생성(`init()`)
	- 최초 요청이 오면 서블릿 인스턴스가 생성되고, 한 번만 실행된다.
2. 요청 처리(`service()`)
	- 클라이언트가 요청을 보낼 때마다 실행된다. (`doGet()`, `doPost()` 호출)
3. 서블릿 종료 (`destroy()`)
	- 웹 애플리케이션 종료 또는 서블릿 제거 시 실행된다.
```java
public class ExampleServlet extends HttpServlet {
    // 1. 객체 생성 단계
    public void init(ServletConfig config) throws ServletException {
        super.init(config);
        // 서블릿 초기화 코드
    }

    // 2. 서비스 단계
    protected void doGet(HttpServletRequest request, HttpServletResponse response) 
    throws ServletException, IOException {
        // GET 요청 처리 로직
    }

    protected void doPost(HttpServletRequest request, HttpServletResponse response) 
    throws ServletException, IOException {
        // POST 요청 처리 로직
    }

    // 3. 종료 단계
    public void destroy() {
        // 리소스 정리 코드
    }
}
```
---
### 서블릿과 설정 파일 (web.xml)
> - `web.xml`은 
> 	- Java 웹 애플리케이션의 배포 설정 파일로, 서블릿 컨테이너가 애플리케이션을 실행하는 데 필요한 정보를 담는다.
> 	- 서블릿, 필터, 리스터 등을 사용하는데 사용한다.

- 생성한 서블릿을 사용자가 요청한 경로와 매핑시켜줘야 요청을 처리할 수 있다.
```xml
<?xml version="1.0" encoding="UTF-8"?>

<web-app xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
     xmlns="http://xmlns.jcp.org/xml/ns/javaee"
     xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_3_1.xsd"
     version="3.1">
		
     <servlet> //서블릿 클래스를 서블릿으로 등록
           <servlet-name>myServlet</servlet-name> //해당 서블릿을 참조할 때 사용할 이름
           <servlet-class>controller.MyServlet</servlet-class> //서블릿으로 사용할 서블릿 클래스의 FullName
     </servlet>
 
     <servlet-mapping>
           <servlet-name>myServlet</servlet-name> //매핑할 서블릿의 이름
           <url-pattern>/myServlet</url-pattern> //매핑할 URL 패턴
     </servlet-mapping>
 
</web-app>
```

---
### 🍪 JSP(Java Server Pages) 란?
> HTML 내에 Java 코드를 포함하여 동적 웹페이지를 생성할 수 있는 기술.

- 서블릿은 자바 코드 속에 HTML 코드가 들어가는 형태,
	- JSP는 HTML 코드 속에 자바 코드가 들어가 있다.

- `<% 소스코드 %>` 또는 `<%= 소스코드 =%>` 형태.
	- 이 부분은 브라우저가 아닌 웹 서버에서 실행되는 부분이다.
	- 컴파일과 같은 과정이 필요없다.

- 서블릿의 복잡한 규칙으로 JSP가 등장했다.
- JSP는 WAS(Web Application Server)에 의해 서블릿 클래스로 변환되어 사용된다.

#### 서블릿 동작 과정
1. 클라이언트가 웹 서버에 요청한다. (`http://example.com/hello`)
2. 웹 컨테이너(Tomcat)가 서블릿 객체를 생성한다.
3. `service()` → `doGet()` or `doPost()` 메서드 실행한다.
4. `response.getWriter().println("<html>...</html>")` 방식으로 HTML 코드 생성한다.
5. 클라이언트에게 HTML 응답을 전송한다.
```java
// 서블릿 예제
@WebServlet("/hello")
public class HelloServlet extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest request, HttpServletResponse response) 
            throws ServletException, IOException {
        response.setContentType("text/html");
        PrintWriter out = response.getWriter();
        out.println("<html><body>");
        out.println("<h1>Hello, Servlet!</h1>");
        out.println("</body></html>");
    }
}
```

#### JSP 동작 과정
1. 클라이언트가 `.jsp` 페이지를 요청한다. (`http://example.com/hello.jsp`)
2. JSP 파일이 서블릿으로 변환된다. (`Hello_jsp.java` 생성)
3. 변환된 서블릿이 컴파일되고 실행된다.
4. HTML 코드가 포함된 응답을 클라이언트에 전송한다.
```html
<!--JSP 코드-->
<html>
<body>
    <h1>현재 시간: <%= new java.util.Date() %></h1>
</body>
</html>
```
```java
// 서블릿으로 변환된 JSP 코드
public class Hello_jsp extends HttpServlet {
    protected void service(HttpServletRequest request, HttpServletResponse response)
            throws ServletException, IOException {
        response.setContentType("text/html");
        PrintWriter out = response.getWriter();
        out.println("<html><body>");
        out.println("<h1>현재 시간: " + new java.util.Date() + "</h1>");
        out.println("</body></html>");
    }
}
```

#### 서블릿 vs JSP (표)

| 비교 항목           | **JSP(Java Server Pages)**       | **서블릿(Servlet)**                 |
| --------------- | -------------------------------- | -------------------------------- |
| **역할**          | 웹 페이지(View) 구현                   | 비즈니스 로직(Controller) 처리           |
| **구성**          | HTML + Java 코드                   | 순수 Java 코드                       |
| **파일 확장자**      | `.jsp`                           | `.java`                          |
| **코딩 스타일**      | HTML 중심, Java 코드 포함 (`<% %>` 사용) | Java 중심, HTML은 `PrintWriter`로 출력 |
| **초기 실행 과정**    | 첫 실행 시 서블릿으로 변환됨                 | 처음부터 서블릿 클래스 작성                  |
| **속도**          | 서블릿 변환 후 실행되므로 초기 실행 시 약간 느림     | 직접 Java 코드로 실행되므로 초기 실행 속도가 빠름   |
| **유지보수**        | HTML 중심이라 유지보수 용이                | HTML 코드 포함 시 가독성이 떨어짐            |
| **MVC 패턴에서 역할** | **View (화면 출력)** 담당              | **Controller (로직 처리)** 담당        |

---
## 🧙‍♂️ 요약
- 서블릿이란
	- 서버 측에서 실행되는 Java 프로그램, 클라이언트의 HTTP 요청을 처리하고 응답을 생성한다.
	- 멀티스레드 환경에서 동작한다.

- 서블릿 동작 과정
	1. 클라이언트 요청 -> 웹 서버(Tomcat)가 서블릿 컨테이너에 전달.
	2. 서블릿 컨테이너가 서블릿 객체 생성.(최초 요청 시)
	3. `service()` 실행 -> 요청에 맞는 `doGet()` 또는 `doPost()` 호출.
	4. HTML/JSON 응답 생성 -> 클라이언트에게 반환

- JSP (Java Server Pages)
	- HTML 내의 Java 코드 -> 동적 웹페이지 생성
	- JSP 실행 -> 서블릿으로 변환

> 현업에서는 JSP 대신 Thymeleaf [[템플릿 엔진]]을 많이 사용한다.
---
## 📝 추가 학습
- [[템플릿 엔진]]
- [[필터와 인터셉터]]
---
> **참고사이트**
> - [01-서블릿이란 무엇인가?](https://coding-factory.tistory.com/742)
> - [02-스프링 MVC와 서블릿 동작 이해](https://f-lab.kr/insight/understanding-spring-mvc-and-servlet-20241111?gad_source=1)
> - [03-자바 서블릿과 스프링 프레임워크의 이해](https://f-lab.kr/insight/understanding-java-servlet-and-spring-framework?gad_source=1)
> - [04-서블릿이란?](https://mangkyu.tistory.com/14)
