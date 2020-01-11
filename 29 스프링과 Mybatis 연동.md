29 스프링과 Mybatis 연동
=======================
Mybatis 프레임워크는 Ibatis에서 파생되었으므로 Mybatis의 전체적인 구조는 Ibatis와 매우 유사하다.  
심지어 XML 설정 파일들과 자바 API 까지도 Ibatis와 거의 유사하다.    
   
하지만 Mybatis 는 스프링 프레임워크와의 연동 부분에서 Ibatis와 다른데  
이번 학습에서는 스프링과 Mybatis의 연동 부분을 중점으로 살펴보고자 한다.   
그리고 지금부터 진행하는 모든 학습은 이전 시간에 사용했던 MybatisProject가 아닌   
BoardWeb 프로젝트에서 진행할 것이다.

# 1. 라이브러리 내려받기 
스프링과 Ibatis 프레임워크를 연동할 때는 스프링 쪽에서 Ibatis 연동을 위한 API를 지원했다.  
다음은 스프링 프레임워크에서 제공하는 Ibatis 연동에 사용되는 클래스들이다.   
   
```java
org.springframework.orm.ibatis.SqlMapClientFactoryBean
org.springframework.orm.ibatis.SqlMapClientTemplate
```
그러나 Mybatis는 스프링 쪽에서 연동에 필요한 API를 제공하지 않으며,  
오히려 Mybatis에서 스프링 연동에 필요한 API를 제공한다.  
따라서 스프링과 Mybatis를 연동하려면 Mybatis에서 제공하는 다음과 같은 클래스들을 이용해서 연동해야한다.  
```java
org.mybatis.spring.SqlSessionFactoryBean
org.mybatis.spring.SqlSessionTemplate
```
스프링과 Mybatis 연동에 필요한 라이브러리들을 내려 받으려면 pom.xml 파일에 ```<dependency>```를 추가한다.  
   
**pom.xml**  
```
~ 생략 ~
		<!-- Mybatis -->
		<dependency>
			<groupId>org.mybatis</groupId>
			<artifactId>mybatis</artifactId>
			<version>3.3.1</version>
		</dependency>
		<!-- Mybatis Spring-->
		<dependency>
			<groupId>org.mybatis</groupId>
			<artifactId>mybatis-spring</artifactId>
			<version>1.2.4</version>
		</dependency>
		<!-- Ibatis -->
		<dependency>
			<groupId>org.apache.ibatis</groupId>
			<artifactId>ibatis-core</artifactId>
			<version>3.0</version>
		</dependency>
~ 생략 ~
```
mybatis-3.3.1.jar 파일은 순수하게 Mybatis 관련 라이브러리고,  
mybatis-spring-1.2.4.jar 파일은 Mybatis와 스프링을 연동하기 위해 사용하는 라이브러리다.  
   
***
# 2. Mybatis 설정 파일 복사 및 수정 
스프링과 Mybatis를 연동하려면 Mybatis 메인 환경설정 파일인 sql-map-config.xml과  
SQL 명령어들이 저장되어 있는 Mapper 파일이 필요하다.   
따라서 MybatisProject에서 작성했던 XML 설정 파일들을 복사하여 BoardWeb 프로젝트의 ```src/main/resources```소스 폴더에 추가한다.  
     
이 중에서 Mybatis 메인 환경설정 파일인 ```sql-map-config.xml```을 열어서 데이터소스 관련 설정을 삭제한다.    
   
**sql-map-config.xml**
```
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
  PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
  "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
	<typeAliases>
		<typeAlias type="com.springbook.biz.BoardVO" alias="board"></typeAlias>
	</typeAliases>
	<mappers>
		<mapper resource="mappings/board-mapping.xml" />
	</mappers>
</configuration>
```
데이터 소스는 스프링 프레임워크에서 이미 등록하여 사용하고 있었다.  
그리고 이 데이터 소스는 DB 연동뿐만 아니라 트랜잭션 처리처럼 다양한 곳에서 사영할 수 있으므로   
Mybatis 설정이 아닌 스프링 설정 파일에서 제공하는 것이 맞다.(applicationContext.xml)   
그리고 SQL 명령어가 저장된 Mapper XML 파일은 수정 없이 그대로 재사용하면 된다.   
   
***
# 3. 스프링 연동 설정   
스프링 Mybatis를 연동하려면 우선 스프링 설정 파일에 SqlSessionFactoryBean 클래스를 Bean 등록해야한다.    
그래야 SqlSessionFactoryBean 객체로부터 DB 연동 구현에 사용한 SqlSession 객체를 얻을 수 있다.  
  
**applicationContext.xml**
```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xmlns:p="http://www.springframework.org/schema/p"
	xmlns:context="http://www.springframework.org/schema/context"
	xmlns:aop="http://www.springframework.org/schema/aop"
	xmlns:tx="http://www.springframework.org/schema/tx"
	xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
		http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context-4.2.xsd
		http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop-4.2.xsd
		http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx-4.2.xsd">

	<context:component-scan base-package="com.springbook.biz"></context:component-scan>
	<context:property-placeholder location="classpath:config/database.properties"/>
	
	<bean id="dataSource" class="org.apache.commons.dbcp.BasicDataSource" destroy-method="close">
		<property name="driverClassName" value="org.h2.Driver"/>
		<property name="url" value="jdbc:h2:tcp://localhost/~/test" />
		<property name="username" value="sa"/>
		<property name="password" value=""/>
	</bean>
	<!-- SqlSessionFactoryBean 생성 -->
	<bean id="sessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
		<property name="dataSource" ref="dataSource"/>
		<property name="configLocation" value="classpath:sql-map-config.xml" />
	</bean>
	<!-- Spring JDBC 설정 -->
	<bean id="jdbcTemplate" class="org.springframework.jdbc.core.JdbcTemplate">
		<property name="dataSource" ref="dataSource"/>
	</bean>
	<!-- Transaction 실행 -->
	<bean id="txManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
		<property name="dataSource" ref="dataSource"></property>
	</bean>
	<tx:advice id="txAdvice" transaction-manager="txManager">
		<tx:attributes>
			<tx:method name="get*" read-only="true"/>
			<tx:method name="*"/>
		</tx:attributes>
	</tx:advice>
	<aop:config>
		<aop:pointcut expression="execution(* com.springbook.biz..*(..))" id="txPointcut"/>
		<aop:advisor pointcut-ref="txPointcut" advice-ref="txAdvice"/>
	</aop:config>
</beans>
```
MybatisProject에서는 SqlSession 객체를 얻기 위해 SqlSessionFactoryBean 클래스를 유틸리티 클래스로 직접 구현했었다.   
하지만 이 클래스를 Mybatis에서 제공하므로 굳이 작성할 필요 없이 스프링 설정 파일에 ```<bean>``` 등록하면 된다.  
  
SqlSessionFactoryBean 객체가 SqlSession 객체를 생산하려면 반드시 DataSource와 SQLMapper 정보가 필요하다.  
따라서 앞에 등록된 DataSource를 Setter 인젝션으로 참조하고,  
SQL Mapper가 등록된 sql-map-config.xml 파일도 Setter 인젝션으로 설정해야 한다.    
그래야 ```<Bean>``` 등록된 SqlSessionFactoryBean이 SqlSessionFactory 객체를 생성할 수 있다.  
   
***
# 4. DAO 클래스 구현-방법1
Mybatis를 이용하여 DAO 클래스를 구현하는 방법은 2가지이다.   
이 중에 첫 번째는 SqlSessionDaoSupport 클래스를 상속하여 구현하는 것이다.  

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
