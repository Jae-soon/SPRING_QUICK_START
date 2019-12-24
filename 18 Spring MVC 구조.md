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
## 2.2. 스프링 컨테이너 구동  
클라이언트의 요청으로 DispatcherServlet 객체가 생성되고 나면  
DispatcherServlet 클래스에 재정의된 Init() 메소드가 자동으로 실행되어   
XmlWebApplicationContext라는 스프링 컨테이너가 구동된다.
  
XmlWebApplicationContext는 AppicationContext를 구현한 클래스중 하나이다.  
하지만 XmlWebApplicationContext는 우리가 직접 생성하는 것이 아니라 DispatcherServlet이 생성한다.  

SpringMVC의 구성 요소 중에서 DispatcherServlet 클래스가 유일한 서블릿이다.  
따라서 서블릿 컨테이너는 web.xml 파일에 등록된 DispatcherServlet만 생성해준다.    
   
하지만 DispatcherServlet 객체 혼자서는 클라이언트의 요청을 처리할 수 없고,   
반드시 HandlerMapping, Controller, ViewResolver 객체들과 상호작용해야 한다.  
이 객체들을 메모리에 생성하기 위해서 DispatcherServlet은 스프링 컨테이너를 구동하는 것이다.    

우리가 직접 DispatcherServlet 클래스를 개발했을 때는 ```init()``` 메소드에서   
DispatcherServlet이 사용하는 HandlerMapping, Controller, ViewResolver 객체들을 생성했다.    
다만, 스프링에서 제공하는 DispatcherServlet은 스프링 컨테이너를 통해 이 객체들을 생성하는것이 다를 뿐이다.  
```
결국 DispatcherServlet은 클라이언트의 요청 처리에 필요한  
HandlerMapping, Controller, ViewResolver 객체들을 생성하기 위해 스프링 컨테이너를 구동한다.  
``` 
서블릿 컨테이너가 DispatcherServlet 객체를 생성하고 나면 재정의된 init() 메소드가 자동으로 실행된다.    
그러면 init() 메소드는 스프링 설정 파일을 로딩하여 XmlWebApplicationContext를 생성한다.   
즉, 스프링 컨테이너가 구동되는 것이다.  
결국 스프링 설정 파일에 DispatcherServlet이 사용할 HandlerMapping, Controller, ViewResolver 클래스를 
```<bean>```에 등록하면 스프링 컨테이너가 해당 객체들을 생성해준다.  
```
그렇다면 우리는 스프링 설정파일에 해당 클래스들을 등록해주자
```
## 2.3. 스프링 설정 파일 등록
아무것도 조작하지 않은 현재 상태에서는 DispatcherServlet 이 스프링 컨테이너를 구동할 때 무조건
```/WEB-INF/action-servlet.xml```파일을 찾아 로딩한다.   
하지만 해당 위치에 ```action-servlet.xml``` 없으므로 FileNotFoundException이 발생한다.   

DispathcerServlet은 Spring 컨테이너를 구동할 때, web.xml 파일에 등록된 서블릿 이름 뒤에  
```-servlet.xml```을 붙여서 스프링 설정 파일을 찾는다.   
따라서 ```web.xml```파일에 등록된 DispatcherServlet 이름이 dispatcher였다면 ```/WEB-INF/dispatcher-servlet.xml```파일을 찾았을 것이다.  

이제 DispatcherServlet이 스프링 컨테이너를 구동할 때 로딩할 스프링 설정 파일을 추가하자.  
   
1. 이클립스의 프로젝트 탐색 창에서 WEB-INF 폴더에 마우스 오른쪽 버튼을 클릭하여 ```[New]->[Other]```메뉴를 선택한다.   
2. Spring 폴더에서 'Spring Bean configuration file'을 선택하고 ```<Next>```를 클릭한다.  
3. 'File name'에 'action-servlet.xml'파일명을 입력하고 ```<Finish>```를 클릭하면 WEB-INF 폴더에 action-servlet.xml 파일이 생성된다.  
3. 다시 서버를 재구동 해야한다.  



***
# 3. 대주제
> 인용
## 3.1. 소 주제
### 3.1.1. 내용1
```
내용1
```
