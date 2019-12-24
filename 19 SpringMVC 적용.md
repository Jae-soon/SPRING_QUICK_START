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

```

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
