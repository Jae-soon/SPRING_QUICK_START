Model
View
Controller

url에 따라 처리를 다르게 해주는 DispatcherServlet이 존재 (이때는 하나의 Cotroller라 볼 수있다.)    
그런데 통째로 하려다 보니 크기가 커지면 하나의 클래스가 처리하기에 너무 무거워져서 이것을 3가지로 나눈 것  

즉 
```
if(path == "/login.do"){
  DAO 처리
}
return 이동할 페이지
```
이와 같은 코드가 100개가 넘어갈 수 도 있는다는 것인데     
이를 Controller 인터페이스를 두어서 코드를 간략하게 나타냈고   
단순히 Controller 만 구현하기에는 부족한 점이 많아서     
  
if(path == "/login.do")와 같이 url에 맞게 사용할 Controller를 다르게 반환해주는 handlerMapping    
해당 경로에 맞게 DAO 처리를 해주는 Controller    
Controller로 부터 반환된 부족한 경로를 완벽한 경로로 바꿔주는 Resolver 가 등장하게 되었다.    





