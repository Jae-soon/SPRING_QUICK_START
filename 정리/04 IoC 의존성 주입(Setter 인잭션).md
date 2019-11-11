# 1. Setter 인젝션 이용하기  
   
생성자 인젝션(```<constructor-arg>```)은 생성자를 이용하여 의존성을 처리한다.      
**Setter 인젝션은 Setter 메소드를 호출하여 의존성 주입을 처리하는 방법이다.**     
      
두 가지 방법 모두 멤버변수를 원하는 값으로 설정하는 것을 목적으로 하고 있고,     
결과가 같으므로 둘 중 어떤 방법을 쓰든 상관없지만     
코딩 컨벤션(관습)에 따라 한 가지로 통일해서 사용하는데 대부분은 Setter 인젝션을 사용하며,     
Setter 메소드가 제공되지 않는 클래스에 대해서만 생성자 인젝션을 사용한다.          

**즉, 우리가 자주 사용할 것은 생성자 인젝션이니 잘 외우도록 하자.**   
  
## 1.1. Setter 인젝션 기본
Setter 메소드는 스프링 컨테이너가 자동으로 호출하며, 호출하는 시점은 ```<bean>```객체 생성 직후이다.         
따라서 **Setter 인젝션이 동작하려면 Setter 메소드뿐만 아니라 기본 생성자도 반드시 필요하다.**             
   
**Setter 인젝션은 ```<property>``` 엘리먼트를 사용한다.**     
```<property>```의 ```name``` 속성 값이 호출하고자 하는 메소드 이름이다.    
```name``` 속성값이 ```"speaker"```라고 설정 되어 있으면 호출되는 메소드는 ```setSpeaker()```이다.       
여기서 중요한 점은 **name 속성의 값이 메소드의 이름과 꼭 동일해야 한다.**     
   

  

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
	
	public void setSpeaker(Speaker speaker) {
		System.out.println("===> setSpeaker() 호출");
		this.speaker = speaker;
	}

	public void setPrice(int price) {
		System.out.println("===> setPrice() 호출");
		this.price = price;
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
	  <!-- 객체를 생성했고 거기서 Setter 메소드를 호출 이때 중요점이 이름이 같아야한다. -->
    <!-- ref는 앞서 똑같이 전달할 객체의 id 를 넣고 ->
    <!-- value도 앞서 똑같이 전달할 리터럴의 값을 기술해주면 된다, ->  
    <property name="speaker" ref="apple"></property>
		<property name="price" value="2700000"></property>
	</bean>

	<bean id="sony" class="polymorphism.SonySpeaker"></bean>
	<bean id="apple" class="polymorphism.AppleSpeaker"></bean>
		
</beans>
```
