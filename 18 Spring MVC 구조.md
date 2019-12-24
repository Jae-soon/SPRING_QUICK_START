18 Spring MVC 구조
=======================
# 1. Spring MVC 수행 흐름   
대부분의 MVC 프레임워크는 비슷한 구조를 가졌다.    
따라서 하나의 프레임워크만 잘 이해하면 다른 프레임워크도 쉽게 이해할 수 있다.      
   
[사진]
   
**Spring MVC 수행 흐름**   
  
1. 클라이언트로부터 모든 ```.do``` 요청을 DispatcherServlet이 받는다.     
2. DispatcherServlet 은 HandlerMapping을 통해 요청을 처리할 Controller를 검색한다.     
3. DispatcherServlet 은 검색된 Controller를 실행하여 클라이언트의 요청을 처리한다.   
4. Controller는 비즈니스 로직의 수행 결과로 얻어낸 Model 정보와 Model을 보여줄 View 정보를 ModelAndView객체에 저장하여 리턴한다.  
5. DispatcherServlet은 ModelAndView로부터 View 정보를 추출하고, ViewResolver를 이용하여 응답으로 사용할 View를 얻어낸다.  
6. DispatcherServletdms ViewResolver를 통해 찾아낸 View를 실행하여 응답을 전송한다.  

***
# 2. DispatcherSerlvet 등록 및 스프링 컨테이너 구동  
## 2.1. DispatcherServlet 등록
Spring MVC에서 가장 중요한 요소가 모든 클라이언트의 요청을 가장 먼저 받아들이는 DispatcherServlet이다.   
따라서 SpringMVC 적용에서 가장 먼저 해야 할 일은 
WEB-INF/web.xml 파일에 등록된 DispatcherServlet 클래스를 스프링 프레임워크에서 제공하는 DispatcherSerlvet으로 변경하는 것이다.

**web.xml**
```
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xmlns="http://java.sun.com/xml/ns/javaee"
	xsi:schemaLocation="http://java.sun.com/xml/ns/javaee https://java.sun.com/xml/ns/javaee/web-app_2_5.xsd"
	version="2.5">
	<servlet>
		<servlet-name>hello</servlet-name>
		<servlet-class>hello.HelloServlet</servlet-class>
	</servlet>
	<servlet-mapping>
		<servlet-name>hello</servlet-name>
		<url-pattern>/hello.do</url-pattern>
	</servlet-mapping>

	<servlet>
		<servlet-name>action</servlet-name>
		<servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
	</servlet>
	<servlet-mapping>
		<servlet-name>action</servlet-name>
		<url-pattern>*.do</url-pattern>
	</servlet-mapping>
</web-app>
```

***
# 3. 대주제
> 인용
## 3.1. 소 주제
### 3.1.1. 내용1
```
내용1
```
