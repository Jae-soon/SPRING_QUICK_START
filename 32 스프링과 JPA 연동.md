32 스프링과 JPA 연동
=======================
# 1. 스프링과 JPA 연동 기초  
## (1) 프로젝트 변경    
스프링과 JPA를 연동하려면 우선 BoardWeb프로젝트를 JPA 프로젝트로 변환해야 한다.      
     
1. 기존에 사용하던 BoardWeb 프로젝트를 선택하고 마우스 오른쪽 버튼을 클릭한다.      
2. 맨 아래에 있는 [properties]를 클릭하여 Properties창을 띄운다.      
3. 마지막으로 왼쪽의 [project Facets] 선택하여 JPA 항목을 체크한다.        
4. ```<APPLY>```,```<OK>``` 버튼이 활성화 되지 않으면 Futher configuration을 클릭하여     
Disable Library로 전환 시켜주면 활성화가 된다.   
5. ```<APPLY>```,```<OK>``` 버튼을 클릭하면 META-INF에 persistence.xml이 생성된다.  
  
## (2) 라이브러리 내려받기
BoardWeb 프로젝트가 JPA 프로젝트로 변경되었으면,  
이제 pom.xml 파일을 수정하여 SpringORM 라이브러리와 하이버네이트 라이브러리를 내려받는다.  
   
**pom.xml**
```
		<!-- spring-ORM -->
		<dependency>
			<groupId>org.springframework</groupId>
			<artifactId>spring-orm</artifactId>
			<version>${org.springframework-version}</version>
		</dependency>
    
		<!-- JPA, 하이버네이트 -->
		<dependency>
			<groupId>org.hibernate</groupId>
			<artifactId>hibernate-entitymanager</artifactId>
			<version>5.1.0.Final</version>
		</dependency>
```
내려받기가 마무리되면 Maven Dependencies에서 해당하는 라이브러리를 추가로 확인한다.  
   
## (3) JPA 설정 파일 작성  
JPA 프로젝트는 반드시 영속성 유닛 설정 정보가 저장된 persistence.xml 파일이 필요하다.  
하지만 persistence.xml 파일은 JPAProject에 작성된  파일과 같으므르 해당 파일을 복사해서 재사용하자    
    
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
