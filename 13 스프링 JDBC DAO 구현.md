13 스프링 JDBC DAO 구현
=======================
# 1. 첫 번째 방법 : JdbcDaoSupport 클래스 상속  
첫 번째 방법은 JdbcDaoSupport 클래스를 상속하는 방법이다.     
    
**BoardDAOSpring**
```
package com.springbook.biz.board.impl;

import java.sql.Connection;
import java.sql.PreparedStatement;
import java.sql.ResultSet;
import java.sql.SQLException;
import java.util.ArrayList;
import java.util.List;

import javax.sql.DataSource;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.jdbc.core.RowMapper;
import org.springframework.jdbc.core.support.JdbcDaoSupport;
import org.springframework.stereotype.Repository;

import com.springbook.biz.BoardVO;
import com.springbook.biz.common.JDBCUtil;

@Repository
public class BoardDAOSpring extends JdbcDaoSupport {

	private final String BOARD_INSERT = "INSERT INTO board(seq, title, writer, content) "
			+ "VALUES((SELECT nvl(max(seq),0)+1 FROM board),?,?,?);"; // AUTOINCREMENT 기능 직접 추가 nvl(인자1, 인자2) 인자1이 null일
																		// 경우 인자2를 사용하겠다는 뜻
	private final String BOARD_UPDATE = "UPDATE BOARD SET title=?, content=? WHERE seq=?;";
	private final String BOARD_DELETE = "DELETE BOARD WHERE seq=?;";
	private final String BOARD_GET = "SELECT * FROM board WHERE seq=?;";
	private final String BOARD_LIST = "SELECT * FROM board ORDER BY seq DESC;";

	@Autowired
	public void setSuperDataSource(DataSource dataSource) {
		super.setDataSource(dataSource);
	}

	public void insertBoard(BoardVO vo) {
		System.out.println("===> JDBC로 insertBoard() 기능 처리");
		getJdbcTemplate().update(BOARD_INSERT, vo.getTitle(), vo.getWriter(), vo.getContent());
	}

	public void updateBoard(BoardVO vo) {
		System.out.println("===> JDBC로 updateBoard() 기능 처리");
		getJdbcTemplate().update(BOARD_UPDATE, vo.getTitle(), vo.getContent(), vo.getSeq());

	}

	public void deleteBoard(BoardVO vo) {
		System.out.println("===> JDBC로 deleteBoard() 기능 처리");
		getJdbcTemplate().update(BOARD_DELETE, vo.getSeq());
	}

	public BoardVO getBoard(BoardVO vo) {
		System.out.println("===> JDBC로 getBoard() 기능 처리");
		Object[] args = { vo.getSeq() };
		return getJdbcTemplate().queryForObject(BOARD_GET, args, new BoardRowmapper());

	}

	public List<BoardVO> getBoardList(BoardVO vo) {
		System.out.println("===> JDBC로 getBoardList()기능 처리");
		return getJdbcTemplate().query(BOARD_GET, new BoardRowmapper());
	}

}

class BoardRowmapper implements RowMapper<BoardVO> {
	public BoardVO mapRow(ResultSet rs, int rowNum) throws SQLException {
		BoardVO board = new BoardVO();
		board.setSeq(rs.getInt("SEQ"));
		board.setTitle(rs.getString("TITLE"));
		board.setWriter(rs.getString("WRITER"));
		board.setContent(rs.getString("CONTENT"));
		board.setRegDate(rs.getDate("REGDATE"));
		board.setCnt(rs.getInt("CNT"));
		return board;
	}
}
```
DAO 클래스를 구현할 때, JdbcDaoSupport 클래스를 부모 클래스로 지정하면 ```getJdbcTemplate()``` 메소드를 상속받을 수 있다.    
그리고 ```getJdbcTemplate()```를 호출하면 JdbcTemplate 객체가 리턴되어 모든 메소드를 JdbcTemplate 객체로 구현할 수 있다.  
  
그런데 문제는 ```getJdbcTemplate()``` 메소드가 JdbcTemplate 객체를 리턴하려면 DataSource 객체를 가지고 있어야한다.  
따라서 부모 클래스인 JdbcDaoSupport 에 ```setDataSource()```메소드를 호출하여 데이터소스 객체를 의존성 주입해야 한다.   

```
@Autowired
public void setSuperDataSource(DataSource dataSource) {
	super.setDataSource(dataSource);
}
```
```@Autowired``` 어노테이션은 주로 변수 위에 선언하는데 메소드 위에 선언해도 동작한다.     
메소드 위에 ```@Autowired```를 붙이면 해당 메소드를 스프링 컨테이너가 자동으로 해출해주며,     
이때 메소드 매개변수 타입을 확인하고 해당 타입의 객체가 메모리에 존재하면 그 객체를 인자로 전달해준다.
   
## 1.1. 소 주제
### 1.1.1. 내용1
```
내용1
```
## 1.2. 소 주제
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
