27 Mybatis 프레임워크 시작하기
=======================
Mybatis는 원래 Apache에서 Ibatis라는 이름의 프레임워크로 탄생하였으나,     
2010년에 Ibatis가 Apache에서 탈퇴하여 Google로 넘어가면서 이름이 Mybatis로 변경되었다.      
Mybatis는 Ibatis로 부터 파생되어서 기본 개념과 문법은 Ibatis와 거의 유사하다.    

# 1. Mybatis 프레임워크 특징  
Mybatis 프레임워크의 가장 중요한 특징을 2가지로 정리하자면      
    
1. 한두줄의 자바 코드로 DB 연동을 처리한다는 것이며   
2. 둘째는 SQL 명령어를 자바 코드에서 분리하여 XML 파일에 따로 관리하는 것이다.      
    
이 두가지가 기존에 우리가 사용하던 JDBC 기반의 DB연동을 어떻게 개선하는지 살펴보자.       
  
**BoardDAO**
```
package com.springbook.biz.board.impl;

import java.sql.Connection;
import java.sql.PreparedStatement;
import java.sql.ResultSet;
import java.util.ArrayList;
import java.util.List;

import org.springframework.stereotype.Repository;

import com.springbook.biz.BoardVO;
import com.springbook.biz.common.JDBCUtil;

@Repository("boardDAO")
public class BoardDAO  {
	// JDBC 관련 변수
	private Connection conn = null;
	private PreparedStatement pstmt = null;
	private ResultSet rs = null;

	private final String BOARD_LIST = "SELECT * FROM board ORDER BY seq DESC;";
	public List<BoardVO> getBoardList(BoardVO vo) {
		System.out.println("===> JDBC로 getBoardList()기능 처리");
		List<BoardVO> boardList = new ArrayList<BoardVO>();
		try {
			conn = JDBCUtil.getConnection();
			if(vo.getSearchCondition().equals("TITLE")) {
				pstmt = conn.prepareStatement(BOARD_LIST_T);
			} else if(vo.getSearchCondition().equals("CONTENT")) {
				pstmt = conn.prepareStatement(BOARD_LIST_C);
			}
			pstmt.setString(1, vo.getSearchKeyword());
			rs = pstmt.executeQuery();
			while (rs.next()) {
				BoardVO board = new BoardVO();
				board.setSeq(rs.getInt("SEQ"));
				board.setTitle(rs.getString("TITLE"));
				board.setWriter(rs.getString("WRITER"));
				board.setContent(rs.getString("CONTENT"));
				board.setRegDate(rs.getDate("REGDATE"));
				board.setCnt(rs.getInt("CNT"));
				boardList.add(board);
			}
		} catch (Exception e) {
			e.printStackTrace();
		} finally {
			JDBCUtil.close(rs, pstmt, conn);
		}
		return boardList;
	}
}
```
위 코드는 JDBCUtil 클래스를 이용하여 DB 커넥션 획득과 해제 작업을 처리해서 그나마 코드가 간결해졌지만  
JDBCUtil 클래스가 없었다면 더 많은 자바 코드가 필요했을 것이다.   
  
사실 유지보수 관점에서 보면 DB 연동에 사용된 복잡한 자바 코드는 더 이상 중요하지 않다.    
개발자는 실행되는 SQL만 관리하면 되며, Mybatis는 개발자가 이 SQL 관리에만 집중할 수 있도록 도와준다.   

다음은 Mybatis로 변경한 코드이다.  

**BookDAO**
```
public class BoardDAO(){
  public List<BoardVO> getBoardList(BoardVO vo) {
    SqlSession mybatis = SqlSessionFactoryBean.getSqlSessionInstance();
    return mybatis.selectList("BoardDAO.getBoardList",vo);
  }
}
```
Mybatis는 XML 파일에 저장된 SQL 명령어를 대신 실행하고 실행 결과를 VO 같은 자바 객체에 자동으로 매핑까지 해준다.  
그래서 Mybatis 프레임워크를 데이터 맵퍼라고 부른다.  
결국 Mybatis프레임워크를 이용하여 DB 연동을 처리하면 대부분 한두 줄의 자바 코드만으로도 원하는 DB 연동 로직을 처리할 수 있게 된다.  

Mybatis의 2번째 특징은 Java 코드에서 SQL 구문을 분리하는 것이다.  
만약 SQL 명령어가 DAO 같은 자바 클래스에 저장되면 SQL 명령어만 수정하는 상황에서도 자바 클래스를 다시 컴파일해야한다.  
그리고 SQL 명령어들을 한 곳에 모아서 관리하기도 쉽지 않다.  
     
결국, SQL 명령어에 대한 통합 관리를 위해서라도 자바 소스에서 SQL을 분리하는 것은 매우 중요하다.  
  
다음은 SQL을 별도의 XML 파일에 작성한 것이다.  
**board-mapping.xml**
```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
			 "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper>
	<select id="getBoardList" resultType="board">
		select * from board
		where title like '%'||#{searchKeyword}||'%'
		order by seq desc
	</select>
</mapper>			 
```
구체적인 문법을 원한다면 별도의 교재나 Mybatis 홈페이지를 이용하자  

***
# 2. JAVA ORM Plugin 설치  
스프링을 구현을 위해 STS 플러그인을 설치했던 것과 마찬가지로  
Mybatis 역시 Java ORM이라는 플러그인 프로그램이 있어서   
이 플러그인을 사용하면 Mybatis와 관련된 복잡한 XML 설정 파일들을 자동으로 만들고 관리할 수 있다.  

1. [HELP] -> [Eclipse Marketplace] 메뉴를 선택한다.  
2. 검색창에 "ORM"입력후 ENTER 나 [GO]버튼 클릭   
3. Java ORM 플러그인을 찾아 ```<Install>``` 버튼을 클릭하여 설치 진행  

***
# 3. 프로젝트 생성
Mybatis 프레임워크의 구조와 개념을 이해하기 위해 Mybatis만으로 간단한 CRUD 기능을 테스트해보자  
사실 Mybatis를 스프링과 연동하지 않고 단독으로 사용하는 것은 여러 가지로 불편한 점이 많다.  
하지만 Mybatis의 구조와 기능 이해에 집중하기 위해서 간단한 프로젝트를 Mybatis만으로 수행해보는 것이 도움된다.  

나머지는 교재를 참고.........

**pom.xml**
```
		<!-- H2 데이터베이스 -->
		<dependency>
			<groupId>com.h2database</groupId>
			<artifactId>h2</artifactId>
			<version>1.4.200</version>
		</dependency>

		<!-- Mybatis -->
		<dependency>
			<groupId>org.mybatis</groupId>
			<artifactId>mybatis</artifactId>
			<version>3.3.1</version>
		</dependency>
		<!-- Ibatis -->
		<dependency>
			<groupId>org.apache.ibatis</groupId>
			<artifactId>ibatis-core</artifactId>
			<version>3.0</version>
		</dependency>
```

***
# 4. VO 클래스 작성
XML 파일에 저장된 SQL 명령어에 사용자가 입력한 값들을 전달하고 실행 결과를 매핑할 VO 클래스를 작성한다.  
기존에 게시판 프로그램에서 사용한 BoardVO 클래스와 같으므로 복사해서 사용해도 된다.  

**BoardVO**
```
package com.springbook.biz;

import java.util.Date;

import javax.xml.bind.annotation.XmlAccessType;
import javax.xml.bind.annotation.XmlAccessorType;
import javax.xml.bind.annotation.XmlAttribute;
import javax.xml.bind.annotation.XmlTransient;

import org.springframework.web.multipart.MultipartFile;

import com.fasterxml.jackson.annotation.JsonIgnore;

@XmlAccessorType(XmlAccessType.FIELD)
public class BoardVO {
	
	@XmlAttribute
	private int seq;
	private String title;
	private String writer;
	private String content;
	private Date regDate;
	private int cnt;
	@XmlTransient
	private String searchCondition;
	@XmlTransient
	private String searchKeyword;
	@XmlTransient
	private MultipartFile uploadFile;

	public int getSeq() {
		return seq;
	}

	public void setSeq(int seq) {
		this.seq = seq;
	}

	public String getTitle() {
		return title;
	}

	public void setTitle(String title) {
		this.title = title;
	}

	public String getWriter() {
		return writer;
	}

	public void setWriter(String writer) {
		this.writer = writer;
	}

	public String getContent() {
		return content;
	}

	public void setContent(String content) {
		this.content = content;
	}

	public Date getRegDate() {
		return regDate;
	}

	public void setRegDate(Date regDate) {
		this.regDate = regDate;
	}

	public int getCnt() {
		return cnt;
	}

	public void setCnt(int cnt) {
		this.cnt = cnt;
	}
	
	@JsonIgnore
	public String getSearchCondition() {
		return searchCondition;
	}
	
	public void setSearchCondition(String searchCondition) {
		this.searchCondition = searchCondition;
	}

	@JsonIgnore
	public String getSearchKeyword() {
		return searchKeyword;
	}

	public void setSearchKeyword(String searchKeyword) {
		this.searchKeyword = searchKeyword;
	}

	@JsonIgnore
	public MultipartFile getUploadFile() {
		return uploadFile;
	}

	public void setUploadFile(MultipartFile uploadFile) {
		this.uploadFile = uploadFile;
	}

	@Override
	public String toString() {
		return "BoardVO [seq=" + seq + ", title=" + title + ", writer=" + writer + ", content=" + content + ", regDate="
				+ regDate + ", cnt=" + cnt + "]";
	}
}

```
