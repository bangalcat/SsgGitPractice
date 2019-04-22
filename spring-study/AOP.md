# AOP

> AOP는 IoC/DI, 서비스 추상화와 더불어 스프링의 3대 기반기술의 하나다. 

> 스프링에 적용된 가장 인기 있는 AOP의 적용 대상은 바로 선언적 트랜잭션 기능이다. 서비스 추상화를 통해 많은 근본적인 문제를 해결했던 트랜잭션 경계설정 기능을 AOP를 이용해 더욱 세련되고 깔끔한 방식으로 바꿔보자.

UserService에서 Transaction 부분을 분리해보자.

트랜잭션 경계설정의 코드와 비즈니스 로직 코드 간에 서로 주고받는 정보가 없다..

`UserServiceTest`(Client)가 `UserService` 클래스를 직접 쓰는게 아니라 `UserService`를 `Interface`로 변환하고 `UserServiceImpl`이 구현하도록.

> 한 번에 두개의 UserService 인터페이스 구현 클래스를 동시에 이용한다면 어떨까?

`UserServiceTx` : 스스로는 비즈니스 로직을 담고 있지 않고 단지 트랜잭션의 경계설정이라는 책임을 맡음. 실제적인 로직 처리 작업은 `UserServiceImpl`에 위임.

#### 트랜잭션 적용을 위한 DI 설정

스프링의 DI 설정에 의해 결국 만들어질 빈 오브젝트와 그 의존관계는 다음과 같다

> Client -> UserServiceTx -> UserServiceImpl

#### 트랜잭션 분리에 따른 테스트 수정

```java
@Autowired UserService userService;
```

일반적인 `UserService` 테스트에서는 UserService만 있어도 되지만, MailSender 목 오브젝트를 이용한 테스트에서는 직접 MailSender를 DI해줘야 할 필요가 있어 DI해줄 대상을 구체적으로 알고 있어야 하기 때문에 UserServiceImpl 클래스의 오브젝트를 가져올 필요가 있다.

```java
userServiceImpl.setMailSender(mockMailSender);
```

## 테스트 대상 오브젝트 고립시키기

### 복잡한 의존관계 속의 테스트

![테스트 대상의 의존구조](C:\Users\user\Downloads\feature2.png)

테스트하고자 하는 대상인 UserService 코드만 바르게 작성되어 있으면 된다. 그런데 3가지 의존관계를 가지고 있어서 테스트가 진행되는 동안에 같이 실행된다. 게다가 3가지 의존 오브젝트들도 또다른 의존성이 있음

### 테스트 대상 오브젝트 고립시키기

*고립시킨 USerServiceImpl에 대한 테스트 구조*

##### UserDao 목 오브젝트

`userDao.getAll()` 테스트용 UserDao에는 DB에서 읽어온것처럼 미리 준비된 사용자 목록을 제공해줘야 한다. `userDao.update(user)`의 호출은 리턴값이 없다. 따라서 테스트용 UserDao가 특별히 미리 준비해둘 것은 없다. 테스트가 진행되도록 하기 위해서라면 아무런 내용도 없는 빈 메소드로 만들어도 된다.

> 하지만 `update()` 메소드의 사용은 `upgradeLevels()`의 핵심 로직인 '전체 사용자 중에서 업그레이드 대상자는 레벨을 변경해준다'에서 '변경'에 해당하는 부분을 검증할 수 있는 중요한 기능이기도 하다. 업그레이드를 통해 레벨이 변경된 사용자는 DB에 반영되도록 userDao의 `update()`에 전달돼야 하기 때문이다.

그래서 getAll()에 대해서는 스텁으로서, update()에 대해서는 목 오브젝트로서 동작하는 UserDao타입의 테스트 대역이 필요하다. 이 클래스의 이름을 `MockUserDao`라고 하자.

### 단위테스트와 통합테스트

단위테스트와 통합테스트 중에 어떤 방법을 쓸 것인가에 대한 가이드라인

- 항상 단위 테스트를 먼저 고려
- 외부 리소스를 사용해야만 하는 테스트는 통합 테스트로
- 대표적으로 DAO. DB라는 외부 리소스를 사용하기에 통합 테스트로 분류되지만, 코드에서 보자면 하나의 기능 단위를 테스트하는 것이기도 하다. DAO를 충분히 검증해두면, DAO를 이용하는 코드를 DAO 역할을 스텁이나 목 오브젝트로 대체해서 테스트 가능.

### 목 프레임워크

다행히도 번거로운 목 오브젝트를 편리하게 작성하도록 도와주는 다양한 목 오브젝트 지원 프레임워크가 있다.

#### Mockito 프레임워크

그 중에서도 사용하기도 편리하고, 코드도 직관적인 Mockito 프레임워크.

간단한 스태틱 메소드 호출만으로 다이내믹하게 특정 인터페이스르르 구현한 테스트용 목 오브젝트 생성 가능

```java
UserDao mockUserDao = mock(UserDao.class)
```

여기에 먼저 getAll() 메소드가 불려올 때 사용자 목록을 리턴하도록 스텁 기능 추가

```java
when(mockUserDao.getAll()).thenResturn(this.users);
```

update() 호출이 있었는지 검증 :

```java
verify(mockUserDao, times(2)).update(any(User.class));
```



## 다이나믹 프록시와 팩토리 빈

트랜잭션 경계설정 코드를 비즈니스 로직 코드에서 분리해낼 때 적용했던 기법을 검토해보자

전략 디자인 패턴과 다르게, 클라이언트에서  부가기능을 사용하는데 핵심기능의 인터페이스로 마치 핵심기능을 쓰듯이 쓴다. 부가 기능은 다시 핵심기능을 호출해 핵심 로직 기능을 위임

> 이렇게 마치 자신이 클라이언트가 사용하려고 하는 실제 대상인 것처럼 위장해서 클라이언트의 요청을 받아주는 것을 대리자, 대리인과 같은 역할을 한다고 해서 프록시<sup>proxy</sup>라고 부른다. 그리고 프록시를 통해 최종적으로 요청을 위임받아 처리하는 실제 오브젝트를 타겟<sup>target</sup> 또는 실체<sup>real subject</sup> 라고 부른다.

프록시의 특징은 타깃과 같은 인터페이스를 구현했다는 것, 프록시가 타깃을 제어할 수 있는 위치라는것.

프록시는 사용 목적에 따라 두 가지로 구분.

1. 클라이언트가 타깃에 접근하는 방법을 제어하기 위해서
2. 타깃에 부가적인 기능을 부여해주기 위해서 -> *데코레이터 패턴*

#### 다이나믹 프록시

프록시를 만드는 일도 상당히 번거롭다. 매번 새로운 클래스를 정의해야 하고, 인터페이스의 구현해야 할 메소드는 많으면 모든 메소드를 일일이 구현해서 위임하는 코드를 넣어야 하기 때문.

1. 타깃의 인터페이스를 구현하고 위임하는 코드를 작성하기가 번거롭다.
2. 부가기능 코드가 중복될 가능성이 많다.

![](C:\Users\user\Downloads\feature3.png)

> 다이나믹 프록시의 동작방식

다이나믹 프록시 오브젝트는 타깃 인터페이스와 같은 타입으로 만들어진다. 클라이언트는 다이나믹 프록시 오브젝트를 타깃 인터페이스를 통해 사용. 다만 프록시로서 필요한 부가기능 제공 코드는 직접 작성해야한다. 부가기능은 프록시 오브젝트와 독립적으로 InvocationHandler를 구현한 오브젝트에 담는다. InvocationHandler 인터페이스는 다음과 같은 메소드 한 개만 가진 간단한 인터페이스다.

```java
public Object invoke(Object proxy, Method method, Object[] args)
```

`invoke()` 메소드는리플렉션의 Method 인터페이스를 파라미터로 받는다. 메소드를 호출할때 전달되는 파라미터도 args로 받는다. 다이나믹 프록시 오브젝트는 클라이언트의 모든 요청을 리플렉션 정보로 변환해서 InvocationHandler 구현 오브젝트의 invoke() 메소드로 넘기는 것이다.

`InvocationHandler`를 구현한 클래스 생성

### 다이나믹 프록시를 이용한 트랜잭션 부가기능

##### 트랜잭션 InvocationHandler

```java
public class TransactionHandler implements InvocationHandler {
    private Object target;
    private PlatforTransactionManager transactionManager;
    private String pattern;
    // ...
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        //트랜잭션 적용 대상 메소드를 선별해서 트랜잭션 경계설정 기능을 부여
        if(method.getName().startsWith(pattern)) {
            return invokeInTransaction(method, args);
        } else {
            return method.invoke(target, args);
        }
    }
    private Object invokeInTransaction(Method method, Object[] args) throws Throwable {
        TransactionStatus status = this.transactionManager.getTransaction(new DefaultTransactionDefinition());
        try {
            //트랜잭션 시작하고 타깃 오브젝트의 메소드를 호출
            Object ret = method.invoke(target, args);
            this.transactionManager.commit(status);
            return ret;
        } catch (InvocationTargetException e){
            this.transactionManager.rollback(status);
            throw e.getTargetException();
        }
    }
}
```

### 다이나믹 프록시를 위한 팩토리 빈

어떤 타깃에도 적용가능한 트랜잭션 부가기능을 담은 TransactionHandler를 만들엇고, 이를 이용하는 다이나믹 프록시를 UserService에 적용하는 테스트를 만들어봤다. 이제 TransactionHandler와 다이나믹 프록시를 스프링의 DI를 통해 사용할 수 있도록 만들어야 할 차례다.

스프링은 내부적으로 리플렉션 API를 이용해서 빈 정의에 나오는 클래스 이름을 가지고 빈 오브젝트를 생성한다. 문제는 다이나믹 프로젝트는 이런식으로 프록시 오브젝트가 생성되지 않는다는 점이다. 사실 다이나믹 프록시 오브젝트의 클래스가 어떤 것인지도 알 수도 없다. 따라서 사전에 프록시 오브젝트의 클래스 정보를 미리 알아내서 스프링의 빈에 정의할 방법이 없다. 다이나믹 프록시는 Proxy 클래스의 newProxyInstance()라는 스태틱 팩토리 메소드를 통해서만 만들 수 있다.

##### 팩토리 빈

스프링은 클래스 정보를 가지고 디폴트 생성자를 통해 오브젝트를 만드는 방법 외에도 빈을 만들 수 있는 여러 가지 방법을 제공한다. 대표적으로 팩토리 빈을 이용한 빈 생성 방법을 들 수 있다.



### 스프링의 프록시 팩토리 빈

