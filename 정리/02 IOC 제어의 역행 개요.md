IOC 제어의 역행 개요
=======================
# 1. 결합도    
제어의 역행 개념을 얻기 위해 결합도에 대한 이해를 먼저하자     
    
**SamsungTV**
```
package polymorphism;

public class SamsungTV {
	public void powerOn() {
		System.out.println("SamsungTV---전원 켠다.");
	}
	public void powerOff() {
		System.out.println("SamsungTV---전원 끈다.");
	}
	public void volumeUp() {
		System.out.println("SamsungTV---소리 올린다.");
	}
	public void volumeDown() {
		System.out.println("SamsungTV---소리 내린다.");
	}
}
```
**LgTV**
```
package polymorphism;

public class LgTV {
	public void turnOn() {
		System.out.println("LgTV---전원 켠다.");
	}
	public void turnOff() {
		System.out.println("LgTV---전원 끈다.");
	}
	public void soundUp() {
		System.out.println("LgTV---소리 올린다.");
	}
	public void soundDown() {
		System.out.println("LgTV---소리 내린다.");
	}
}
```
**TVUSer**
```
package polymorphism;

public class TVUser {

	public static void main(String[] args) {
		// TODO Auto-generated method stub
		SamsungTV tv = new SamsungTV();
		tv.powerOn();
		tv.volumeUp();
		tv.volumeDown();
		tv.powerOff();
		////////////////아래와 같이 수정////////////////
		LgTV tv2 = new LgTV();
		tv2.turnOn();
		tv2.soundUp();
		tv2.soundDown();
		tv2.turnOff();
	}
}
```
SamsungTV와 LgTv가 존재한다고 가정시에 TVUser는 둘중의 하나의 Tv만을 사용할 것이다.    
처음에 SamsungTV를 잘 사용하고 있다가 문제가 생겨 LgTV로 바꿔야 된다고 가정을 해보면 
실제 동작하는 기능은 갖지만 전체 소스코드를 수정해야한다.(클래스명,메소드이름)   
**그리고 이러한 클래스와 연관된 파일이 수백개가 넘는다면?**        
     
이렇게 한 모듈과 다른 모듈간의 상호의존도 또는 각 모듈 사이의 연관도의 관계를 나타내는 것을 **결합도**라 부르며,   
결합도가 높을 수록 유지보수 해야할 코드와 시간이 많아지는 것을 의미하므로 나쁜 특성이라 할 수 있다.         

## 1.1. 결합도 해결하기-1 다형성 이용하기(인터페이스)      
  
![KakaoTalk_20191027_190652982](https://user-images.githubusercontent.com/50267433/67632948-23055380-f8ed-11e9-9a59-7c090bcd6ce1.jpg)
  
상속과 인터페이스는 공통의 규약을 통해 보다 일관되게 코드를 구성할 수 있다.      
결합도를 낮추기 위해서 가장 쉽게 생각할 수 있는 것이 객체지향 언어의 핵심 개념인 다형성을 이용하는 것이다.     
다형성을 이용하려면 상속과 메소드 재정의 그리고 형변환이 필요하며, 객체지향 언어는 이를 문법으로 지원한다.     
   
**TV.interface**
```
package polymorphism;

public interface TV {
	public void powerOn();
	public void powerOff();
	public void volumeUp();
	public void volumeDown();
}
```
**SamsungTV**
```
package polymorphism;

public class SamsungTV implements TV {
	
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
		System.out.println("SamsungTV---소리 올린다.");
	}
	@Override
	public void volumeDown() {
		System.out.println("SamsungTV---소리 내린다.");		
	}
  
}
```
**LgTV**
```
package polymorphism;

public class LgTV implements TV{

	@Override
	public void powerOn() {
		System.out.println("LgTV---전원 켠다.");		
	}
	@Override
	public void powerOff() {
		System.out.println("LgTV---전원 끈다.");
	}
	@Override
	public void volumeUp() {
		System.out.println("LgTV---소리 올린다.");
	}
	@Override
	public void volumeDown() {
		System.out.println("LgTV---소리 내린다.");		
	}
  
}
```
**TVUser**
```
package polymorphism;

public class TVUser {

	public static void main(String[] args) {
		// TODO Auto-generated method stub
		TV tv = new SamsungTV();
		tv.powerOn();
		tv.volumeUp();
		tv.volumeDown();
		tv.powerOff();
		
		//////////////////////////////////////
		
		TV tv2 = new LgTV();
		tv2.powerOn();
		tv2.volumeUp();
		tv2.volumeDown();
		tv2.powerOff();
	}
}
```
이렇게 다형성을 이용하면 TVUser와 같은 클라이언트 프로그램이 여러 개 있더라도 최소한의 수정으로 TV를 교체할 수 있다.   
따라서 유지보수는 좀 더 편해졌다고 할 수 있다. (이러한 경우는 메소드 시그니처에 대한 통일성을 도와준다고 볼 수 있다.)       

## 1.2. 결합도 해결하기-1 디자인 패턴 이용하기      
다형성을 사용하더라도 TV를 변경하고자 할 때, TV 클래스 객체를 생성하는 소스를 수정해야만 한다.    
```
TV tv = new SamsungTV(); -> new LgTV();
```
**마찬가지로 이러한 코드를 가진 파일이 100개나 된다면?**      
우리는 100개의 파일의 소스코드를 전부 찾아서 이를 수정해야 할 것이다.     

TV를 교체할 때,  
클라이언트 소스를 수정하지 않고 TV를 교체할 수 만 있다면 유지보수는 더욱 관리해질 것이다.     
이를 위해서 Factory 패턴을 사용해야 하는데,     
**Factory 패턴은 클라이언트에서 사용할 객체 생성을 캡슐화하여 TVUser와 TV 사이를 느슨한 결합 상태로 만들어준다.**    
     
즉, 쉽게 말하면 소스파일 위에 이를 관리하는 파일이 있다면        
100개 파일의 소스코드를 수정하는 것이 아닌 위에서 관리하는 파일의 소스코드만 수정하면 된다.      
   
**BeanFactory**
```
package polymorphism;

public class BeanFactory {
	public Object getBean(String beanName) {
		if (beanName.equals("samsung")) {
			return new SamsungTV();
		}
		if (beanName.equals("lg")) {
			return new LgTV();
		}
		return null;
	}
}
```
**TVUser**
```
package polymorphism;

public class TVUser {

	public static void main(String[] args) {
		BeanFactory factory = new BeanFactory();
		TV tv = (TV)factory.getBean(args[0]);
		tv.powerOn();
		tv.volumeUp();
		tv.volumeDown();
		tv.powerOff();
	}
}
```
![KakaoTalk_20191027_193230236](https://user-images.githubusercontent.com/50267433/67633231-a5434700-f8f0-11e9-8cc6-6436558557bc.jpg)  
    
참고로 ```Object``` 에서 ```TV```로 형변환이 가능했다는 것이다.      
이는 단순히 클래스 관점에서 보면 불가능 하지만 참조변수가 참조하는 대상은 **TV를 구현한 인스턴스**이다.     
그렇기에 그 과정에서 Object로 참조했다가 TV로 다시 참조를 해도 기본 인스턴스는 그대로이기에 가능한 것이다.   
   
클라이언트에 해당하는 TVUser는 자신이 필요한 객체를 직접 생성하지 않는다.  
결국 클라이언트는 소스를 수정하지 않고도 실행되는 객체를 변경할 수 있다. (BeanFactory에서)    
