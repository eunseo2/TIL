# Decorator Pattern
- 객체에 추가 요소(새로운 행동 or 기능)을 동적으로 더할 수 있는 패턴 
## 구조
<img width="558" alt="image" src="https://github.com/eunseo2/TILL/assets/70589857/d0d39bc4-3a22-41f6-b88c-2cd731c30c57">


https://refactoring.guru/ko/design-patterns/decorator


1. **Component; 모두의 슈퍼 클래스**
    
    컴포넌트는 Wrapper(Base Decorator)들과 래핑되는 객체(Concrete Components Of Component)들 모두에 대한 공통 인터페이스를 선언한다.
    
2. **Concrete Components; 구상 컴포넌트, 구현 컴포넌트**
    
    슈퍼 클래스의 구현체로, **기본 행동(Default 기능)을 구현한다.** 데코레이터 구현체에게 포장되는 객체(래핑되는 객체)를 말한다.
    
3. **Base Decorator; 기초 데코레이터**
    
    래핑되는 객체를 참조하기 위한 필드가 있다. 필드의 타입(Type)은 구상 컴포넌트와 구상 데코레이터를 모두 포함할 수 있는 Component 인터페이스 타입(다형성)으로 선언한다.
    
4. **Concrete Decorators; 구현 데코레이터**
    
    컴포넌트에 동적으로 추가될 수 있는 추가 행동(기능)들을 정의하는 클래스. 구현 데코레이터는 슈퍼 클래스의 추상 메서드(`execute()` )에 **추가 요소(확장을 위한 추가 기능 or 행동)**에 대한 부분을 정의한다.

## 코드
  
#### Notifier (Component)
```java
public abstract class Notifier {
	public abstract void sendMessage();
}
```
#### BaseNotifier (Concrete Components)
```java
public class BaseNotifier extends Notifier{
	@Override
	public void sendMessage() {
		System.out.println("기본 메세지 전송");
	}
}
```
#### BaseDecorator (Base Decorator)
```java
public abstract class BaseDecorator extends Notifier {
	private Notifier decoratedNotification;

	public BaseDecorator(Notifier decoratedNotification) {
		this.decoratedNotification = decoratedNotification;
	}

	@Override
	public void sendMessage() { decoratedNotification.sendMessage(); }
}
```
#### KakaoDecorator (Concrete Decorators)
```java
public class KakaoDecorator extends BaseDecorator {

	public KakaoDecorator(Notifier decoratedNotification) {
		super(decoratedNotification);
	}

	@Override
	public void sendMessage() {
		super.sendMessage();
		sendKakaoMessage();
	}

	private void sendKakaoMessage() { System.out.println("카카오 메세지 전송"); }
}
```
#### NaverDecorator (Concrete Decorators)
```java
public class NaverDecorator extends BaseDecorator {
	public NaverDecorator(Notifier decoratedNotification) {
		super(decoratedNotification);
	}

	@Override
	public void sendMessage() {
		super.sendMessage();
		sendNaverMessage();
	}

	private void sendNaverMessage() { System.out.println("네이버 메세지 전송"); }
}

```

### 참고자료
https://refactoring.guru/ko/design-patterns/decorator

https://i-am-your-father.notion.site/Gang-of-Four-4e354fd5da844a56ace1e8454f39894b#95cb881f21cb4285b5b4ed0f1887b45f
