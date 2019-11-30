17 MVC 프레임워크 개발
=======================
# 1. MVC 프레임워크 구조  
지금까지 개발한 게시판 프로그램은 MVC 아키텍처를 적용하긴 했지만,  
DispatcherServlet 클래스 하나로 Controller 기능을 구현했다.  
하지만 이렇게 하나의 서블릿으로 Controller를 구현하면 클라이언트의 모든 요청을 하나의 서블릿이 처리하게 된다.  
따라서 수많은 분기 처리 로직을 가질 수밖에 없고, 이는 오히려 개발과 유지보수를 어렵게 만든다.   

**DispatcherServlet**
```
package com.springbook.view.controller;

import java.io.IOException;
import java.util.List;

import javax.servlet.ServletException;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import javax.servlet.http.HttpSession;

import com.springbook.biz.BoardVO;
import com.springbook.biz.board.impl.BoardDAO;
import com.springbook.biz.user.UserVO;
import com.springbook.biz.user.impl.UserDAO;

public class DispatcherServlet extends HttpServlet {
	private static final long serialVersionUID = 1L;

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
			if(user != null){
				response.sendRedirect("getBoardList.do");
			} else {
				response.sendRedirect("login.jsp");
			}
			
		} else if (path.equals("/logout.do")) {
			System.out.println("로그아웃 처리");
			// 1. 브라우저와 연결된 세션 객체를 강제 종료한다.  
			HttpSession session = request.getSession();
			session.invalidate();

			// 2. 세션 종료 후, 메인 화면으로 이동한다.   
			response.sendRedirect("login.jsp");
		} else if (path.equals("/insertBoard.do")) {
			System.out.println("글 등록 처리");
			
			// request.setCharacterEncoding("UTF-8");
			String title = request.getParameter("title");
			String writer = request.getParameter("writer");
			String content = request.getParameter("content");
			
			// 2. DB 연동 처리
			BoardVO vo = new BoardVO();
			vo.setTitle(title);
			vo.setWriter(writer);
			vo.setContent(content);
			
			BoardDAO boardDAO = new BoardDAO();
			boardDAO.insertBoard(vo);
			
			// 3. 화면 네비게이션
			response.sendRedirect("getBoardList.do");
			
		} else if (path.equals("/updateBoard.do")) {
			System.out.println("글 수정 처리");
			// 1. 사용자 입력 정보 추출 
			
			request.setCharacterEncoding("UTF-8");
			String title = request.getParameter("title");
			String content = request.getParameter("content");
			String seq = request.getParameter("seq");
			
			
			// 2. DB 연동 처리
			BoardVO vo = new BoardVO();
			vo.setTitle(title);
			vo.setContent(content);
			vo.setSeq(Integer.parseInt(seq));
			
			BoardDAO boardDAO = new BoardDAO();
			boardDAO.updateBoard(vo);
			
			// 3. 화면 네비게이션
			response.sendRedirect("getBoardList.do");
		} else if (path.equals("/deleteBoard.do")) {
			System.out.println("글 삭제 처리");
			// 1. 사용자 입력 정보 추출 
			
			request.setCharacterEncoding("UTF-8");
			String seq = request.getParameter("seq");
			
			
			// 2. DB 연동 처리
			BoardVO vo = new BoardVO();
			vo.setSeq(Integer.parseInt(seq));
			
			BoardDAO boardDAO = new BoardDAO();
			boardDAO.deleteBoard(vo);
			
			// 3. 화면 네비게이션
			response.sendRedirect("getBoardList.do");
		} else if (path.equals("/getBoard.do")) {
			System.out.println("글 상세 조회 처리");
			
			// 1. 검색할 게시글 번호 추출
			String seq = request.getParameter("seq");

			// 2. DB 연동 처리
			BoardVO vo = new BoardVO();
			vo.setSeq(Integer.parseInt(seq));

			BoardDAO boardDAO = new BoardDAO();
			BoardVO board = boardDAO.getBoard(vo);
			
			// 3. 응답 화면 구성
			HttpSession session = request.getSession();
			session.setAttribute("board", board);
			response.sendRedirect("getBoard.jsp");
			
		} else if (path.equals("/getBoardList.do")) {
			System.out.println("글 목록 검색 처리");
			// 1. 사용자 입력 정보 추출(검색 기능은 나중에 구현)
			// 2. DB 연동처리
			BoardVO vo = new BoardVO();
			BoardDAO boardDAO = new BoardDAO();
			List<BoardVO> boardList = boardDAO.getBoardList(vo);
			
			// 3. 검색 결과를 세션에 저장하고 목록 화면으로 이동한다.  
			HttpSession session = request.getSession();
			session.setAttribute("boardList", boardList);
			response.sendRedirect("getBoardList.jsp");
		}
	}
}
```
DispatcherServlet 클래스가 이렇게 복잡하게 구현되어 있으면 특정 기능을 수정하려고 할 때  
코드를 찾는 것부터가 쉽지 않고, 새로운 기능을 추가할 때마다 분기 처리 로직은 계속 늘어나므로 복잡도는 계속 증가할 것이다.  
   
결국, Controller를 서블릿 클래스 하나로 구현하는 것은 여러 측면에서 문제가 있으며,      
다양한 디자인 패턴을 결합하여 개발과 유지보수의 편의성이 보장되도록 잘 만들어야 한다.   
하지만 프레임워크에서 제공하는 Controller를 사용하면 우리가 직접 Controller를 구현하지 않아도 된다.  
우리가 Struts나 Spring(MVC)같은 MVC프레임워크를 사용하는 이유는 바로 이런 프레임워크들이 효율적인 Controller를 제공해주기 때문이다.   

우리가 최종적으로 적용할 Spring MVC는 DispatcherServlet을 시작으로 다양한 객체들이    
상호작용하면서 클라이언트의 요청을 처리한다.  
하지만 MVC 프레임워크에서 제공하는 Controller는 기능과 구조가 복잡해서 바로 적용하기가 어렵다.     
   
따라서 본격적으로 Spring MVC를 적용하기 전에 Spring MVC와 동일한 구조의  
프레임워크를 직접 구현하여 적용하려고 한다.  
이런 단계를 거침으로써 Spring MVC 의 구성 요소와 동작 원리를 더욱 쉽게 이해할 수 있을 것이다.   
   
[사진]   
   
이제 MVC 프레임워크에서 Controller를 구성하는 각 요소를 직접 구현해봄으로써  
스프링에서 제공하는 클래스들의 기능도 유추할 수 있다.  
이제 로그인 기능을 구현하면서 각 구성 요소들을 만들어보자   
    
***
# 2. MVC 프레임워크 구현
## 2.1. Controller 인터페이스 작성
Controller를 구성하는 요소 중에서 DispatcherServlet은 클라이언트의 요청을 가장 먼저 받아들이는 Front Controller 이다.  
하지만 클라이언트의 요청을 처리하기 위해 DispatcherServlet이 하는 일은 거의 없으며,    
실질적인 요청 처리는 각 Controller에서 담당한다.  
  
구체적인 Controller 클래스들을 구현하기에 앞서 모든 Controller를 같은 타입으로 관리하기 위한 인터페이스를 만들어야한다.  
클라이언트의 요청을 받은 DispatcherServlet은 HandlerMapping을 통해 Controller 객체를 검색하고, 검색된 Controller를 실행한다.   
이때 어떤 Controller 객체가 검색되더라도 같은 코드로 실행하려면 모든 Controller의 최상위 인터페이스가 필요하다.  
   
**Controller**
```
package com.springbook.view.controller;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

public interface Controller {
	String HandlerRequset(HttpServletRequest request, HttpServletResponse response);
}
```
## 2.2. LoginController 
Controller 인터페이스를 구현한 LoginController 클래스를 만들고
이를 구현하는 Controller 클래스를 작성해주면 된다.   
이때 DispatcherServlet 클래스에서 ```/login.do```에 해당하는 소스를 복사하여 구현해보자   

**LoginController**
```
package com.springbook.view.controller;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

import com.springbook.biz.user.UserVO;
import com.springbook.biz.user.impl.UserDAO;

public class LoginController implements Controller {
	@Override
	public String HandlerRequset(HttpServletRequest request, HttpServletResponse response) {
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
		if (user != null) {
			return "getBoardList.do";
		} else {
			return "login";
		}		
	}
}
```   
로그인 처리 소스는 DispatcherServlet의 로그인 처리 기능과 같다.     
다만, Controller 인터페이스의 handleRequest()메소드를 재정의 했으므로    
로그인 처리 기능의 마지막은 이동할 화면을 리다이렉트 하지 않고 리턴하는 것으로 처리한다.    
  
그런데 로그인에 실패했을 때 이동할 화면 정보가 login,jsp가 아니라 그냥 login이다.      
이는 나중에 추가할 ViewResolve 클래스를 만들어야 정확하게 이해할 수 있으니     
여기서는 단지 handleRequest() 메소드가 확장자 없는 문자열을 리턴하면,      
자동으로 ```.jsp```확장자가 붙어서 처리된다는 것만 이해하자    

## 2.3. HandlerMapping 클래스 작성 
HandlerMapping은 모든 Controller 객체들을 저장하고 있다가,  
클라이언트의 요청이 들어오면 요청을 처리할 특정 Controller를 검색하는 기능을 제공한다.  
    
HandlerMapping 객체는 DispatcherSerlvet이 사용하는 객체이다.    
따라서 DisaptcherServlet이 생성되고 init()메소드가 호출될 때 단 한번 생성된다.  

**HandlerMapping**
```
package com.springbook.view.controller;

import java.util.HashMap;
import java.util.Map;

public class HandlerMapping {
	private Map<String, Controller> mappngs;
	
	public HandlerMapping() {
		mappngs = new HashMap<String, Controller>();
		mappngs.put("/login.do", new LoginController());
	}
	
	public Controller getController(String path) {
		return mappngs.get(path);
	}

}
```
HandlerMapping은 Map 타입의 컬렉션을 맴버변수로 가지고 있으면서   
게시판 프로그램에 필요한 모든 Controller 객체들을 등록하고 관리한다.    
   
getController() 메소드는 매개변수로 받은 path에 해당하는 Controller 객체를 HashMap 컬렉션으로부터 검색하여 리턴한다.   
지금은 LoginController 객체 하나만 등록되어 있지만, 앞으로 계속 Controller 객체들이 추가될 것이다.  
따라서 HashMap에 등록된 정보를 보면 Controller 객체가 어떤 ```.do```요청과 매핑되어있는지 확인할 수 있다.  

## 2.4. ViewResolver 클래스 작성 
ViewResolver 클래스는 Controller가 리턴한 View 이름에 접두사와 접미사를 결합하여 최종으로 실행될 View 경로와 파일명을 완성한다.     
ViewResolver도 HandlerMapping과 마찬가지로 DispatcherServlet의 init()메소드가 호출될 때 생성된다.      
   
**ViewResolve**
```
package com.springbook.view.controller;

public class ViewResolve {

	public String prefix;
	public String suffix;

	public void setPrefix(String prefix) {
		this.prefix = prefix;
	}

	public void setSuffix(String suffix) {
		this.suffix = suffix;
	}

	public String getView(String viewName) {
		return prefix + viewName + suffix;
	}

}
```
ViewResovle는 setPrefix()와 setSuffix() 메소드로 접두사(prefix)와 접미사(suffix)를 초기화한다.  
그리고 getView() 메소드가 호출될 때 viewName 앞에 prefix를 결합하고  
viewName 뒤에 suffix를 결합하여 리턴한다.  
  
## 2.5. DispatcherServlet 수정   
DispatcherSerlvet은 FrontController 기능의 클래스로서 Controller 구성 요소 중 가장 중요한 역할을 수행한다.  
지금부터 DispatcherServlet 클래스를 수정해야 하는데,  
수정하기 전에 바탕화면이나 다른곳에 복사본을 만들어 놓아야한다.  
그래야 나중에 구체적인 Controller 클래스 구현에서 소스를 재사용할 수 있다.   

**DispatcherSerlvet**
```
package com.springbook.view.controller;

import java.io.IOException;
import java.util.List;

import javax.servlet.ServletException;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import javax.servlet.http.HttpSession;

import com.springbook.biz.BoardVO;
import com.springbook.biz.board.impl.BoardDAO;
import com.springbook.biz.user.UserVO;
import com.springbook.biz.user.impl.UserDAO;

public class DispatcherServlet extends HttpServlet {
	private static final long serialVersionUID = 1L;
	private HandlerMapping handlermapping;
	private ViewResolve viewResolve;

	@Override
	public void init() throws ServletException {
		handlermapping = new HandlerMapping();
		viewResolve = new ViewResolve();
		viewResolve.setPrefix("./");
		viewResolve.setPrefix(".jsp");
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
		String uri = request.getRequestURI(); // request가 사용한 uri 얻기 여기서는 do동작시 가져오기에 제각기이다.
		String path = uri.substring(uri.lastIndexOf("/")); // 시작위치가 /부터 된다. 즉 /포함 이후 글자 얻어내기

		// 2. HandlerMapping을 통해 path에 해당하는 Controller를 검색한다.
		Controller ctrl = handlermapping.getController(path); // 그럼 /login.do가 가서 mapping된 객체 가져온다.

		// 3. 검색된 Controller를 실행한다.
		String viewName = ctrl.HandlerRequset(request, response);

		// 4.
		String view = null;
		if (!viewName.contains(".do")) {
			view = viewResolve.getView(viewName);
		} else {
			view = viewName;
		}

		// 5. 검색된 화면으로 이동한다.
		response.sendRedirect(view);
	}
}
```



***
# 3. 대주제
> 인용
## 3.1. 소 주제
### 3.1.1. 내용1
```
내용1
```
