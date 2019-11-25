16 Model 2 아키텍처로 게시판 개발
=======================
# 1. Model 2 아키텍처 구조
시스템의 규모가 크고 기능이 복잡한 엔터프라이즈 시스템을 개발한다면 Model 1 아키텍처는 적합하지 않다.   
Model 1 아키텍처가 엔터프라이즈 시스템에 적합하지 않은 이유는  
자바 로직과 디자인이 통합되어 유지보수가 어렵기 때문이다.   
디자인이 통합되어 있으면, 자바 로직을 찾기도 힘들고 디자인을 변경할 때도 복잡한 자바 코드들로 인해 어려움을 느낀다.  
   
이런 Model 1 아키텍처의 문제를 해결하기 위해 고안된 웹 개발 모델이 Model 2 아키텍처,   
즉 MVC 아키텍처이고 가장 중요한 특징은 Controller의 등장이며, 이 Controller는 서블릿 클래스를 중심으로 구현된다.   
   
[사진]   
  
Model 2 아키텍처에서는 기존에 JSP가 담당했던 Controller 로직이 별도의 Controller 기능의 서블릿으로 옮겨졌다.     
따라서 기존에 Model 1 아키텍처로 개발한 프로그램에서 JSP 파일에 있는 자바 코드만 Controller로 이동하면     
Model 2 아키텍처가 된다. 결과적으로 Controller 로직이 사라진 JSP에는 View와 관련된 디자인만 남게 되어  
디자이너는 JSP 파일을 관리하고, 자바 개발자는 Controller 와 Model 만 관리하면 된다.  

Model 기능은 VO, DAO 클래스로 자바 개발자가 구현하고 관리한다.  
그리고 View 기능은 디자이너가 JSP 파일로 구현하고 관리한다.  
사실 MVC 아키텍처에서 가장 중요한 부분이 바로 Controller인데,  
이 Controller를 성능과 유지보수의 편의성을 고려하여 잘 만드는 것이 무엇보다 중요하다.  

Controller는 자바 개발자들이 직접 구현할 수도 있지만, MVC 프레임워크가 제공하는 Controller를 사용해도 된다.     
우리가 직접 구현하는 것보다는 MVC 프레임워크에서 제공하는 잘 만들어진 Controller를 사용하는 것이 더 효율적이고 안정적이다.  
문제는 프레임워크가 제공하는 Controller 구조가 너무 복잡하고 어렵다는 점이다.    
그러니 우선 우리는 Controller의 기능을 중점으로 두는 것으로 공부를 진행하자.   
   
***
# 2. Controller 구현하기
## 2.1. 서블릿 생성 및 등록
Controller 기능을 수행하는 서블릿 클래스를 하나 추가하여  
기존의 Model 1 기반으로 개발된 게시판 프로그램을 MVC 아키텍처로 변경하자.  
  
Controller에 해당하는 서블릿 클래스를 구현할 때 이클립스의 기능을 이용하면 좀 더 쉽게 작성할 수 있다.  

1. ```src/main/java``` 에서 오른쪽 버튼으로 ```[New]->[Servlet]```을 선택한다.     
2. 'java package'에 'com.springbook.view.controller'를, 'Class name'에 'DispatcherServlet'을 입력하고 ```<Next>```클릭    
3. 'Name'에 'action'으로 입력하고, 'URL mappings'에 '/action'을 더블 클릭하여 Pattern을 '*.do'로 설정한고 ```<Finish>```클릭  
DispatcherServlet 클래스가 만들어지는 순간 web.xml 파일에 서블릿 관련 설정이 자동으로 추가된다.  

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
		<description></description>
		<display-name>action</display-name>
		<servlet-name>action</servlet-name>
		<servlet-class>com.springbook.view.controller.DispatcherServlet</servlet-class>
	</servlet>
	<servlet-mapping>
		<servlet-name>action</servlet-name>
		<url-pattern>*.do</url-pattern>
	</servlet-mapping>
</web-app>
```
위 설정은 클라이언트의 모든 ```*.do```요청을 DispatcherServlet 클래스의 객체가 처리한다는 설정이다.    
확장자 '.do'는 얼마든지 다른 이름으로 변경할 수 있다.  

## 2.2. Controller 서블릿 구현  
서블릿 클래스가 만들어질 때 자동으로 추가되는 주석들은 모두 제거하고  
DispatcherServlet 클래스가 Controller 기능을 수행하도록 구현한다.   

**DispatcherServlet**
```
package com.springbook.view.controller;

import java.io.IOException;
import javax.servlet.ServletException;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

public class DispatcherServlet extends HttpServlet {
	private static final long serialVersionUID = 1L;

	public DispatcherServlet() {
		super();
	}

	protected void doGet(HttpServletRequest request, HttpServletResponse response)
			throws ServletException, IOException {
		process(request, response);
	}

	protected void doPost(HttpServletRequest request, HttpServletResponse response)
			throws ServletException, IOException {
		request.setCharacterEncoding("UTF-8");
		process(request, response);
	}

	private void process(HttpServletRequest request, HttpServletResponse response) throws IOException {
		// 1. 클라이언트의 요청 path 정보를 추출한다.
		String uri = request.getRequestURI();
		String path = uri.substring(uri.lastIndexOf("/"));
		System.out.println(path);

		// 2. 클라이언트의 요청 path에 따라 적절히 분기처리 한다.
		if (path.equals("/login.do")) {
			System.out.println("로그인 처리");
		} else if (path.equals("/logout.do")) {
			System.out.println("로그아웃 처리");
		} else if (path.equals("/insertBoard.do")) {
			System.out.println("글 등록 처리");
		} else if (path.equals("/updateBoard.do")) {
			System.out.println("글 수정 처리");
		} else if (path.equals("/deleteBoard.do")) {
			System.out.println("글 삭제 처리");
		} else if (path.equals("/getBoard.do")) {
			System.out.println("글 상세 조회 처리");
		} else if (path.equals("/getBoardList.do")) {
			System.out.println("글 목록 검색 처리");
		}
	}
}
```
DispatcherServlet에는 GET 방식 요청을 처리하는 doGet() 메소드와    
POST 방식 요청을 처리하는 doPost() 메소드가 재정의되어 있다.    
하지만 어떤 방식으로 요청하든 process() 메소드를 통해 클라이언트의 요청을 처리한다.  

POST 방식의 요청에 대해서 doPost() 메소드가 호출되는데, 이때 한글이 깨지지 않도록 인코딩 처리를 추가한다.   
이제 글 등록, 글 수정 작업에서 한글 데이터가 깨지는 일은 발생하지 않는다.   
  
그리고 무엇보다 인코딩 작업을 DispatcherServlet 클래스에서 일괄 처리하므로 인코딩을 변경할 때는  
DispatcherServlet 클래스만 수정하면 된다.  

DispatcherServlet에서 가장 중요한 process() 메소드에서는 가장 먼저 클라이언트의 요청 URI로부터 path 정보를 추출하고 있는데,  
이때 추출된 path는 URI 문자열에서 마지막 ```/xxx.do```문자열이다.   
그리고 추출된 path 문자열에 따라서 복잡한 분기 처리 로직이 실행된다.   
   
이제 작성된 DispatcherServlet을 저장하고 서버를 재구동한 다음,  
브라우저의 URL 검색 창에 다음의 URL을 차례로 요청해보자 .  
```
http://localhost:8080/BoardWeb/login.do
http://localhost:8080/BoardWeb/logout.do
http://localhost:8080/BoardWeb/insertBoard.do
http://localhost:8080/BoardWeb/updateBoard.do
http://localhost:8080/BoardWeb/deleteBoard.do
http://localhost:8080/BoardWeb/getBoard.do
http://localhost:8080/BoardWeb/getBoardList.do
```


***
# 3. 대주제
> 인용
## 3.1. 소 주제
### 3.1.1. 내용1
```
내용1
```
 
