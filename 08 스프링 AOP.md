08 스프링 AOP
=======================
비즈니스 컴포넌트 개발에서 가장 중요한 두가지 원칙은  
1. 낮은 결합도 (IoC)
2. 높은 응집도 (AOP)
를 유지하는 것이다.      
      
스프링의 **의존성 주입 IoC**을 이용하면     
비즈니스 컴포넌트가 구성하는 객체들의 결합도를 떨어뜨릴 수 있어서 의존 관계를 쉽게 변경할 수 있다.        
      
# 1. AOP 이해하기
엔터프라이즈 애플리케이션의 메소드들은 대부분 복잡한 코드들로 구성되어 있다.  
이 중에서 **핵심 비즈니스 로직은 몇 줄 안되고**  
주로 **로깅이나 예외, 트랜잭션 처리 같은 부가적인 코드가 대부분이다.**   
이런 부가적인 코드들로 인해서 비즈니스 메소드의 복잡도는 증가하고 결국 개발자를 지치게 한다.   
  
그렇다고 이러한 코드들을 삭제하거나 소홀히 해서는 절대 안 된다.     
중요한 점은 **비즈니스 메소드마다 이런 부가적인 코드들을 매번 반복해야 한다는 것이다.**   
**AOP는 이러한 부가적인 공통 코드들을 효율적으로 관리하는데 주목한다.**   
   
**AOP**를 이해하는데 가장 중요한 핵심 개념이 바로 **관심분리**이다.     
* 횡단 관심 : AOP에서 메소드마다 공통으로 등장하는 로깅이나 예외, 트랜잭션 처리 같은 코드들      
* 핵심 관심 : 사용자의 요청에 따라 실제로 수행되는 핵심 비즈니스 로직      
  
만약 이 두 관심을 완벽하게 분리할 수 있다면,  
구현하는 메소드에는 실제 비즈니스 로직만으로 구성할 수 있으므로 더욱 간결하고 응집도 높은 코드를 유지할 수 있다.     
하지만 기존의 OOP 언어에서는 횡단 관심에 해당하는 공통 코드를 완벽하게 독립적인 모듈로 분리해내기가 어렵다.         
     
**LogAdvice**
```
package com.springbook.biz.common;

public class LogAdvice {

	public void printLog() {
		System.out.println("[공통 로그] 비즈니스 로직 수행 전 동작");
	}
}
```
**BoardService**
```
package com.springbook.biz.board.impl;

import java.util.List;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

import com.springbook.biz.BoardVO;
import com.springbook.biz.common.LogAdvice;

@Service("boardService")
public class BoardServiceImpl implements BoardService {
	@Autowired
	private BoardDAO boardDAO;
	private LogAdvice log;
	
	public BoardServiceImpl() {
		log = new LogAdvice();
	}
	
	@Override
	public void insertBoard(BoardVO vo) {
		log.printLog();
		boardDAO.insertBoard(vo);
	}

	@Override
	public void updateBoard(BoardVO vo) {
		log.printLog();
		boardDAO.updateBoard(vo);
	}

	@Override
	public void deleteBoard(BoardVO vo) {
		log.printLog();
		boardDAO.deleteBoard(vo);
	}

	@Override
	public BoardVO getBoard(BoardVO vo) {
		log.printLog();
		return boardDAO.getBoard(vo);
	}

	@Override
	public List<BoardVO> getBoardList(BoardVO vo) {
		log.printLog();
		return boardDAO.getBoardList(vo);
	}
}
```

## 1.1. 소 주제
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
