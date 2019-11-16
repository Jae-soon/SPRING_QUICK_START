11 JoinPoint와 바인드 변수
=======================
**조인포인트** : advice 메소드가 실행 될 수 있는 모든 메소드를 의미 (비즈니스 메소드)   

어드바이스 메소드는 비즈니스 메소드의 정보를 이용해 보다 더 의미 있게 구현할 수 있다.        
스프링에서는 어드바이스 메소드에서 비지니스 메소드 정보들을 이용할 수 있도록 JoinPoint 인터페이스를 제공한다.       
          
# 1. JoinPoint 인터페이스와 메소드
**UserServiceClient-사용할 파일**
```
package com.springbook.biz.user.impl;

import org.springframework.context.support.AbstractApplicationContext;
import org.springframework.context.support.GenericXmlApplicationContext;

import com.springbook.biz.user.UserVO;

public class UserServiceClient {

	public static void main(String[] args) {
		// TODO Auto-generated method stub
		
		// 1. Spring 컨테이너 구동한다.  
		AbstractApplicationContext container =
				new GenericXmlApplicationContext("applicationContext.xml");
		
		// 2. Spring 컨테이너로부터 UserServiceImpl 객체를 Lookup 한다.  
		UserService userService = 
				(UserService)container.getBean("userService");
		
		// 3. 로그인 기능 테스트
		UserVO vo = new UserVO();
		vo.setId("test");
		vo.setPassword("test123");
		
		UserVO user = userService.getUser(vo);
		if(user != null ) {
			System.out.println(user.getName() + "님 환영합니다.");
		} else {
			System.out.println("로그인 실패");
		}
		
		// 4. Spring 컨테이너를 종료한다.  
		container.close();
	}

}
```
## 1.1. 어드바이스 클래스의 메소드
```
public void printLog(JoinPoint jp) {
	System.out.println("[공통 로그] 비즈니스 로직 수행 전 동작");
}
```
어드바이스 메소드에 JoinPoint 인터페이스를 사용하려 한다면 단순히 매개변수에 ```JoinPoint jp```를 기술해주면 된다.    
**하지만 바로 jp 변수를 사용하는 것이 아니고 jp 변수를 이용해 주제에 맞는 객체를 얻는 것이 우선이다.**

## 1.2. JoinPoint 객체 반환 메소드
```
Signature getSignature() : 클라이언트가 호출한 메소드의 시그니처 정보가 저장된 Signature 객체 리턴  
Object getTarget() : 클라이언트가 호출한 비즈니스 메소드를 포함하는 비즈니스 객체 리턴
Object[] getArgs() : 클라이언트가 메소드를 호출할 때 넘겨준 인자 목록을 Object 배열로 리턴  
```
주의할 점은 Before, After Returning, After Throwing, After 어드바이스에서는 JoinPoint를 사용해야 하고,   
**Around 어드바이스에서만 ProceedingJoinPoint를 매개변수로 사용해야 한다.**

PoroceedingJoinPoint 인터페이스는 JoinPoint를 상속한다.      
그렇기에 PoroceedingJoinPoint 인터페이스는 JoinPoint가 가진 메소드를 지원하며,      
proceed() 메소드를 추가해서 사용할 수 있다. (기존에 가로챈 메소드를 호출할 수 있게 해준다.)    
   
Around 어드바이스는 proceed()메소드가 필요하기 때문에 PoroceedingJoinPoint 를 매개변수로 갖는다.
   
## 1.3. Signature 객체   
```
sigObj = jp.getSignature();
```
getSignature() 메소드가 리턴하는 Signature 객체를 이용하면, 호출되는 메소드에 대한 다양한 정보를 얻을 수 있다.   
```
String getName() : 클라이언트가 호출한 메소드 이름 리턴
String toLongString() : 클라이언트가 호출한 메소드의 리턴타입, 이름, 매개변수를 패키지 경로까지 포함하여 리턴 
String toShortString() : 클라이언트가 호출한 메소드 시그니처를 축약한 문자열로 리턴  
```
**JoinPoint는 어드바이스의 종류(5가지)에 따라 사용 방법이 다소 다르다.**   
그렇기에 앞으로 각 어드바이스마다 사용 방법에 대해서 배워보자   
   
***
# 2. Before 어드바이스
**Before 어드바이스는 비즈니스 메소드가 실행되기 전에 동작할 로직을 구현한다.**  
따라서 호출된 메소드 시그니처만 알 수 있으면 다양한 사전 처리 로직을 구현할 수 있다.

**BeforeAdvice**
```
// 어드바이스 클래스 
package com.springbook.biz.common;

import org.aspectj.lang.JoinPoint;

public class BeforeAdvice {
	public void beforeLog(JoinPoint jp) {
		String method = jp.getSignature().getName();
		Object[] args = jp.getArgs();
		
		System.out.println("[사전 처리]" + method + "() 메소드 ARGS 정보 : "+ args[0].toString());
	}
}
```
JoinPoint 객체의 ```getSignature()``` 메소드를 이용하면,    
```getName()``` 메소드를 통해 클라이언트가 호출한 비즈니스 메소드의 이름을 출력할 수 있다.      
   
그리고 ```getArgs()``` 메소드를 통해 인자로 사용된 객체들을 Object 배열로 얻어낼 수 있어서 이를 처리할 수 있다. 
위 코드에서는 우리가 앞서 ```UserVO```의 ```toString()``` 메소드를 오버라이딩 한 것을 호출하여 정보를 얻었다.       
**결과**
```
INFO : org.springframework.beans.factory.xml.XmlBeanDefinitionReader - Loading XML bean definitions from class path resource [applicationContext.xml]
INFO : org.springframework.context.support.GenericXmlApplicationContext - Refreshing org.springframework.context.support.GenericXmlApplicationContext@782830e: startup date [Sat Nov 16 22:33:30 KST 2019]; root of context hierarchy
INFO : org.springframework.beans.factory.annotation.AutowiredAnnotationBeanPostProcessor - JSR-330 'javax.inject.Inject' annotation found and supported for autowiring
[사전 처리]getUser() 메소드 ARGS 정보 : UserVO [id=test, password=test123, name=null, role=null]
===> JDBC로 getUser() 기능 처리
관리자님 환영합니다.
INFO : org.springframework.context.support.GenericXmlApplicationContext - Closing org.springframework.context.support.GenericXmlApplicationContext@782830e: startup date [Sat Nov 16 22:33:30 KST 2019]; root of context hierarchy
```
   
***
# 3. AfterReturning 어드바이스 
AfterReturning은 비즈니스 메소드가 수행되고 나서, 결과 데이터를 리턴할 때 동작하는 어드바이스이다.       
**따라서 어떤 메소드가 어떤 값을 리턴했는지를 알아야 사후 처리 기능을 다양하게 구현할 수 있다.**       
    
**afterReturningLog**  
```
package com.springbook.biz.common;

import org.aspectj.lang.JoinPoint;

import com.springbook.biz.user.UserVO;

public class AfterReturningAdvice {
	public void afterReturningLog(JoinPoint jp, Object returnObj) {
		String method = jp.getSignature().getName();
		if(returnObj instanceof UserVO) {
			UserVO user = (UserVO) returnObj;
			if(user.getRole().equals("Admin")) {
				System.out.println(user.getName()+ "로그인(admin)");
			}
		}
		System.out.println("[사후 처리]"+ method + "() 메소드 리턴값 : "+returnObj.toString());
	}
}
```
```afterReturningLog()``` 메소드는 JoinPoint 객체를 첫 번째 매개변수로 선언했다.          
그리고 Object 타입의 변수도 두 번째 매개변수로 선언되어 있는데, 이를 **바인드 변수**라고 한다.           
          
**바인드 변수** :    
비즈니스 메소드가 리턴한 결괏값을 바인딩할 목적으로 사용, 어떤 값이 리턴될지 모르기에 ```Object``` 타입으로 선언한다.          
   
After Returning 어드바이스 메소드에 바인드 변수가 추가됐다면        
**반드시 바인드 변수에 대한 매핑 설정을 스프링 설정 파일에 추가해야 한다.**         
이때 ```<aop:after-returning>```엘리먼트의 ```returning=""``` 속성을 사용한다.       
   
**applicationContext.xml**
```
	<aop:config>
		<aop:pointcut expression="execution(* com.springbook.biz..*Impl.*(..))" id="allPointcut" />
		<aop:pointcut expression="execution(* com.springbook.biz..*Impl.get*(..))" id="getPointcut" />
		
		<aop:aspect ref="afterReturning"> 
			<aop:after-returning pointcut-ref="getPointcut" method="afterReturningLog" returning="returnObj" /> 
		</aop:aspect> 
	</aop:config>
```
위의 설정은 비즈니스 메소드가 리턴한 결괏값을 ```returnObj```라는 바인드 변수에 바인드하라는 설정이다.(인자)     
```AfterReturningAdvice``` 클래스의 ```afterReturningLog()``` 메소드는 해당 변수를 ```Object``` 객체로 받는다.   

**returning 속성은 ```<aop:after-returning> 엘리먼트에서만 사용할 수 있는 속성이며,```  
속성값은 반드시 어드바이스 메소드 매개변수로 선언된 바인드 변수 이름과 같아야 한다.**    
  
**결과**  
```
INFO : org.springframework.beans.factory.xml.XmlBeanDefinitionReader - Loading XML bean definitions from class path resource [applicationContext.xml]
INFO : org.springframework.context.support.GenericXmlApplicationContext - Refreshing org.springframework.context.support.GenericXmlApplicationContext@782830e: startup date [Sat Nov 16 22:31:30 KST 2019]; root of context hierarchy
INFO : org.springframework.beans.factory.annotation.AutowiredAnnotationBeanPostProcessor - JSR-330 'javax.inject.Inject' annotation found and supported for autowiring
===> JDBC로 getUser() 기능 처리
관리자로그인(admin)
[사후 처리]getUser() 메소드 리턴값 : UserVO [id=test, password=test123, name=관리자, role=Admin]
관리자님 환영합니다.
INFO : org.springframework.context.support.GenericXmlApplicationContext - Closing org.springframework.context.support.GenericXmlApplicationContext@782830e: startup date [Sat Nov 16 22:31:30 KST 2019]; root of context hierarchy
```
   
***
# 4. After Throwing 어드바이스  
After Throwing 은 ```BoardServiceClient```를 사용해서 확인할 것이다.
   
After Throwing은 비즈니스 메소드가 수행되다가 예외가 발생할 때 동작하는 어드바이스이다.    
**따라서 어떤 메소드에서 어떤 예외가 발생했는지를 알아야 한다.     
그래야 발생한 예외의 종류에 따라 정확한 예외 처리를 구현할 수 있을 것이다.**   
    
**AfterThrowingAdvice**  
```
package com.springbook.biz.common;

import org.aspectj.lang.JoinPoint;

public class AfterThrowingAdvice {
	public void execeptionLog(JoinPoint jp, Exception exceptObj) {
		String method = jp.getSignature().getName();
		System.out.println("[예외 처리]"+ method + "() 메소드 수행 중 발생된 예외 메시지 : " + exceptObj.getMessage());
	}
}
```  
```exceptionLog()``` 메소드는 ```JoinPoint``` 객체를 매개변수로 받는다.                  
그리고 비즈니스 메소드에서 발생한 예외 객체를 ```exceptObj``` 라는 바인드 변수를 통해 받는다.      
바인드 변수는 모든 예외 객체를 바인드할 수 있도록 예외 클래스의 최상위 타입인 ```Exception```으로 선언한다.        
       
어드바이스 메소드에 바인드 변수가 추가됐다면    
반드시 바인드 변수에 대한 매핑 설정을 스프링 설정 파일에 추가해야 한다.        
이때 ```<aop:after-throwing>```엘리먼트의 ```throwing=""``` 속성을 사용한다.     
  
**applicationContext.xml**  
```
	<context:component-scan base-package="com.springbook.biz"></context:component-scan>
	<bean id="afterThrowing" class="com.springbook.biz.common.AfterThrowingAdvice"></bean>
	<aop:config>
		<aop:pointcut expression="execution(* com.springbook.biz..*Impl.*(..))" id="allPointcut"/>
		<aop:aspect ref="afterThrowing">
			<aop:after-throwing pointcut-ref="allPointcut" method="execeptionLog" throwing="exceptObj"/>
		</aop:aspect>
	</aop:config>
```
throwing 속성은 ```<aop:after-throwing>```에서만 사용할 수 있는 속성이며     
**속성값은 어드바이스 메소드 매개변수로 선언된 바인드 변수 이름과 같아야 한다.**     
     
**BoardServiceImpl-예외상황 발생을 위한 클래스 수정**   
```
package com.springbook.biz.board.impl;

import java.util.List;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

import com.springbook.biz.BoardVO;
import com.springbook.biz.common.Log4jAdvice;
import com.springbook.biz.common.LogAdvice;

@Service("boardService")
public class BoardServiceImpl implements BoardService {
	@Autowired
	private BoardDAO boardDAO;

	@Override
	public void insertBoard(BoardVO vo) {
		if (vo.getSeq() == 0) {
			throw new IllegalArgumentException("0번 글은 등록할 수 없습니다.");
		}
		boardDAO.insertBoard(vo);
	}

	@Override
	public void updateBoard(BoardVO vo) {
		boardDAO.updateBoard(vo);
	}

	@Override
	public void deleteBoard(BoardVO vo) {
		boardDAO.deleteBoard(vo);
	}

	@Override
	public BoardVO getBoard(BoardVO vo) {
		return boardDAO.getBoard(vo);
	}

	@Override
	public List<BoardVO> getBoardList(BoardVO vo) {
		return boardDAO.getBoardList(vo);
	}

}
```
**결과**
```
INFO : org.springframework.beans.factory.xml.XmlBeanDefinitionReader - Loading XML bean definitions from class path resource [applicationContext.xml]
INFO : org.springframework.context.support.GenericXmlApplicationContext - Refreshing org.springframework.context.support.GenericXmlApplicationContext@782830e: startup date [Sat Nov 16 22:52:30 KST 2019]; root of context hierarchy
INFO : org.springframework.beans.factory.annotation.AutowiredAnnotationBeanPostProcessor - JSR-330 'javax.inject.Inject' annotation found and supported for autowiring
[예외 처리]insertBoard() 메소드 수행 중 발생된 예외 메시지 : 0번 글은 등록할 수 없습니다.
Exception in thread "main" java.lang.IllegalArgumentException: 0번 글은 등록할 수 없습니다.
	at com.springbook.biz.board.impl.BoardServiceImpl.insertBoard(BoardServiceImpl.java:20)
	at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
	at sun.reflect.NativeMethodAccessorImpl.invoke(Unknown Source)
	at sun.reflect.DelegatingMethodAccessorImpl.invoke(Unknown Source)
	at java.lang.reflect.Method.invoke(Unknown Source)
	at org.springframework.aop.support.AopUtils.invokeJoinpointUsingReflection(AopUtils.java:302)
	at org.springframework.aop.framework.ReflectiveMethodInvocation.invokeJoinpoint(ReflectiveMethodInvocation.java:190)
	at org.springframework.aop.framework.ReflectiveMethodInvocation.proceed(ReflectiveMethodInvocation.java:157)
	at org.springframework.aop.aspectj.AspectJAfterThrowingAdvice.invoke(AspectJAfterThrowingAdvice.java:58)
	at org.springframework.aop.framework.ReflectiveMethodInvocation.proceed(ReflectiveMethodInvocation.java:179)
	at org.springframework.aop.interceptor.ExposeInvocationInterceptor.invoke(ExposeInvocationInterceptor.java:92)
	at org.springframework.aop.framework.ReflectiveMethodInvocation.proceed(ReflectiveMethodInvocation.java:179)
	at org.springframework.aop.framework.JdkDynamicAopProxy.invoke(JdkDynamicAopProxy.java:208)
	at com.sun.proxy.$Proxy11.insertBoard(Unknown Source)
	at com.springbook.biz.board.impl.BoardServiceClient.main(BoardServiceClient.java:24)
```     
**AfterThrowingAdvice**
```
package com.springbook.biz.common;

import org.aspectj.lang.JoinPoint;

public class AfterThrowingAdvice {
	public void execeptionLog(JoinPoint jp, Exception exceptObj) {
		String method = jp.getSignature().getName();
		System.out.println(method + "() 메소드 수행 중 예외 발생!");
		
		if(exceptObj instanceof IllegalArgumentException) {
			System.out.println("부적합한 값이 입력되었습니다.");
		} else if(exceptObj instanceof NumberFormatException) {
			System.out.println("숫자 형식의 값이 아닙니다.");
		} else if(exceptObj instanceof Exception) {
			System.out.println("문제가 발생했습니다.");
		}
	}
}
```  
```AfterThrowingAdvice``` 어드바이스 클래스의 ```execeptionLog()``` 메소드를 구현할 때,     
발생하는 예외 객체의 종류에 따라 다양하게 예외처리를 진행할 수도 있다.   
  
**결과**
```
INFO : org.springframework.beans.factory.xml.XmlBeanDefinitionReader - Loading XML bean definitions from class path resource [applicationContext.xml]
INFO : org.springframework.context.support.GenericXmlApplicationContext - Refreshing org.springframework.context.support.GenericXmlApplicationContext@782830e: startup date [Sat Nov 16 22:57:25 KST 2019]; root of context hierarchy
INFO : org.springframework.beans.factory.annotation.AutowiredAnnotationBeanPostProcessor - JSR-330 'javax.inject.Inject' annotation found and supported for autowiring
insertBoard() 메소드 수행 중 예외 발생!
부적합한 값이 입력되었습니다.
Exception in thread "main" java.lang.IllegalArgumentException: 0번 글은 등록할 수 없습니다.
	at com.springbook.biz.board.impl.BoardServiceImpl.insertBoard(BoardServiceImpl.java:20)
	at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
	at sun.reflect.NativeMethodAccessorImpl.invoke(Unknown Source)
	at sun.reflect.DelegatingMethodAccessorImpl.invoke(Unknown Source)
	at java.lang.reflect.Method.invoke(Unknown Source)
	at org.springframework.aop.support.AopUtils.invokeJoinpointUsingReflection(AopUtils.java:302)
	at org.springframework.aop.framework.ReflectiveMethodInvocation.invokeJoinpoint(ReflectiveMethodInvocation.java:190)
	at org.springframework.aop.framework.ReflectiveMethodInvocation.proceed(ReflectiveMethodInvocation.java:157)
	at org.springframework.aop.aspectj.AspectJAfterThrowingAdvice.invoke(AspectJAfterThrowingAdvice.java:58)
	at org.springframework.aop.framework.ReflectiveMethodInvocation.proceed(ReflectiveMethodInvocation.java:179)
	at org.springframework.aop.interceptor.ExposeInvocationInterceptor.invoke(ExposeInvocationInterceptor.java:92)
	at org.springframework.aop.framework.ReflectiveMethodInvocation.proceed(ReflectiveMethodInvocation.java:179)
	at org.springframework.aop.framework.JdkDynamicAopProxy.invoke(JdkDynamicAopProxy.java:208)
	at com.sun.proxy.$Proxy11.insertBoard(Unknown Source)
	at com.springbook.biz.board.impl.BoardServiceClient.main(BoardServiceClient.java:24)
```
위 실습 까지 끝냈으면  ```BoardServiceImpl```의 ```insert()```는 다시 주석처리를 해주어야 한다.
```
@Override
public void insertBoard(BoardVO vo) {
	/*
	if (vo.getSeq() == 0) {
		throw new IllegalArgumentException("0번 글은 등록할 수 없습니다.");
	}
	 */
	boardDAO.insertBoard(vo);
}
```

***
# 5. Around 어드바이스  
```Around``` 어드바이스는 반드시 ```ProceedingJoinPoint```sms  객체를 매개변수로 받아야한다.     
```ProceedingJoinPoint```는 ```JoinPoint```를 상속했으며      
비즈니스 메소드를 호출하는 ```proceed()```메소드를 사용할 수 있다.    

**AroundAdvice**
```
package com.springbook.biz.common;

import org.aspectj.lang.ProceedingJoinPoint;
import org.springframework.util.StopWatch;

public class AroundAdvice {

	public Object aroundLog(ProceedingJoinPoint pjp) throws Throwable {
		String method = pjp.getSignature().getName();
		
		StopWatch stopWatch = new StopWatch();
		stopWatch.start();
		
		Object obj = pjp.proceed();
		
		stopWatch.stop();
		System.out.println(method+"() 메소드 수행에 걸린 시간 : "+stopWatch.getTotalTimeMillis()+"(ms)초");
		return obj;
	}
}
```
**결과**
```
INFO : org.springframework.beans.factory.xml.XmlBeanDefinitionReader - Loading XML bean definitions from class path resource [applicationContext.xml]
INFO : org.springframework.context.support.GenericXmlApplicationContext - Refreshing org.springframework.context.support.GenericXmlApplicationContext@782830e: startup date [Sat Nov 16 23:18:56 KST 2019]; root of context hierarchy
INFO : org.springframework.beans.factory.annotation.AutowiredAnnotationBeanPostProcessor - JSR-330 'javax.inject.Inject' annotation found and supported for autowiring
===> JDBC로 getUser() 기능 처리
getUser() 메소드 수행에 걸린 시간 : 165(ms)초
관리자님 환영합니다.
INFO : org.springframework.context.support.GenericXmlApplicationContext - Closing org.springframework.context.support.GenericXmlApplicationContext@782830e: startup date [Sat Nov 16 23:18:56 KST 2019]; root of context hierarchy
```
