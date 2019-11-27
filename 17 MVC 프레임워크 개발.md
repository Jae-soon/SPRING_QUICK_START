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
Controller를 구성하는 요소 중에서 Dispatcher
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
