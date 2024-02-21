# Strategy Pattern
- 실행(Runtime, 런타임) 중에 알고리즘 전략을 선택하여 객체 동작을 실시간으로 바뀌도록 할 수 있게 하는 행위 디자인 패턴
- “변하지 않는 것”은 그대로 두고 “변하는 것”은 찾아서 나머지 코드에 영향을 주지 않도록 캡슐화를 해야 한다.
- 캡슐화라는 것은 객체 간의 협력 시, 내부 구현은 숨기고 퍼블릭 인터페이스만 사용해서 서로 간의 메시지를 주고 받게 만드는 것을 캡슐화라고 한다.


## 구조
<img width="659" alt="image" src="https://github.com/eunseo2/TILL/assets/70589857/17e8643e-e176-4d5c-bdcf-b7399f259a2a">

- 전략 패턴은 상속이 아닌 구성을 사용한다.
- 변하지 않는 것은 Navigator에 그대로 두고 변할 수 있는 RouteStrategy는 공통 인터페이스를 통해서 캡슐화 되어있다.
- Navigator는 RouteStrategy의 구현이 아닌 인터페이스만 알고 있기 때문에 최소한의 정보만 알고 있으므로 느슨한 결합도이다.
- RouteStrategy의 내부 구현이 변경되더라도 Navigator까지는 변경의 전염성이 퍼지지 않는다.


### 실행
```java
public class Factory {
    public Navigator createNavigator(String strategy) {
        RouteStrategy selected = createStrategy(strategy);
        return new Navigator(selected);
    }

    private RouteStrategy createStrategy(String strategy){
        return switch (strategy) {
            case "road" -> new RoadStrategy();
            case "walking" -> new WalkingStrategy();
            default -> new PublicTransportStrategy();
        };
    }
}
```

### 참고자료
https://refactoring.guru/ko/design-patterns/strategy

https://i-am-your-father.notion.site/Gang-of-Four-4e354fd5da844a56ace1e8454f39894b#95cb881f21cb4285b5b4ed0f1887b45f
