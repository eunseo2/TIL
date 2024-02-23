# Observer Pattern
- 자신이 관찰 중인 객체에 발생하는 모든 이벤트에 대하여 구독 메커니즘을 정의할 수 있도록 하는 행동 디자인 패턴


## 구조
![image](https://github.com/eunseo2/TILL/assets/70589857/7b095062-bd2b-4d86-a41b-77a6e56275d6)

https://www.scaler.com/topics/design-patterns/observer-design-pattern/

### Subject || Observable || Publisher; 발행자

객체의 **`상태 변경 여부` || `상태 변경에 대한 메시지`**를 발행하는 주체

### Observer || Subscriber; 구독자

객체의 **`상태 변경에 대한 자세한 정보를 전달 받는 구독자`** || **`상태 변경에 대한 정보가 담겨 있는 메시지를 구독`**


### 실행
ApplicationEventPublisher 활용
- Spring의 ApplicationContext가 상속하는 인터페이스중 하나
- 디자인 패턴중 하나인 옵저버 패턴(Observer Pattern)의 구현체

  
#### Event
```java
public class CustomSpringEvent extends ApplicationEvent {
    private String message;

    public CustomSpringEvent(Object source, String message) {
        super(source);
        this.message = message;
    }
    public String getMessage() {
        return message;
    }
}
```

#### Publisher
```java
@Component
public class CustomSpringEventPublisher {
    @Autowired
    private ApplicationEventPublisher applicationEventPublisher;

    public void publishCustomEvent(final String message) {
        System.out.println("Publishing custom event. ");
        CustomSpringEvent customSpringEvent = new CustomSpringEvent(this, message);
        applicationEventPublisher.publishEvent(customSpringEvent);
    }
}
```

#### Subscriber
```java
@Component
public class CustomSpringEventListener implements ApplicationListener<CustomSpringEvent> {
    @Override
    public void onApplicationEvent(CustomSpringEvent event) {
        System.out.println("Received spring custom event - " + event.getMessage());
    }
}
```

### 참고자료
https://refactoring.guru/ko/design-patterns/observer

https://www.baeldung.com/spring-events

https://i-am-your-father.notion.site/Gang-of-Four-4e354fd5da844a56ace1e8454f39894b#95cb881f21cb4285b5b4ed0f1887b45f
