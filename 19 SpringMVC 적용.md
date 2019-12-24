SpringMVC 적용
=======================
우선 ```Spring MVC``` 패턴을 적용하기 위해서 기존 17장에서의 ```Controller``` 관련 파일들을 모두 삭제하자  
기존 Contoller들은 ```Model``` 처리를 한 후 이동할 ```View``` 페이지의 경로를 ```String```으로 반환하여 해당 경로로 이동 시켰다.  

Spring 에서 제공하는 ```Controller```도 이와 비슷하지만 ```String```이 아닌 ```ModelAndView```를 리턴한다.
따라서 우리가 작성한 모든 ```Controller``` 에서 ```handleRequest()``` 메소드의 리턴타입을 ```String```에서 ```ModelAndView```로 변경만 해주면 된다. 


# 1. 로그인 기능 구현
## 1.1. LoginController 구현
로그인 기능은 기존에 작성한 LoginControoler 클래스를 다음과 같이 수정한다.   
   
**LoginController.java**
```
package com.springbook.view.user;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

import org.springframework.web.servlet.ModelAndView;
import org.springframework.web.servlet.mvc.Controller;

import com.springbook.biz.user.UserVO;
import com.springbook.biz.user.impl.UserDAO;

public class LoginController implements Controller {
	@Override
	public ModelAndView handleRequest(HttpServletRequest request, HttpServletResponse response) throws Exception {
		System.out.println("로그인 처리");

		// 1. 사용자 입력 정보 추출
		String id = request.getParameter("id");
		String password = request.getParameter("password");

		// 2. DB 연동 처리
		UserVO vo = new UserVO();
		vo.setId(id);
		vo.setPassword(password);

		UserDAO userDAO = new UserDAO();
		UserVO user = userDAO.getUser(vo);

		// 3. 화면 네비게이션
		ModelAndView mav = new ModelAndView();
		if (user != null) {
			mav.setViewName("getBoardList.do");
		} else {
			mav.setViewName("login.jsp");
		}
		return mav;
	}
}
```
리턴 타입을 ModelAndView로 수정을 하고,  
화면 네비게이션에서 로그인 성공과 실패일 때 실행될 각 화면 정보를 ModelAndView 객체에 저장하여 리턴한다.    
     
## 1.2. HandlerMapping 등록    
작성된 LoginController 가 클라이언트의 ```"/login.do"'''요청에 대해서 동작하게 하려면 스프링 설정파일인     
presentation-layer.xml에 HanlderMapping 과 LoginController를 ```<bean>``` 등록해야 한다.     
   
**presentation-layer.xml**   
```
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xmlns="http://java.sun.com/xml/ns/javaee"
	xsi:schemaLocation="http://java.sun.com/xml/ns/javaee https://java.sun.com/xml/ns/javaee/web-app_2_5.xsd"
	version="2.5">

	<!-- HandlerMapping 등록 -->
	<bean calss="org.springframework.web.servlet.handler.SimpleUrlHandlerMapping">
		<property name="mappings">
			<props>
				<prop key="/login.do">login</prop>
			</props>
		</property>
	</bean>
	
	<!-- Controller 등록 -->
	<bean id="login" class="com.springbook.view.user.LoginController"></bean>


	<filter>
		<filter-name>characterEncoding</filter-name>
		<filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
		<init-param>
			<param-name>encoding</param-name>
			<param-value>UTF-8</param-value>
		</init-param>
	</filter>
	<filter-mapping>
		<filter-name>characterEncoding</filter-name>
		<url-pattern>*.do</url-pattern>
	</filter-mapping>

	<servlet>
		<servlet-name>action</servlet-name>
		<servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
		<init-param>
			<param-name>contextConfigLocation</param-name>
			<param-value>/WEB-INF/config/presentation-layer.xml</param-value>	
		</init-param>
	</servlet>
	<servlet-mapping>
		<servlet-name>action</servlet-name>
		<url-pattern>*.do</url-pattern>
	</servlet-mapping>
</web-app>
```
위 설정에서 SimpleUrlHandlerMapping 객체는 Setter 인젝션을 통해 Properties 타입의 컬렉션 객체를 의존성 주입하고 있다.  
그리고 의존성 주입된 Properties 컬렉션에는 ```/login.do``` 경로 요청에 대해 아이디가 login인 객체가 매핑되어 있다.  
그리고 LoginController 클래스를 ```<bean>```등록하는데, 반드시 SimpleUrlHandlerMapping에서 ```/login.do``` 킷값으로    
매핑한 값과 같은 아이디로 등록해야 한다.  
  
사실 SimpleUrlHandlerMapping의 기능은 우리가 직접 구현한 HandlerMapping과 같다. 
우리가 직접 구현한 HandlerMapping 도 Properties 대신 HashMap 객체를 이용한 것만 제외하면  
스프링의 SimpleUrlHandlerMapping과 같은 기능을 제공한다.  

### 1.2.1. 내용1
```
내용1
```

***
# 2. 대주제
> 인용
## 2.1. 소 주제
### 2.1.1. 내용1
```
내용1
```   

***
# 3. 대주제
> 인용
## 3.1. 소 주제
### 3.1.1. 내용1
```
내용1
```
