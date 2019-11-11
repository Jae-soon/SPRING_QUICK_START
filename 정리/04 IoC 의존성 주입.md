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
1. volume관련 메소드를 각각 호출하면 SonySpeaker 객체가 쓸데 없이 두개나 생성된다.      
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
