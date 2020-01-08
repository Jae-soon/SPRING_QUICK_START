각각의 xml 정리
=======================
# 1. web.xml
**web.xml**
```
웹과 관련된 설정 (리스너, 어플리케이션 파라미터, 서블릿 설정, 필터 설정 등..)을 담고 있고 
tomcat은 web.xml 이지만 웹 컨테이너 마다 조금씩 이름은 다르다. 
``` 
기본적으로 서블릿 컨테이너와 연관된 xml 이므로 
1. 서블릿 객체 등록 ```<servlet>``` 
2. SpringMVC 적용시 DispatcherServlet 등록 
3  서블릿 또는 객체에 대한 url 매핑 
4. Spring 컨테이너 등록 가능 ```<context-param>```(근데 우리는 안 했었다)  
5. ```<listener>```로 우선 생성할 컨테이너 등록 가능

***
# 2. pom.xml
**pom.xml**
```
빌드/배포와 관련된 모든 설정을 담고 있고 
maven 이라는 유틸리티에서 메타 정보로 사용하는 설정파일 입니다.
``` 
스프링부트에서 gradle 사용하듯이 maven 으로 라이브러리 버전관리     
    
***
# 3. applicationContext.xml
우선 ```applicationContext.xml```은 ```web.xml``` 에서 ```<context-param>```을 이용해 생성하는 xml 이다.     
   
기존 web.xml 은 웹 어플리케이션에서 일반적으로 사용되는 큰 틀의 xml을 의미한다면   
applicationContext.xml 그중 스프링에 관련된 부분만 실행 시킬 수 있도록 따로 빼 놓은 xml이라 생각하면 된다.    
   
그리고 필수 사항은 아니어서 이름도 아무렇게나 지어도 되고 여러개 만들어도 된다.  

1. 의존성 주입을 위한 ```<bean>``` 객체 등록
2. AOP를 위한 어드바이스 객체 지정 및 포인트컷 지정
3. JDBC 관련 엘리먼트 사용 
4. 프로퍼티 읽어오기  

