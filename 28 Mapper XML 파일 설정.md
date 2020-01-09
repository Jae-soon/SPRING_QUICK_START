28 Mapper XML 파일 설정
=======================
# 1. SQL MapperXML 기본 설정  
## 1.1. Mybatis 구조  
  
[사진]  
  
우선 SqlMapConfig.xml 파일은 Mybatis 메인 환경 설정 파일이다. 
Mybatis는 이 파일을 읽어 들여 어떤 DBMS와 커넥션을 맺을지, 어떤 SQL Mapper XML 파일들이 등록되어 있는지 알 수 있다.     
   
Mybatis는 SqlMap.xml 파일에 등록된 각 SQL 명령어들을 Map 구조로 저장하여 관리한다.    
각 SQL 명령어는 고유한 아이디 값을 가지고 있으므로 특정 아이디로 등록된 SQL을 실행할 수 있다.    
그리고 SQL이 실행될 때 필요한 값들은 Input 형태의 데이터로 할당하고,    
실행된 SQL이 SELECT 구문일 때는 output 형태의 데이터로 리턴한다.  

## 1.2. Mapper XML 파일 구조  
Mybatis 프레임워크에서 가장 중요한 파일은 SQL 명령어들이 저장되는 SQL Mapper XML파일이다.   
SQL Mapper는 ```<mapper>```를 루트 엘리먼트로 가지는 XML 파일이다.  
     
[사진]      
      
Mapper 파일 구조를 살펴보면 가장 먼저 DTD 선언이 등장하고, 그 밑에 ```<mapper>``` 루트 엘리먼트가 선언된다.        
```<mapper>``` 엘리먼트는 namespace 속성을 가지는데, 이 네임스페이스를 이용하여 더 쉽게 유일한 SQL 아이디를 만들 수 있다.     
네임스페이스가 지정된 Mapper SQL을 DAO 클래스에서 참조할 때는 네임스페이스와 SQL의 아이디를 결합하여 참조해야 한다.      

**Mapper XML**
```
<mapper namespace="BoardDAO">
  <delete id="deleteBoard">
    delete board where seq=#{seq}
  </delete>
</mapper>
```
  
**DAO 클래스**
```
public void deleteBoard(BoardVO vo){
  mybatis.delete("BoardDAO.deleteBoard",vo)
}
여기서 BoardDAO는 현재 클래스를 의미하는 것이 아닌  
XML <mapper>의 네임스페이스를 의미 deleteBoard도 마찬가지  
```
Mapper 파일에 SQL 명령어들을 등록할 때는 SQL 구문의 종류에 따라 적절한 엘리먼트를 사용한다.  
INSERT 구문은 ```<insert>``` 엘리먼트를,  
SELECT 구문은 ```<select>``` 엘리먼트를 사용하는 식이다.  
이때, 각 엘리먼트에서 사용할 수 있는 속성들이 다르므로 그 의미와 용도를 이해해야 한다.     

## 1.3. ```<select>``` 엘리먼트  
```<select>``` 엘리먼트는 데이터를 조회하는 SELECT 구문을 작성할 때 사용한다.    
```<select>``` 엘리먼트에는 parameterType과 resultType 속성을 사용할 수 있다.    
  
**Mapper XML**
```
<mapper namespace="BoardDAO">
  <select id="getBoard" parameterType="board" resultType="board">
    select * from board where seq=#{seq}
  </select>
  <select id="getBoardList" parameterType="board" resultType="board">
    select * from board 
    where title like '%'||#{searchKeyword}||'%'
    order by seq desc
  </select>
</mapper>
```
  
### (1) id 속성  
```<select>``` 엘리먼트에 선언된 id 속성은 필수 속성으로,      
반드시 전체 Mapper 파일들 내에서 유일한 아이디를 등록해야 한다.       
       
그래야 나중에 DAO 클래스에서 특정 아이디로 등록된 SQL을 실행할 수 있다.   
이 id 속성과 관련하여 살펴볼 것이 루트 엘리먼트인 ```<mapper>```이다.    
   
```<mapper>``` 엘리먼트에 설정된 네임스페이스는  
```<mapper>``` 엘리먼트 안에서 선언된 여러 아이디를 하나의 네임스페이스로 묶을 수 있다.    
   
예를 들어,  
다음 두 파일에 선언된 ```getTotalCount```라는 아이디는 네임스페이스가 다르므로 다른 아이디로 처리될 수 있다.           
  
**board-mapping.xml**
```
<mapper namespace="BoardDAO">
  <select id="getTotalCount" resultType="int">
    select count(*) from board
  </select>
</mapper>
```
  
**user-mapping.xml**
```
<mapper namespace="UserDAO">
  <select id="getTotalCount" resultType="int">
    select count(*) from users
  </select>
</mapper>
```  
  
### (2) parameterType 속성  
Mapper 파일에 등록된 SQL을 실행하려면 SQL 실행에 필요한 데이터를 외부로부터 받아야한다.  
이때 사용하는 속성이 parameterType 속성이다.    
parameterType 속성값으로는 일반적으로 기본형이나 VO 형태의 클래스를 지정한다.    
```
<insert id="insertBoard" parameterType="com.springbook.biz.board.BoardVO">
  insert into board(seq, title, writer, content)
  values ((select nvl(max(seq), 0)+1 from board), #{title}, #{writer}, #{content})
</insert>
```
이때 Mybatis 메인 설정 파일(sql-map-config.xml)에 등록된      
```<typeAlias>```의 Alias를 사용하면 설정을 더 간결하게 처리할 수 있다.  
    
    
    
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
