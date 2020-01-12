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
ORM 프레임워크의 가장 중요한 특징이자 장점은 애플리케이션 수행에 필요한 SQL 구문을 자동으로 생성한다는 것이다.    
그런데 문제는 이 SQL을 아무리 표준에 따라서 잘 작성한다고 해도 DBMS에 따라서 키 생성 방식도 다르고 지원되는 함수도 조금씩 다르다.  
따라서 DBMS가 변경되면 이런 세세한 부분을 개발자가 적절하게 수정해야 한다.  
    
JPA 는 특정 DBMS에 최적화된 SQL을 제공하기 위해서 DBMS마다 다른 Dialect 클래스를 제공한다.     
Dialect는 사투리 혹은 방언이라는 의미인데, 이 Dialect 클래스가 해당 DBMS에 최적화된 SQL 구문을 생성한다.      
우리는 H2 데이터베이스를 사용하고 있으므로 H2Dialect 클래스를 등록하면 된다.     
```
			<property name="hibernate.dialect" value="org.hibernate.dialect.H2Dialect"/>
```
이제 DBMS가 변경되는 경우 Dialect 클래스만 변경하면 SQL이 자동으로 변경되어 생성되므로 유지보수는 크게 향상될 것이다.    
Dialect 클래스들의 정확한 위치는 Maven Dependencies에서 hibernate-core-5.1.0.Final.jar 파일이다.  
직접 해당 라이브러리를 확인해보면 알겠지만 현존하는 대부분 관계형 데이터베이스에 해당하는 Dialect 클래스들이 제공된다.  
  
[사진]   
   
```

DB2			: org.hibernate.dialect.DB2Dialect
PostgreSQL		: org.hibernate.dialect.PostgreDialect
MySQL			: org.hibernate.dialect.MySQLDialect
Oracle			: org.hibernate.dialect.OracleDialect
Oracle 9i/10g		: org.hibernate.dialect.Oracle9Dialect
Sybase			: org.hibernate.dialect.SybaseDialect
Microsoft SQL Server	: org.hibernate.dialect.SQLServerDialect
SAP DB			: org.hibernate.dialect.SAPDBDialect
H2			: org.hibernate.dialect.H2Dialect

```
  
## 1.5. JPA 구현체 관련 속성 설정  
마지막으로 하이버네이트 관련 속성 설정이 등장하는데,   
이는 우리가 사용할 JPA 구현체가 하이버네이트 프레임워크이기 때문이다.   

```
			<!-- 옵션 -->
			<property name="hibernate.show_sql" value="true"/>
			<property name="hibernate.format_sql" value="true"/>
			<property name="hibernate.use_sql_comments" value="false"/>
			<property name="hibernate.id.new_generator_mappings" value="true"/>
			<property name="hibernate.hbm2ddl.auto" value="create"/>
```
각 속성의 의미는 아래 표와 같다.
   
[사진]   
    
```
hibernate.show_sql			: 생성된 SQL을 콘솔에 출력한다.
hibernate.format_sql			: SQL을 출력할 때, 일정한 포맷으로 보기 좋게 출력한다.	
hibernate.use_sql_comments		: SQL에 포함된 주석도 같이 출력한다.
hibernate.id.new_generator_mappings	: 새로운 키 생성 전력을 사용한다.  
hibernate.hbm2ddl.auto			: 테이블 생성이나 수정, 삭제 같은 DDL 구문을 자동으로 처리할지를 지정한다.
```
이 중에서 DDL 명령어와 관련된 ```hibernate.hbm2ddl.auto``` 설정이 중요한데,  
속성값으로 아래 표와 같은 것들을 사용할 수 있다.  
  
[사진]  
  
```
create		: 애플리케이션을 실행할 때, 기존 테이블을 삭제하고 엔티티 클래스에 설정된 매핑 설정을 참조하여 새로운 테이블을 생성한다.(DROP -> CREATE)
create-drop	: create 와 같지만 애플리케이션이 종료되기 직전에 생성된 테이블을 삭제한다. (DROP -> CREATE -> DROP)
update 		: 기존에 사용 중인 테이블이 있으면 새 테이블을 생성하지 않고 재사용한다. 만약 엔티티 클래스의 매핑 설정이 변경되면 변경된 내용만 반영한다. (ALTER) 
```
이전에 작업했던 JPAProject를 실행시켜보면 create로 되어 있으므로 기존 테이블을 삭제했다 다시 만드는 것을 알 수 있다.  
```
Hibernate: 
    drop table BOARD if exists
Hibernate: 
    drop sequence if exists hibernate_sequence
Hibernate: create sequence hibernate_sequence start with 1 increment by 1
Hibernate: 
    create table BOARD (
        seq integer not null,
        cnt integer not null,
        content varchar(255),
        regDate date,
        title varchar(255),
        writer varchar(255),
        primary key (seq)
    )
```
DDL 자동 생성 기능은 애플리케이션 실행 시점에 테이블이 자동으로 생성되므로 굉장히 편리해 보인다.    
하지만 일반적으로 프로젝트 초기에 데이터 모델링이 마무리되고 나서 비즈니스 컴포넌트 개발로 들어가므로    
이 기능을 사용할 일은 거의 없을 것이다.  
다만 초기에 엔티티 클래스와 테이블 매핑을 연습하는 상황에서는 이 기능을 사용하면 학습에 도움된다.  

다음은 영속성 유닛 관리에 필요한 기본적인 내용이 모두 설정된 persistence.xml 파일이다.
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
   
***
# 2. 엔티티 클래스 기본 매핑
JPA의 기본은 엔티티 클래스를 기반으로 관계형 데이터베이스에 저장된 데이터를 관리하는 것이다.  
엔티티 클래스를 작성하는데 아주 복잡한 규칙은 없다.   
물론 public 클래스로 만들어야 하고, 기본 생성자가 반드시 있어야 하는 등의 제약은 있지만,  
이런 제약은 큰 의미가 없다.    
그냥 ValueObject 처럼 POJO 클래스로 작성하면 된다.  
  
사실 엔티티 매핑에서 가장 복잡하고 중요한 설정은 연관 매핑 설정이다.    
연관 매핑은 JPA에서 가장 복잡하고 중요한 개념이지만 전문적인 지식을 요구하므로      
이번 시간에는 가장 기본적인 엔티티 매핑 정보만 확인하도록 하자     
   
## 2.1. @Entity, @Id
@Entity는 특정 클래스를 JPA가 관리하는 엔티티 클래스로 인식하는 가장 중요한 어노테이션이다.  
엔티티 클래스 선언부 위에 @Entity를 붙이면 JPA가 이 클래스를 엔티티 클래스로 인식하여 관련된 테이블과 자동으로 매핑 처리한다.    

엔티티 클래스와 매핑되는 테이블은 각 ROW를 식별하기 위한 PK 칼럼을 가지고 있다.  
이런 테이블과 매핑되는 엔티티 클래스 역시 PK 칼럼과 매핑될 변수를 가지고 있어야 하며,  
이런 변수를 식별자 필드라고 한다.   
이 식별자 필드는 엔티티 클래스라면 무조건 가지고 있어야 하며 @Id를 이용하여 선언한다.  

**Board 일부**
```
package com.springbook.biz.board;

import java.util.Date;

import javax.persistence.Entity;
import javax.persistence.GeneratedValue;
import javax.persistence.Id;
import javax.persistence.Table;
import javax.persistence.Temporal;
import javax.persistence.TemporalType;


@Entity
@Table(name = "BOARD")
public class Board{
	@Id
	@GeneratedValue
	private int seq;
	private String title;
	private String writer;
	private String content;
	@Temporal(TemporalType.DATE)
	private Date regDate = new Date();
	private int cnt;
```
@Entity가 추가된 Board 클래스는 BOARD 테이블과 자동으로 매핑된다.  
만약 Board 클래스와 다른 테이블을 매핑하려면 @Table을 사용해야 한다.  


***
# 3. 대주제
> 인용
## 3.1. 소 주제
### 3.1.1. 내용1
```
내용1
```
