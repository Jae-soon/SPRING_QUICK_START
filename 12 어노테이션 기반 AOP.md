12 어노테이션 기반 AOP
=======================
XML 기반 설정과 어노테이션 기반 설정을 적절히 혼합하여 사용하면 객체들을 효율적으로 관리할 수 있다.   
   
# 1. 어노테이션 기반 AOP 설정
## 1.1. 어노테이션 사용을 위한 스프링 설정 
AOP를 어노테이션으로 설정하려면 ```<aop:aspectj-autoproxy>```엘리먼트를 선언해야 한다.  
**applicationContext.xml**
```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xmlns:p="http://www.springframework.org/schema/p"
	xmlns:context="http://www.springframework.org/schema/context"
	xmlns:aop="http://www.springframework.org/schema/aop"
	xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
		http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context-4.2.xsd
		http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop-4.2.xsd">

	<context:component-scan base-package="com.springbook.biz">
  </context:component-scan>
	<aop:aspectj-autoproxy></aop:aspectj-autoproxy>
</beans>
```
```<aop:aspectj-autoproxy>```를 선언하면 스프링 컨테이너는 AOP 관련 어노테이션들을 인식하고 용도에 맞게 처리해준다.   

AOP 관련 어노테이션들은 어드바이스 클래스에 설정한다.   
그리고 어드바이스 클래스에 선언된 어노테이션들을 스프링 컨테이너가 처리하게 하려면,  
**반드시 어드바이스 객체가 생성되어 있어야 한다.**
   
그렇기에 어드바이스 클래스는 반드시 ```<bean>```이나 ```@Service```를 이용하여 객체를 등록해주어야 한다.     

## 1.2. 포인트컷 설정
어노테이션 설정으로도 포인트컷을 설정할 수 있다.
어노테이션 설정으로 포인트컷을 선언할 때는 ```@Pointcut```을 사용하며,   
하나의 어드바이스 클래스 안에 여러 개의 포인트컷을 선언할 수 있다.  
따라서 여러 포인트컷을 식별하기 위한 식별자가 필요한데, 이때 참조 메소드를 이용한다.   
  
참조 메소드는 메소드 몸체가 비어있는, 즉 구현 로직이 없는 메소드이다.    
따라서 어떤 기능 처리를 목적으로 하지 않고 단순히 포인트컷을 식별하는 이름으로만 사용된다.  
참고로 식별을 한다는 것을 해당 메소드를 기준으로 AOP 순서를 지정하게끔 한다는 것이다.

**LogAdvice**
```
package com.springbook.biz.common;

import org.aspectj.lang.JoinPoint;
import org.aspectj.lang.annotation.Pointcut;

public class LogAdvice {
	
	@Pointcut("execution(* com.springbook.biz..*Impl.*(..))")
	public void allPointcut() {}
	
	@Pointcut("execution(* com.springbook.biz..*Impl.get*(..))")
	public void getPointcut() {}
	
	public void printLog(JoinPoint jp) {
		System.out.println("[공통 로그] 비즈니스 로직 수행 전 동작");
	}
}
```
위 소스는 ```allPointcut()```과 ```getPointcut()``` 메소드 위에   
각각 ```@Pointcut```을 이용하여 두 개의 포인트컷을 선언했다.   
그러면 이후에 이 포인트컷을 참조할 때, ```@Pointcut```이 붙은 참조 메소드를 이용하여 특정 포인트컷을 지정할 수 있다.   


## 1.3. 어드바이스 설정
어드바이스 클래스에는 어드바이스 메소드가 구현되어 있다.(비즈니스 메소드에 따라 동작하는)     
우리는 이 어드바이스 메소드가 언제 동작할지 결정하여 관련된 어노테이션을 메소드위에 설정해주면 된다.      
어드바이스의 동작 시점 어노테이션은 XML 설정과 마찬가지로 5가지가 제공된다.  
   
이때 반드시 어드바이스 메소드가 결합될 포인트컷을 참조해야 한다.  
포인트컷을 참조하는 방법은 어드바이스 어노테이션 뒤에 괄호를 추가하고 포인트컷 참조 메소드를 지정하면 된다.   
   
**LogAdvice**
```
package com.springbook.biz.common;

import org.aspectj.lang.JoinPoint;
import org.aspectj.lang.annotation.Before;
import org.aspectj.lang.annotation.Pointcut;

public class LogAdvice {
	
	@Pointcut("execution(* com.springbook.biz..*Impl.*(..))")
	public void allPointCut() {}
	
	@Pointcut("execution(* com.springbook.biz..*Impl.get*(..))")
	public void getPointCut() {}
	
	@Before("allPointcut()")
	public void printLog(JoinPoint jp) {
		System.out.println("[공통 로그] 비즈니스 로직 수행 전 동작");
	}
}
```
위 설정은 ```allPointcut()``` 참조 메소드로 지정한 비즈니스 메소드가 호출될 때,  
어드바이스 메소드인 ```printLog()``` 메소드가 ```Before``` 형태로 동작하도록 설정한 것이다.  
하지만 위 코드도 아직 애스팩트 설정을 안해주었기에 완벽한 코드라고 할 수 없다.(위빙이 안된다.)   

**어드바이스 동작 시점 어노테이션**
```
@Before           : 비즈니스 메소드 실행 전에 동작  
@AfterReturning   : 비즈니스 메소드가 성공적으로 리턴되면 동작
@AfterThrowing    : 비즈니스 메소드 실행 중 예외가 발생하면 동작 
@After            : 비즈니스 메소드가 실행된 후, 무조건 실행
@Around           : 호출 자체를 가로채 비즈니스 메소드 실행 전후에 처리할 로직을 삽입할 수 있음
```

## 1.4. 애스팩트 설정
AOP 설정에서 가장 중요한 애스팩트는 ```@Aspect```를 이용하여 설정한다.    
애스팩트는 용어 정리 시간에 살펴봤듯이 포인트컷과 어드바이스의 결합이다.   
따라서 ```@Aspect```가 설정된 애스팩트 객체에는 반드시 포인트컷과 어드바이스를 결합하는 설정이 있어야 한다.  
   
**LogAdvice**
``` 

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
