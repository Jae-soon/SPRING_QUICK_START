31 JPA 환경설정
=======================
대부분 프레임워크는 실행에 필요한 다양한 환경설정을 XML 파일로 처리한다.  
JPA역시 VO 객체와 테이블을 매핑하기 위한 다양한 설정 정보가 필요한데,  
이를 위해 persistence.xml 파일을 환결설정 파일로 사용한다.  
  
persistence.xml 파일은 ```<persistence>```를 루트 엘리먼트로 사용하며,     
영속성 유닛과 관련된 다양한 정보가 설정된다.    
    
# 1. 영속성 유닛 설정
## 1.1. 영속성 유닛 이름 지정
영속성 유닛은 연동할 데이터베이스당 하나씩 등록하며,  
영속성 유닛에 설정된 이름은 나중에 DAO 클래스를 구현할 때 EntityManagerFactory 객체 생성에 사용된다.   

**persistence.xml**
```
<?xml version="1.0" encoding="UTF-8"?>
<persistence version="2.1" xmlns="http://xmlns.jcp.org/xml/ns/persistence" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/persistence http://xmlns.jcp.org/xml/ns/persistence/persistence_2_1.xsd">
	<persistence-unit name="JPAProject">
		<class>com.springbook.biz.board.Board</class>
		<properties>
			<!-- 필수 속성 -->
			<property name="javax.persistence.jdbc.driver" value="org.h2.Driver"/>
			<property name="javax.persistence.jdbc.user" value="sa"/>
			<property name="javax.persistence.jdbc.password" value=""/>
			<property name="javax.persistence.jdbc.url" value="jdbc:h2:tcp://localhost/~/test"/>	
			<property name="hibernate.dialect" value="org.hibernate.dialect.H2Dialect"/>
			
			<!-- 옵션 -->
			<property name="hibernate.show_sql" value="true"/>
			<property name="hibernate.format_sql" value="true"/>
			<property name="hibernate.use_sql_comments" value="false"/>
			<property name="hibernate.id.new_generator_mappings" value="true"/>
			<property name="hibernate.hbm2ddl.auto" value="create"/>
		</properties>
	</persistence-unit>
</persistence>
```
```
~생략~
	<persistence-unit name="JPAProject">
~생략~
```

**BoardServiceClient**
```
package com.springbook.biz.board;

import java.util.List;

import javax.persistence.EntityManager;
import javax.persistence.EntityManagerFactory;
import javax.persistence.EntityTransaction;
import javax.persistence.Persistence;

public class BoardServiceClient {

	public static void main(String[] args) {
		// EntityManager 생성
		EntityManagerFactory emf = Persistence.createEntityManagerFactory("JPAProject");
		EntityManager em = emf.createEntityManager();

		// Transaction 생성
		EntityTransaction tx = em.getTransaction();

		try {
			// Transaction 시작
			tx.begin();

			Board board = new Board();
			board.setTitle("JPA 제목");
			board.setWriter("관리자");
			board.setContent("JPA 글 등록 잘되네요");

			// 글 등록
			em.persist(board);

			// 글 목록 조회
			String jpql = "select b from Board b order by b.seq desc";
			List<Board> boardList = em.createQuery(jpql, Board.class).getResultList();

			for (Board brd : boardList) {
				System.out.println("--->" + brd.toString());
			}
			// Transaction commit
			tx.commit();
		} catch (Exception e) {
			e.printStackTrace();
			tx.rollback();
		} finally {
			em.close();
		}
		emf.close();
	}

}
```
```
~ 생략 ~
		EntityManagerFactory emf = Persistence.createEntityManagerFactory("JPAProject");
~ 생략 ~ 
```
JPA를 이용하여 DB 연동을 구현하려면 EntityManager 객체가 필요하다.    
그런데 EntityManager를 얻으려면 EntityManger 객체를 생성하기 위한 공장 기능의 EntityManagerFactory 객체가 필요하다.    
이 EntityManagerFactory를 생성할 때, 영속성 유닛이 사용된다.  
  
## 1.2. 엔티티 클래스 등록  
영속성 유닛 설정에서 가장 먼저 엔티티 클래스를 등록하는데,  
이 엔티티 클래스는 JPA 프로젝트에서 JPA Entity 클래스를 작성하는 순간 자동으로 persistence.xml 파일에 등록된다.    
**persistence.xml**
```
	<persistence-unit name="JPAProject">
		<class>com.springbook.biz.board.Board</class>
 	</persistence-unit>
```
사실 스프링 프레임워크나 J2EE 환경에서 JPA를 사용한다면   
자동으로 엔티티 클래스를 검색하여 처리하는 기능이 제공되므로 엔티티 클래스들을 일일이 등록할 필요는 없다.    
하지만 JPA를 단독으로 사용하는 경우라면 엔티티 클래스를 등록하는 것이 가장 확실한 방법이므로 이 방법을 권장한다.    
  
## 1.3. 영속성 유닛 프로퍼티 설정 
엔티티 클래스를 등록했으면 이제 엔티티 클래스의 영속성 관리에 필요한 프로퍼티들을 설정해야 하는데,  
이중에서 가장 기본이면서 중요한 것이 DB 커넥션 관련 설정이다.  
이 DB 커넥션 정보를 바탕으로 하이버네이트 같은 JPA 구현체가 특정 데이터베이스와 커넥션을 맺을 수 있기 때문이다.  
  
하지만 스프링 프레임워크와 연동할 때는   
데이터소스가 스프링 설정 파일에 등록되어 있으므로 영속성 유닛 설정에서는 제거될 수도이 있다.      
   
**persistence.xml**
```
<!-- 필수 속성 -->
			<property name="javax.persistence.jdbc.driver" value="org.h2.Driver"/>
			<property name="javax.persistence.jdbc.user" value="sa"/>
			<property name="javax.persistence.jdbc.password" value=""/>
			<property name="javax.persistence.jdbc.url" value="jdbc:h2:tcp://localhost/~/test"/>	
```			
각 프로퍼티의 의미는 아래 표와 같다.  
  
[사진]  
  
```
javax.persistence.jdbc.driver     : JDBC 드라이버 클래스
javax.persistence.jdbc.user       : 데이터베이스 아이디
javax.persistence.jdbc.password   : 데이터베이스 비밀번호
javax.persistence.jdbc.url        : JDBC URL 정보
```
  
## 1.4. Dialect 클래스 설정


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
