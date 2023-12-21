# AOP
- AOP는 Aspect Oriented Programming의 약자로 관점 지향 프로그래밍이라고 불린다. 
- 관점 지향은 쉽게 말해 어떤 로직을 기준으로 핵심적인 관점, 부가적인 관점으로 나누어서 보고 그 관점을 기준으로 각각 모듈화하겠다는 것이다.
- 여기서 모듈화란 **어떤 공통된 로직이나 기능을 하나의 단위로 묶는 것**을 말한다. 
- AOP는 로깅, 트랜잭션, 인증처리에서 많이 사용되고 있다. 


## AOP 로깅 적용
- AOP를 이용하여 API가 호출될때, REQUEST : 클래스이름.메서드이름 로그를 찍고 메서드가 실행된 이후 "RESPONSE : 클래스이름.메서드이름 = result : 메서드결과 (실행시간ms)" 로그를 기록하도록 처리하였다. 


```java
import org.aspectj.lang.ProceedingJoinPoint;
import org.aspectj.lang.annotation.Around;
import org.aspectj.lang.annotation.Aspect;
import org.springframework.stereotype.Component;

import lombok.extern.slf4j.Slf4j;

@Slf4j
@Component
@Aspect
public class Logging {
	@Around("within(com.example.study..*)")  // study 패키지에 포함된 포인트컷의 전/후에 필요한 동작을 추가함
	public Object logging(ProceedingJoinPoint proceedingJoinPoint) throws Throwable {
		long startAt = System.currentTimeMillis();

		log.info("REQUEST : {}.{}",
			proceedingJoinPoint.getSignature().getDeclaringTypeName(),
			proceedingJoinPoint.getSignature().getName()
		);

		Object result = proceedingJoinPoint.proceed();

		long endAt = System.currentTimeMillis();
		log.info("RESPONSE : {}.{} = result : {} ({}ms)",
			proceedingJoinPoint.getSignature().getDeclaringTypeName(),
			proceedingJoinPoint.getSignature().getName(), result, endAt - startAt
		);

		return result;
	}
}
```


## 테스트 결과 

![image](https://github.com/eunseo2/TIL/assets/70589857/775a50de-abb8-4be5-92ce-a8ac63160342)


### 참고자료
https://docs.spring.io/spring-framework/reference/core/aop.html
https://engkimbs.tistory.com/entry/스프링AOP
