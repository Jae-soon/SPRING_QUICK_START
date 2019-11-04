AOP 용어 및 기본 설정
=======================
# 1. AOP 용어 정리  
## 1.1. 조인포인트
조인포인트는 클라이언트가 호출하는 모든 비즈니스 메소드로서,  
BoardServiceImpl이나 UserServiceImpl 클래스의 모든 클래스를 조인포인트라고 생각하면 된다.    
(인터페이스를 기준으로 해당하는 모든 비즈니스 메소드를 실행하는 중간 역할 클래스)    
  
조인포인트를 '포인트컷 대상' 또는 '포인트컷 후보'라고도 하는데,  
이는 조인포인트 중에서 포인트컷이 선택되기 때문이다.  
  
## 1.2. 포인트컷
클라이언트가 호출하는 모든 비즈니스 메소드가 조인포인트라면,  
포인트컷은 필터링된 조인포인트를 의미한다.       
  
트랜잭션을 처리하는 공통 기능을 만들었다고 가정시  
이 횡단 관심 기능은 등록, 수정, 삭제 기능의 비즈니스 메소드에 대해서는 당연히 동작해야하지만,    
검색 기능의 메소드에 대해서는 트랜잭션과 무관하므로 동작할 필요가 없다.   
  
수많은 비즈니스 메소드 중에서 우리가 원하는 특정 메소드에서만  
횡단 관심에 해당하는 공통기능을 수행시키기 위해서 포인트컷이 필요하다.   
     
포인트컷을 이용하면 메소드가 포함된 클래스와 패키지는 물론이고  
메소드 시그니처까지 정확하게 지정할 수 있다.  
  
```
	<context:component-scan base-package="com.springbook.biz"></context:component-scan>
	
	<bean id="log" class="com.springbook.biz.common.Log4jAdvice"></bean>
	<aop:config>
		<aop:pointcut expression="execution(* com.springbook.biz..*Impl.*(..))" id="allPointcut"/>
		<aop:pointcut expression="execution(* com.springbook.biz..*Impl.get*(..))" id="getPointcut"/>
		
		<aop:aspect ref="log">
			<aop:before pointcut-ref="getPointcut" method="printLogging"/>
		</aop:aspect>
	</aop:config>
```
포인트컷은 ```<aop:pointcut>```엘리먼트로 선언하며,  
id 속성으로 포인트컷을 식별하기 위한 유일한 문자열을 선언한다.  
이 id 가 나중에 포인트컷을 참조할 때 사용된다.   
   
**중요한 것은 expression 속성인데, **  
이 값을 어떻게 설정하느냐에 따라 필터링되는 메소드가 달라진다.    
```
<aop:pointcut id="getPointcut" expression="execution(* com.springbook.biz..*Impl.get*(..))" />
      
*                       : 리턴 타입 
com.springbook.biz.     : 패키지 경로 
*Impl                   : 클래스명 
get*(..)                : 메소드명 및 매개변수

중간 마다 나오는 점은 각 범위를 구분해주는 구분 점이라고 생각하면 된다.  
```
```
INFO : org.springframework.beans.factory.xml.XmlBeanDefinitionReader - Loading XML bean definitions from class path resource [applicationContext.xml]
INFO : org.springframework.context.support.GenericXmlApplicationContext - Refreshing org.springframework.context.support.GenericXmlApplicationContext@782830e: startup date [Mon Nov 04 20:02:27 KST 2019]; root of context hierarchy
INFO : org.springframework.beans.factory.annotation.AutowiredAnnotationBeanPostProcessor - JSR-330 'javax.inject.Inject' annotation found and supported for autowiring
===> JDBC로 insertBoard() 기능 처리
[공통 로그-Log4j] 비즈니스 로직 수행 전 동작
===> JDBC로 getBoardList()기능 처리
---> BoardVO [seq=5, title=임시 제목, writer=홍길동, content=임시 내용.........., regDate=2019-11-04, cnt=0]
---> BoardVO [seq=4, title=임시 제목, writer=홍길동, content=임시 내용.........., regDate=2019-11-02, cnt=0]
---> BoardVO [seq=3, title=임시 제목, writer=홍길동, content=임시 내용.........., regDate=2019-11-02, cnt=0]
---> BoardVO [seq=2, title=임시 제목, writer=홍길동, content=임시 내용.........., regDate=2019-11-01, cnt=0]
---> BoardVO [seq=1, title=가입인사, writer=관리자, content=잘 부탁드립니다..., regDate=2019-10-27, cnt=0]
INFO : org.springframework.context.support.GenericXmlApplicationContext - Closing org.springframework.context.support.GenericXmlApplicationContext@782830e: startup date [Mon Nov 04 20:02:27 KST 2019]; root of context hierarchy

```
  

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
