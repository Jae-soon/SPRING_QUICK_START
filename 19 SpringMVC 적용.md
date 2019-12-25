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
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

<!-- HandlerMapping 등록 -->
	<bean class="org.springframework.web.servlet.handler.SimpleUrlHandlerMapping">
		<property name="mappings">
			<props>
				<prop key="/login.do">login</prop>
			</props>
		</property>
	</bean>
	
	<!-- Controller 등록 -->
	<bean id="login" class="com.springbook.view.user.LoginController"></bean>
</beans>	
```
위 설정에서 SimpleUrlHandlerMapping 객체는 Setter 인젝션을 통해 Properties 타입의 컬렉션 객체를 의존성 주입하고 있다.  
그리고 의존성 주입된 Properties 컬렉션에는 ```/login.do``` 경로 요청에 대해 아이디가 login인 객체가 매핑되어 있다.  
그리고 LoginController 클래스를 ```<bean>```등록하는데, 반드시 SimpleUrlHandlerMapping에서 ```/login.do``` 킷값으로    
매핑한 값과 같은 아이디로 등록해야 한다.  
  
사실 SimpleUrlHandlerMapping의 기능은 우리가 직접 구현한 HandlerMapping과 같다. 
우리가 직접 구현한 HandlerMapping 도 Properties 대신 HashMap 객체를 이용한 것만 제외하면  
스프링의 SimpleUrlHandlerMapping과 같은 기능을 제공한다.   

***
# 3. 글 목록 검색 기능 구현
## 3.1. GetBoardListController 구현
글 목록을 출력하기 위해서 기존에 사용하던 GetBoardListController 클래스를 수정한다.
**GetBoardListController**
```
package com.springbook.view.board;

import java.util.List;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
// import javax.servlet.http.HttpSession;

import org.springframework.web.servlet.ModelAndView;
import org.springframework.web.servlet.mvc.Controller;

import com.springbook.biz.BoardVO;
import com.springbook.biz.board.impl.BoardDAO;

public class GetBoardListController implements Controller {

	@Override
	public ModelAndView handleRequest(HttpServletRequest request, HttpServletResponse response) throws Exception {
		System.out.println("글 목록 검색 처리");
		
		// 1. 사용자 입력 정보 추출(검색 기능은 나중에 구현)
		// 2. DB 연동처리
		BoardVO vo = new BoardVO();
		BoardDAO boardDAO = new BoardDAO();
		List<BoardVO> boardList = boardDAO.getBoardList(vo);
		
		// 3. 검색 결과를 세션에 저장하고 목록 화면으로 이동한다.  
		ModelAndView mav = new ModelAndView();
		mav.addObject("boardList", boardList); //모델 정보 입력
		mav.setViewName("getBoardList.jsp"); // 뷰 정보 입력
		
		return mav;
	/*
		HttpSession session = request.getSession();
		session.setAttribute("boardList", boardList);
		return "getBoardList.jsp";
	*/	
	}
}
```
여기서 중요한 것은 검색 결과를 세션이 아닌 ModelAndView 객체에 저장하고 있다는 점이다.     
세션이라는 것은 클라이언트 브라우저 하나당 하나씩 서버 메모리에 생성되어      
클라이언트의 상태정보를 유지하기 위해서 사용한다.   
  
따라서 세션에 많은 정보가 저장되면 될수록 서버에 부담을 줄 수 밖에 없다.  
   
결국, 검색 결과는 세션이 아닌 HttpServletRequest 객체에 저장하는 것이 맞다.  
HttpServletRequest는 클라이언트의 요청으로 생성됐다가 응답 메시지가 클라이언트로 전송되면 자동으로 삭제되는 일회성 객체이다.  
따라서 검색 겨로가를 세션이 아닌 HttpServletRequest 객체에 저장하여 공유하면 서버에 부담을 주지 않고도 데이터를 공유할 수 있다.  

**하지만**  
GetBoardListController는 검색 결과를 HttpSession도 아니고 HttpServletRequest도 아닌 ModelAndView에 저장하고 있다.    
ModelAndView 는 클래스 이름에서 알 수 있듯이 Model과 View 정보를 모두 저장하여 리턴할 때 사용한다.  
   
DispatcherServlet은 Controller가 리턴한 ModelAndView 객체에서 Model 정보를 추출한 다음   
HttpServletRequest 객체에 검색 결과에 해당하는 Model 정보를 저장하여 JSP로 포워딩한다.  
따라서 JSP 파일에서는 검색 결과를 세션이 아닌 HttpServletRequest로 부터 꺼내 쓸 수 있다.  
   
## 3.2. HandlerMapping 등록  
이제 GetBoardListController 객체가 ```/getBoardList.do```요청에 동작할 수 있도록,   
SimpleUrlHandlerMapping에 매핑 정보를 추가하면 된다.  

**presentation-layer.xml**
```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

<!-- HandlerMapping 등록 -->
	<bean class="org.springframework.web.servlet.handler.SimpleUrlHandlerMapping">
		<property name="mappings">
			<props>
				<prop key="/login.do">login</prop>
				<prop key="/getBoardList.do">getBoardList</prop>
			</props>
		</property>
	</bean>
	
	<!-- Controller 등록 -->
	<bean id="login" class="com.springbook.view.user.LoginController"></bean>
	<bean id="getBoardList" class="com.springbook.view.board.GetBoardListController"></bean>

</beans>
```
