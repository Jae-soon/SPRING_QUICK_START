04 IoC 의존성 주입
=======================
# 1. 의존성 관리    
## 1.1. 스프링의 의존성 관리 방법      
스프링 프레임워크는 **객체의 생성과 의존관계를 컨테이너가 자동으로 관리한다는 점이다.**      
이것이 바로 스프링 IoC의 핵심 원리이기도 하다.        
   
스프링 IoC를 다음 두 가지 형태로 지원한다.  
1. Dependency Lookup 
2. Dependency injection     
    
![KakaoTalk_20191028_091834174](https://user-images.githubusercontent.com/50267433/67644353-2d166900-f964-11e9-8029-c5d2210a7f84.jpg)
     
**Dependency Lookup :**     
컨테이너가 애플리케이션 운용에 필요한 객체를 생성하고            
클라이언트는 컨테이너가 생성한 객체를 검색하여 사용하는 방식              
          
**Dependency Injection :**          
객체 사이의 의존관계를 스프링 설정 파일에 등록된 정보를 바탕으로 컨테이너가 자동으로 처리해 준다.              
따라서 의존성 설정을 바꾸고 싶을 때 프로그램 코드를 수정하지 않고 스프링 설정 파일 수정만으로                 
변경사항을 적용할 수 있어서 유지보수가 항상된다.         
Setter 메소드를 기반으로 하는 세터 인젝션과 생성자를 기반으로 하는 생성자 인젝션으로 나뉜다.               
    
## 1.2. 의존성 관계
의존성 관계란 객체와 객체의 결합 관계이다.     
즉 하나의 객체에서 다른 객체의 변수나 메소드를 사용해야 한다면   
이용하려는 객체에 대한 객체 생성과 생성된 객체의 레퍼런스 정보가 필요하다.     
   
**SonySpeaker**
```
package polymorphism;

public class SonySpeaker {
	public SonySpeaker() {
		System.out.println("===> SonySpeaker 객체 생성");
	}
	public void volumeUp() {
		System.out.println("SonySpeaker---소리 올린다.");
	}
	public void volumeDown() {
		System.out.println("SonySpeaker---소리 내린다.");
	}
}
```
**SamsungTV**
```
package polymorphism;

public class SamsungTV implements TV {
	private SonySpeaker speaker;

	public void initMethod() {
		System.out.println("객체 초기화 작업 처리..");
	}

	public void destroyMethod() {
		System.out.println("객체 삭제 전에 처리할 로직 처리...");
	}

	public SamsungTV() {
		System.out.println("===> SamsungTV 객체 생성");
	}

	@Override
	public void powerOn() {
		System.out.println("SamsungTV---전원 켠다.");
	}

	@Override
	public void powerOff() {
		System.out.println("SamsungTV---전원 끈다.");
	}

	@Override
	public void volumeUp() {
		speaker = new SonySpeaker();
		speaker.volumeUp();
	}

	@Override
	public void volumeDown() {
		speaker = new SonySpeaker();
		speaker.volumeDown();
	}
}
```
하지만 이 프로그램에는 2가지 문제가 있다.     
1. volume관련 메소드를 각각 호출하면 SonySpeaker 객체가 쓸데 없이 2개나 생성된다.      
2. 운영 과정에서 SonySpeaker의 성능이 떨어져서 AppleSpeaker 와 같이 다른 Speaker로 변경하고자 할 때    
앞서 보았던 식별자가 다름으로 생기는 전체 코드 수정 및 유지보수의 어려움       
     
이러한 이유는 **의존 관계에 있는 Speaker 객체에 대한 객체 생성 코드를 직접 명시했기 때문이다.**     
스프링은 이 문제를 스프링 컨테이너에 **의존성 주입**을 이용하여 해결한다.         
       
***    
# 2. 생성자 인젝션 이용하기  
스프링 컨테이너는 XML 설정 파일에 등록된 클래스를 찾아서 객체 생성할 때 기본적으로 매개변수가 없는 기본 생성자를 호출한다.   
하지만 **컨테이너가 기본 생성자 말고 매개변수를 가지는 다른 생성자를 호출하도록 설정할 수 있는데,  
이 기능을 이용하여 생성자 인젝션을 처리한다.**  
(즉, 여러 오버로딩된 생성자 중에 매개변수 없는 것을 기본으로 사용했지만 매개변수 있는것을 사용해 인젝션을 처리)    

생성자 인젝션을 사용하면 생성자의 매개변수로 의존관계에 있는 개체의 주소 정보를 전달할 수 있다.
(매개변수로 해당 클래스와 연관이 있는 인스턴스를 넣어 줄 수도 있다는 뜻)  

**생성자 추가**    
```
package polymorphism;

public class SamsungTV implements TV {
	private SonySpeaker speaker;

	public void initMethod() {
		System.out.println("객체 초기화 작업 처리..");
	}

	public void destroyMethod() {
		System.out.println("객체 삭제 전에 처리할 로직 처리...");
	}

	public SamsungTV() {
		System.out.println("===> SamsungTV 객체 생성");
	}
	
	public SamsungTV(SonySpeaker speaker) {
		System.out.println("===> SamsungTV(2) 객체 생성");
		this.speaker = speaker;
	}

	@Override
	public void powerOn() {
		System.out.println("SamsungTV---전원 켠다.");
	}

	@Override
	public void powerOff() {
		System.out.println("SamsungTV---전원 끈다.");
	}

	@Override
	public void volumeUp() {
		speaker.volumeUp();
	}

	@Override
	public void volumeDown() {
		speaker.volumeDown();
	}

}
```
**applicationContext.xml 설정**
```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
	
	<!-- 클래스 지정시에 bean 태그를 사용한다. -->
	<bean id="tv" class="polymorphism.SamsungTV" init-method="initMethod" destroy-method="destroyMethod" lazy-init="true">
		<constructor-arg ref="sony"></constructor-arg> <!-- sony 는 id 가 sony인 bean을 의미 -->
	</bean>
	
	<bean id="sony" class="polymorphism.SonySpeaker"></bean>
</beans>
```
생성자 인젝션을 위해서는 기존 클래스 ```<bean>``` 등록 설정에서      
시작 태그와 종료 태그 사이에 ```<constructor-arg>```엘리먼트를 추가하면 된다. (즉,bean 객체의 어떤 생성자 사용 지정)    
그리고 생성자 인자로 전달할 객체의 아이디를 ```<constructor-arg>```엘리먼트에 **ref속성**으로 참조한다.    
   
**실행 결과**
```
INFO : org.springframework.beans.factory.xml.XmlBeanDefinitionReader - Loading XML bean definitions from class path resource [applicationContext.xml]
INFO : org.springframework.context.support.GenericXmlApplicationContext - Refreshing org.springframework.context.support.GenericXmlApplicationContext@39ed3c8d: startup date [Mon Oct 28 10:15:25 KST 2019]; root of context hierarchy
===> SonySpeaker 객체 생성
===> SamsungTV(2) 객체 생성
객체 초기화 작업 처리..
SamsungTV---전원 켠다.
SonySpeaker---소리 올린다.
SonySpeaker---소리 내린다.
SamsungTV---전원 끈다.
INFO : org.springframework.context.support.GenericXmlApplicationContext - Closing org.springframework.context.support.GenericXmlApplicationContext@39ed3c8d: startup date [Mon Oct 28 10:15:25 KST 2019]; root of context hierarchy
객체 삭제 전에 처리할 로직 처리...
```    
실행결과로 알 수 있는 사실은 2가지이다.    
1. SamsungTV 클래스 객체가 생성될 때, 기본 생성자가 아닌 두 번째 생성자가 사용되었다는 점    
2. 스프링 설정 파일에 SonySpeaker 가 SamsungTV 클래스 밑에 등록 되었는데도 먼저 생성되고 있다는 점   
      
스프링 컨테이너는 기본적으론 bean 등록된 순서대로 객체를 생성하며, 모든 객체는 기본 생성자 호출을 원칙으로 한다.     
하지만 생성자 인잭션으로 주입될 경우 이러한 원칙이 적용되지 않는다.   
     
## 2.1. 다중 변수 매핑(매개변수 여러개)   
생성자 인젝션에서 초기해야할 맴버 변수가 여러 개이면, 여러 개의 값을 한꺼번에 전달해야 한다.    
**SamsungTV**
```
package polymorphism;

public class SamsungTV implements TV {
	private SonySpeaker speaker;
	private int price ;
	
	public void initMethod() {
		System.out.println("객체 초기화 작업 처리..");
	}

	public void destroyMethod() {
		System.out.println("객체 삭제 전에 처리할 로직 처리...");
	}

	public SamsungTV() {
		System.out.println("===> SamsungTV 객체 생성");
	}
	
	public SamsungTV(SonySpeaker speaker) {
		System.out.println("===> SamsungTV(2) 객체 생성");
		this.speaker = speaker;
	}
	
	public SamsungTV(SonySpeaker speaker, int price) {
		System.out.println("===> SamsungTV(3) 객체 생성");
		this.speaker = speaker;
		this.price = price;
	}
	
	@Override
	public void powerOn() {
		System.out.println("SamsungTV---전원 켠다. (가격 : "+ price +")");
	}

	@Override
	public void powerOff() {
		System.out.println("SamsungTV---전원 끈다.");
	}

	@Override
	public void volumeUp() {
		speaker.volumeUp();
	}

	@Override
	public void volumeDown() {
		speaker.volumeDown();
	}
}
```
**applicationContext.xml**  
```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
	
	<!-- 클래스 지정시에 bean 태그를 사용한다. -->
	<bean id="tv" class="polymorphism.SamsungTV" init-method="initMethod" destroy-method="destroyMethod" lazy-init="true">
		<constructor-arg ref="sony"></constructor-arg> <!-- sony 는 id 가 sony인 bean을 의미 -->
		<constructor-arg value="270000"></constructor-arg> <!-- sony 는 id 가 sony인 bean을 의미 -->
	</bean>
	<bean id="sony" class="polymorphism.SonySpeaker"></bean>
</beans>
```
**결과**
```
INFO : org.springframework.beans.factory.xml.XmlBeanDefinitionReader - Loading XML bean definitions from class path resource [applicationContext.xml]
INFO : org.springframework.context.support.GenericXmlApplicationContext - Refreshing org.springframework.context.support.GenericXmlApplicationContext@39ed3c8d: startup date [Mon Oct 28 10:50:22 KST 2019]; root of context hierarchy
===> SonySpeaker 객체 생성
===> SamsungTV(3) 객체 생성
객체 초기화 작업 처리..
SamsungTV---전원 켠다. (가격 : 270000)
SonySpeaker---소리 올린다.
SonySpeaker---소리 내린다.
SamsungTV---전원 끈다.
INFO : org.springframework.context.support.GenericXmlApplicationContext - Closing org.springframework.context.support.GenericXmlApplicationContext@39ed3c8d: startup date [Mon Oct 28 10:50:22 KST 2019]; root of context hierarchy
객체 삭제 전에 처리할 로직 처리...
```
**ref 와 value**   
```<constructor-arg>```엘리먼트에는 ```ref```와 ```value``` 속성을 사용하여 생성자 매개변수로 전달할 값을 지정할 수 있다.     
   
* ref 속성 : 인자로 전달될 데이터가 ```<bean>```으로 등록된 다른 객체(아이디나 이름을 참조)     
* value 속성 : 고정된 문자열이나 정수 같은 기본형 데이터일 때 사용 (리터럴 값 사용)     
 
그런데 생성자가 여러 개 오버로딩 되어 있다면 어떤 생성자를 호출해야 할지 분명하지 않을 수 있으니    
**```index``` 속성을 지원하며, ```index``` 속성을 이용하면 어떤 값이 몇 번째 매개변수로 매핑되는지 지정할 수 있다.**  
index는 0부터 시작한다. (index=0)          
**applicationContext.xml**    
```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
	
	<!-- 클래스 지정시에 bean 태그를 사용한다. -->
	<bean id="tv" class="polymorphism.SamsungTV" init-method="initMethod" destroy-method="destroyMethod" lazy-init="true">
		<constructor-arg index="0" ref="sony"></constructor-arg> <!-- sony 는 id 가 sony인 bean을 의미 -->
		<constructor-arg index="1" value="270000"></constructor-arg> <!-- sony 는 id 가 sony인 bean을 의미 -->		
	</bean>
	<bean id="sony" class="polymorphism.SonySpeaker"></bean>
</beans>
```  
## 2.2. 의존관계 변경(유지보수)   
SamsungTV는 유지보수 과정에서 다른 스피커로 교체하는 상황도 발생할 것이다.    
의존성 주입은 이런 상황을 매우 효과적으로 처리해준다. 
**Speaker.interface**
```
package polymorphism;

public interface Speaker {
	void volumeUp();
	void volumeDown();
}
```
**AppleSpeaker**
```
package polymorphism;

public class AppleSpeaker implements Speaker {

	public  AppleSpeaker() {
		System.out.println("===> AppleSpeaker 객체 생성");
	}
		
	@Override
	public void volumeUp() {
		System.out.println("AppleSpeaker---소리 올린다.");
	}
	@Override
	public void volumeDown() {
		System.out.println("AppleSpeaker---소리 내린다.");
		
	}
}
```
**SonySpeaker**
```
package polymorphism;

public class SonySpeaker implements Speaker {
	public  SonySpeaker() {
		System.out.println("===> SonySpeaker 객체 생성");
	}
		
	@Override
	public void volumeUp() {
		System.out.println("SonySpeaker---소리 올린다.");
	}
	@Override
	public void volumeDown() {
		System.out.println("SonySpeaker---소리 내린다.");
		
	}
}
```
**SamsungTV**
```
package polymorphism;

public class SamsungTV implements TV {
	private Speaker speaker;
	private int price ;
	
	public void initMethod() {
		System.out.println("객체 초기화 작업 처리..");
	}

	public void destroyMethod() {
		System.out.println("객체 삭제 전에 처리할 로직 처리...");
	}

	public SamsungTV() {
		System.out.println("===> SamsungTV 객체 생성");
	}
	
	public SamsungTV(Speaker speaker) {
		System.out.println("===> SamsungTV(2) 객체 생성");
		this.speaker = speaker;
	}
	
	public SamsungTV(Speaker speaker, int price) {
		System.out.println("===> SamsungTV(3) 객체 생성");
		this.speaker = speaker;
		this.price = price;
	}
	@Override
	public void powerOn() {
		System.out.println("SamsungTV---전원 켠다. (가격 : "+ price +")");
	}

	@Override
	public void powerOff() {
		System.out.println("SamsungTV---전원 끈다.");
	}

	@Override
	public void volumeUp() {
		speaker.volumeUp();
	}

	@Override
	public void volumeDown() {
		speaker.volumeDown();
	}

}
```
**applicationContext.xml**
```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
	
	<!-- 클래스 지정시에 bean 태그를 사용한다. -->
	<bean id="tv" class="polymorphism.SamsungTV" init-method="initMethod" destroy-method="destroyMethod" lazy-init="true">
		<constructor-arg ref="apple"></constructor-arg> <!-- sony 는 id 가 sony인 bean을 의미 -->
		<constructor-arg value="270000"></constructor-arg> <!-- sony 는 id 가 sony인 bean을 의미 -->
		
	</bean>
	
	<bean id="sony" class="polymorphism.SonySpeaker"></bean>
	<bean id="apple" class="polymorphism.AppleSpeaker"></bean>

</beans>
```
결국 스프링 설정 파일만 적절히 관리하면 동작하는 TV도 변경할 수 있고,  
TV가 사용하는 스피커도 변경할 수 있다.  
여기에서 핵심은 이 과정에서 어떤 자바 코드도 변경하지 않는다는 것이다.  
