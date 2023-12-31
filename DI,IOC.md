# DI

DI(Dependency Injection)란,

어떤 객체가 사용하는 의존 객체를 직접 만들어 사용하는게 아니라, 주입 받아 사용하는 방법이다. 

외부에서 주입 받기 때문에 결합도를 낮출 수 있고, 런타임시에 의존관계가 결정되기 때문에 유연한 구조를 가진다.

![image](https://github.com/eunseo2/TIL/assets/70589857/28ece183-9883-4653-8381-6e797662b2d2)


첫번째 방법은 A객체가 B와 C객체를 New 생성자를 통해서 직접 생성하는 방법이고, 두번째 방법은 외부에서 생성 된 객체를 setter()를 통해 사용하는 방법이다.

이러한 두번째 방식이 의존성 주입의 예시인데, A 객체에서 B, C객체를 사용(의존)할 때 A 객체에서 직접 생성 하는 것이 아니라 외부(IOC컨테이너)에서 생성된 B, C객체를 조립(주입)시켜 setter 혹은 생성자를 통해 사용하는 방식이다.

### 생성자 기반 의존관계 주입을 선택해야 하는 이유?

### 1. 초기화시에 필요한 모든 의존관계가 형성되기 때문에 안전하다.

필요한 의존관계가 생성자에 필수로 제시되야한다면 이 객체가 생성시 필요한 모든 의존관계를 주입받았다고 확신할 수 있다. 그러면 나중에 참조하는 필드가 없어서 `NullPointerException` 이 발생할 확율이 없다.

### 2. 불변성을 확보한다.

생성자 주입은 불면객체를 만드는걸 도와준다. final 키워드를 사용해서 한번 만들어진 의존관계가 변경이 안된다.


![image](https://github.com/eunseo2/TIL/assets/70589857/c3fe9a28-5007-4866-9c7a-7713c40da943)


스프링 ApplicationContext는 일종에 ioc Container 이다.  

ApplicaitonContext는 BeanFactory를 상속하는데 객체에 대한 생성, 조합, 의존관계설정 등을 제어하는 IoC 기본기능을 BeanFactory에 담당한다.

Bean은 Ioc Container에서 관리 되어지는 객체를 의미한다.

Bean은 기본적으로 singleton scope으로 정의가 되는것을 알 수 있다.

### 싱글톤인 이유는?
사용자가 요청할 때마다 새로운 객체를 생성해서 제공하는 것은 비용이 크기 때문에 기본적으로 싱글톤을 사용하여 하나의 인스턴스를 공유한다.



# IOC
IoC(Inversion of Control)란 "제어의 역전" 이라는 의미로, 말 그대로 메소드나 객체의 호출작업을 개발자가 결정하는 것이 아니라, 외부에서 결정되는 것을 의미한다.

객체의 의존성을 역전시켜 객체 간의 결합도를 줄이고 유연한 코드를 작성할 수 있게 하여 가독성 및 코드 중복, 유지 보수를 편하게 할 수 있게 한다.

ex) 객체 생성 -> 의존성 객체 주입 (스스로가 만드는것이 아니라 제어권을 스프링에게 위임하여 스프링이 만들어놓은 객체를 주입한다) -> 의존성 객체 메소드 호출

제어의 흐름을 사용자가 컨트롤 하는 것이 아니라 스프링에게 맡겨 작업을 처리하게 된다.


### 참고자료
https://cheershennah.tistory.com/227

https://velog.io/@gillog/Spring-DIDependency-Injection
