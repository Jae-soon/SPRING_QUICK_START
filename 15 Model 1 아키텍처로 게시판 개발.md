15 Model 1 아키텍처로 게시판 개발
=======================
# 1. Model 1 아키텍처 구조
90년대 말부터 2000년대 초까지 자바 기반의 웹 어플리케이션 개발에 사용됐던 아키텍처는 Model 1 이다.   
Model 1 아키텍처는 JSP 와 JavaBeans만 사용하여 웹을 개발하는 것이다.   
![KakaoTalk_20191124_200251571](https://user-images.githubusercontent.com/50267433/69493719-8e930e80-0ef5-11ea-87b3-fa04cd82b098.jpg)  
  
Bean 이라는 용어는 객체를 의미하는 것으로 JavaBean 하면 자바 객체를 의미한다.   
결국, JavaBeans 는 데이터베이스 연동에 사용되는 자바 객체들이다.   
   
원래 Model의 정확한 의미는 데이터베이스 연동 로직을 제공하면서 DB에서 검색한 데이터가 저장되는 자바 객체다.  
우리는 스프링 IoC와 AOP 관련 실습을 진행하면서 VO 와 DAO 클래스를 사용했으며,  
이 VO, DAO 클래스가 바로 Model 기능의 자바 객체다.  

Model 1 아키텍처에서는 JSP 파일이 가장 중요한 역할을 수행하는데,  
이는 JSP 가 Controller 와 View 기능을 모두 처리하기 때문이다.   
   
먼저 Controller 기능은 JSP 파일에 작성된 자바 코드를 의미한다.   
하지만 JSP 파일에 작성된 모든 자바 코드를 Controller 라고 하지 않으며  
일반적으로 Controller 는 사용자의 요청 처리와 관련된 자바 코드를 의미한다.   
  
![KakaoTalk_20191124_201029116](https://user-images.githubusercontent.com/50267433/69493796-8f787000-0ef6-11ea-8b03-6c8392cf0179.jpg)  
  
그리고 JSP에서는 Model을 사용하여 검색한 데이터를 사용자가 원하는 화면으로 제공하기 위해서 다양한 마크업을 사용한다.  
이때 사용되는 대표적인 마크업 언어가 바로 HTML 과 CSS 이며 이 두가지 마크업이 View 기능을 담당한다.  
  
Model 1 구조는 JSP 파일에서 Controller 기능과 View 기능을 모두 처리한다는 특징이 있다.  
결과적으로 JSP 파일에 자바 코드와 마크업 관련 코드들이 뒤섞여 있어서 역할 구분이 명확하지 않고,  
JSP 파일에 대한 디버깅과 유지보수에 많은 어려움이 생길 수 밖에 없다.   
  
따라서 Model 1 구조는 적은 개발인력으로 간단한 프로젝트를 수행하는 때라면 사용할 수 있지만,  
엔터프라이즈급의 복잡한 시스템에는 부적절한 모델이라 할 수 있다.  
   
그래서 등장한 것이 Model 2, 즉 MVC 아키텍처다.  
Model 2 를 일반적으로 MVC 라고 부르는데, 이는 Model, View, Controller 요소로 기능을 분리하여 개발하기 때문이다.  
결국, MVC는 Model 1 구조의 단점을 보완하기 위해 만들어진 구조라고 생각할 수 있다.  
  
우리가 최종적으로 적용할 것은 Model 2, 즉 MVC 아키텍처이지만 이번 시간에는 Model 1 아키텍처로 게시판 프로그램을 개발할 것이다.   
이번 실습을 통해서 JSP 파일 작성에 필요한 기본 문법도 확인하고, 게시판 프로그램의 전반적인 기능도 이해하기 바란다.    
그리고 게시판 프로그램에 사용할 화면은 HTML 태그만 사용하여 최대한 단순하게 처리할 것이다.  
  
이제 게시판 프로그램에서 가장 기본이 되는 로그인 기능부터 BOARD 테이블과 관련되 CRUD 기능을 차례로 구현해보도록 한다.    
   
***
# 2. 로그인 기능 구현
## 2.1. 로그인 화면
게시판 사용자는 로그인을 성공해야 게시판 목록 화면을 볼 수 있으므로 가장 먼저 로그인 기능을 개발한다.    
우선 사용자에게 로그인 화면을 제공하기 위해서 login.jsp 파일을 만든다.  
  
앞으로 작성하는 모든 JSP 파일은 ```src/main``` 폴더에 있는 webapp 폴더에 등록해야 한다.   
따라서 이클립스의 프로젝트 탐색 창에서 ```src/main/webapp``` 폴더를 선택하고,   
마우스 오른쪽 버튼을 클릭하여 ```[New]->[JSP File]```을 선택한다.   

JSP 파일 생성 대화상자가 열리면 'File name'에 'login' 또는 'login.jsp'를 입력하고 ```<Finish>``` 버튼을 클릭한다.       
   
방금 생성한 login.jsp 파일에 HTML 태그들을 이용하여 로그인 화면을 구성한다.   

**login.jsp**
```
<%@ page language="java" contentType="text/html; charset=UTF-8"
    pageEncoding="UTF-8"%>
<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8">
<title>로그인</title>
</head>
<body>
<center>
<h1>로그인</h1>
<hr>
<form action="login_proc.jsp" method="post">
<table border="1" cellpadding="0" cellspacing="0">
	<tr>
		<td bgcolor="orange">아이디</td>
		<td><input type="text" name="id"></td>
	</tr>
	<tr>
		<td bgcolor="orange">비밀번호</td>
		<td><input type="password" name="password"></td>
	</tr>
	<tr>
		<td colspan="2" align="center">
			<input type="submit" value="로그인" />
		</td>
	</tr>	
</table>
</form>
</center>
</body>
</html>
```
우선 여태까지 예제를 그대로 따라서 했다면      
애플리케이션의 루트 디렉토리가 기존 다른 책들과는 다를 것이다.     
예를 들면 ```http://localhost:8080/biz/login.jsp```로 되어있을 것이다.     
이러한 url을 바꾸려면 실행중인 서버를 더블 클릭하고 아래에 새로 생기는 Modules 탭을 클릭해    
기존 맵핑을 edit 해서 프로젝트 명으로 바꿔줄 수도 있다.    

## 2.2. 로그인 인증 처리   
이제 사용자가 입력한 아이디 비밀번호를 추출하여 로그인을 처리하는 login_proc.jsp 파일을 작성한다.   
   
**login_proc.jsp**
```
<%@page import="com.springbook.biz.user.impl.UserDAO"   %>
<%@page import="com.springbook.biz.user.UserVO"%>
<%@page contentType="text/html; charset=UTF-8" %>
<%
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
		response.sendRedirect("getBoardList.jsp");
	} else {
		response.sendRedirect("login.jsp");
	}
%>
```   
우선 사용자가 입력한 아이디와 비밀번호를 request 객체로부터 추출한다.   
그리고 Model에 해당하는 UesrVO 와 UserDAO 객체를 이용하여 사용자 정보를 검색한다.   
그리고 검색 결과로 UserVO 객체가 리턴되면 로그인이 성공이고, null이 리턴되면 로그인 실패로 처리한다.  
   
USERS 테이블에 등록된 계정으로 로그인에 성공하면 글목록 화면으로 이동하고,  
실패하면 다시 로그인하도록 로그인 화면으로 이동한다.   
화면 내비게이션 방법에는 포워드 방식과 리다이렉트 두 가지 방법이 있지만,  
당분간은 로직의 단순화를 위해서 리다이렉트 방식만을 사용해서 개발한다.  
    
![KakaoTalk_20191124_220330240](https://user-images.githubusercontent.com/50267433/69495089-724b9d80-0f06-11ea-8990-f8919235ad20.jpg)   
           
***
# 3. 글 목록 검색 기능 구현
로그인에 성공한 다음에는 글 목록 화면으로 이동한다.  
BOARD 테이블에서 게시글을 검색하여 글 목록 화면을 구성하는 getBoardList.jsp 파일을 작성한다.   

**getBoardList.jsp**
```
<%@page import="java.util.List"%>
<%@page import="com.springbook.biz.board.impl.BoardDAO"%>
<%@page import="com.springbook.biz.BoardVO"%>
<%@ page language="java" contentType="text/html; charset=UTF-8"
	pageEncoding="UTF-8"%>
<%
	// 1. 사용자 입력 정보 추출(검색 기능은 나중에 구현)
	// 2. DB 연동처리
	BoardVO vo = new BoardVO();
	BoardDAO boardDAO = new BoardDAO();
	List<BoardVO> boardList = boardDAO.getBoardList(vo);

	// 3. 응답 화면 구성
%>


<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8">
<title>글 목록</title>
</head>
<body>
	<center>
		<h1>글 목록</h1>
		<h3>
			테스트님 환영합니다...<a href="logout_proc.jsp">Log-out</a>
		</h3>

		<!-- 검색 시작 -->
		<form action="getBoardList.jsp" method="post">
			<table border="1" cellpadding="0" cellspacing="0" width="700">
				<tr>
					<td align="right"><select name="searchCondition">
							<option value="TITLE">제목
							<option value="CONTENT">내용
					</select> <input name="searchKeyword" type="text" /> <input type="submit"
						value="검색" /></td>
				</tr>
			</table>
		</form>

		<!-- 검색 종료 -->

		<table border="1" cellpadding="0" cellspacing="0" width="700">
			<tr>
				<th bgcolor="orange" width="100">번호</th>
				<th bgcolor="orange" width="200">제목</th>
				<th bgcolor="orange" width="150">작성자</th>
				<th bgcolor="orange" width="150">등록일</th>
				<th bgcolor="orange" width="100">조회수</th>
			</tr>

			<%
				for(BoardVO board : boardList){
			%>
			<tr>
				<td><%= board.getSeq() %></td>
				<td align="left">
					<a href="getBoard.jsp?seq=<%= board.getSeq() %>"> <%=board.getTitle() %></a>
				</td>
				<td><%= board.getWriter() %></td>
				<td><%= board.getRegDate() %></td>
				<td><%= board.getCnt() %></td>
			</tr>
			<%		
				}
			%>
		</table>
		<br> <a href="insertBoard.jsp">새글 등록</a>
	</center>
</body>
</html>
```
getBoardList.jsp 파일에서는 사용자가 입력한 검색 관련 정보를 추출해야 하는데,  
검색 기능은 나중에 추가로 구현하도록 한다.   
따라서 곧바로 BoardVO 와 BoardDAO 객체를 이용하여 BOARD 테이블에 저장된 게시글 목록을 검색한다.    
그리고 검색 결과로 얻은 ```List<BoardVO>``` 객체를 이용하여 게시글 목록 화면을 구성한다.   
   
위 소스에서 주목할 부분은 게시글 제목에 하이퍼링크를 설정하는 코드이다.  
사용자가 게시글 제목을 클릭했을 때, 해당 게시글의 상세 정보를 조회하여 출력하기 위해서 getBoard.jsp 파일로 링크를 연결했다.   
이때, 사용자가 클릭한 게시글 번호를 넘겨주고자 getBoard.jsp 파일 뒤에 "?"를 추가하고 쿼리 문자열 정보를 넘겨주었다.  
```
<td align="left">
	<a href="getBoard.jsp?seq=<%= board.getSeq() %>"> <%=board.getTitle() %></a>
</td>
```
이렇게 작성된 JSP 파일을 실행하면 다음과 같은 글 목록 화면이 출력된다.  
이제 로그인 화면에서 로그인에 실패하면 로그인 화면으로 되돌아가고,  
성공하면 글 목록 화면으로 이동한다.   
## 3.1. 소 주제
### 3.1.1. 내용1
```
내용1
```
