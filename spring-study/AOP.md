# AOP

> AOP는 IoC/DI, 서비스 추상화와 더불어 스프링의 3대 기반기술의 하나다. 

> 스프링에 적용된 가장 인기 있는 AOP의 적용 대상은 바로 선언적 트랜잭션 기능이다. 서비스 추상화를 통해 많은 근본적인 문제를 해결했던 트랜잭션 경계설정 기능을 AOP를 이용해 더욱 세련되고 깔끔한 방식으로 바꿔보자.

UserService에서 Transaction 부분을 분리해보자.

트랜잭션 경계설정의 코드와 비즈니스 로직 코드 간에 서로 주고받는 정보가 없다.. 그렇다면?

`UserServiceTest`(Client)가 `UserService` 클래스를 직접 쓰는게 아니라 `UserService`를 `Interface`로 변환하고 `UserServiceImpl`이 구현하도록.

> 한 번에 두개의 UserService 인터페이스 구현 클래스를 동시에 이용한다면 어떨까?

`UserServiceTx` : 스스로는 비즈니스 로직을 담고 있지 않고 단지 트랜잭션의 경계설정이라는 책임을 맡음. 실제적인 로직 처리 작업은 `UserServiceImpl`에 위임.

#### 트랜잭션 적용을 위한 DI 설정

스프링의 DI 설정에 의해 결국 만들어질 빈 오브젝트와 그 의존관계는 다음과 같다

> Client -> UserServiceTx -> UserServiceImpl

#### 트랜잭션 분리에 따른 테스트 수정

```java
@Test
public void upgradeLevels() throws Exception {
    ...
    MockMailSender mockMailSender = new MockMailSender();
    userServiceImpl.setMailSender(mockMailSencder);
}

@Test
public void upgradeAllorNothing() throws Exception {
    TestUserService testUserService = new TestUserService(users.get(3).getId());
    testUserService.setUserDao(userDao);
    testUserService.setMailSender(mailSender);
    
    UserServiceTx txUserService = new UserServiceTx();
    txUserService.setTransactionManager(transactionManager);
    txUserService.setUserService(testUserSerivce);
    
    userDao.deleteAll();
    for(User user : users) userDao.add(user);
    
    try {
        txUserService.upgradeLevels();
        fail("TestUserServiceException expected");
    }
}

static class TestUserService extends UserServiceImpl { ... }
```

일반적인 `UserService` 테스트에서는 UserService만 있어도 되지만, MailSender 목 오브젝트를 이용한 테스트에서는 직접 MailSender를 DI해줘야 할 필요가 있어 DI해줄 대상을 구체적으로 알고 있어야 하기 때문에 UserServiceImpl 클래스의 오브젝트를 가져올 필요가 있다.

```java
userServiceImpl.setMailSender(mockMailSender);
```

## 테스트 대상 오브젝트 고립시키기

### 복잡한 의존관계 속의 테스트

![테스트 대상의 의존구조](D:\Downloads\feature2.png)

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

간단한 스태틱 메소드 호출만으로 다이내믹하게 특정 인터페이스를 구현한 테스트용 목 오브젝트 생성 가능

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

---

## 다이나믹 프록시와 팩토리 빈

트랜잭션 경계설정 코드를 비즈니스 로직 코드에서 분리해낼 때 적용했던 기법을 검토해보자

전략 디자인 패턴과 다르게, 클라이언트에서  부가기능을 사용하는데 핵심기능의 인터페이스로 마치 핵심기능을 쓰듯이 쓴다. 부가 기능은 다시 핵심기능을 호출해 핵심 로직 기능을 위임

> 이렇게 마치 자신이 클라이언트가 사용하려고 하는 실제 대상인 것처럼 위장해서 클라이언트의 요청을 받아주는 것을 대리자, 대리인과 같은 역할을 한다고 해서 프록시<sup>proxy</sup>라고 부른다. 그리고 프록시를 통해 최종적으로 요청을 위임받아 처리하는 실제 오브젝트를 타겟<sup>target</sup> 또는 실체<sup>real subject</sup> 라고 부른다.

프록시의 특징은 타깃과 같은 인터페이스를 구현했다는 것, 프록시가 타깃을 제어할 수 있는 위치라는것.

프록시는 사용 목적에 따라 두 가지로 구분.

1. 클라이언트가 타깃에 접근하는 방법을 제어하기 위해서 (타깃의 기능을 확장하거나 추가하지 않는다.)

   - 클라이언트가 해당 타깃을 사용하려 할 때 프록시가 타깃 오브젝트를 생성하고 요청을 위임(지연 생성)
   - 또는 특별한 상황에서 타깃에 대한 접근권한을 제어하기 위해 사용

2. 타깃에 부가적인 기능을 부여해주기 위해서 -> *데코레이터 패턴*

   ![img](https://github.com/young891221/blog/raw/master/images/Tobi/6.11.png)

#### 다이나믹 프록시

프록시를 만드는 일도 상당히 번거롭다. 매번 새로운 클래스를 정의해야 하고, 인터페이스의 구현해야 할 메소드는 많으면 모든 메소드를 일일이 구현해서 위임하는 코드를 넣어야 하기 때문. 프록시는 다음의 두 가지 기능으로 구성된다.

1. 타깃과 같은 메소드를 구현하고 있다가 메소드가 호출되면 타깃 오브젝트로 위임한다.
   -> 타깃의 인터페이스를 구현하고 위임하는 코드를 작성하기가 번거롭다.
2. 지정된 요청에 대해서는 부가기능을 수행한다.
   -> 부가기능 코드가 중복될 가능성이 많다.

![다이나믹 프록시의 동작방식](D:\Downloads\feature3.png)

> 다이나믹 프록시의 동작방식

자바에는 `java.lang.reflect` 패키지 안에 프록시를 손쉽게 만들게 지원해주는 클래스들이 있다. 마치 목프레임워크와 비슷하다. 

다이나믹 프록시 오브젝트는 타깃 인터페이스와 같은 타입으로 만들어진다. 클라이언트는 다이나믹 프록시 오브젝트를 타깃 인터페이스를 통해 사용. 다만 프록시로서 필요한 부가기능 제공 코드는 직접 작성해야한다. 부가기능은 프록시 오브젝트와 독립적으로 InvocationHandler를 구현한 오브젝트에 담는다. InvocationHandler 인터페이스는 다음과 같은 메소드 한 개만 가진 간단한 인터페이스다.

```java
public Object invoke(Object proxy, Method method, Object[] args)
```

`invoke()` 메소드는리플렉션의 Method 인터페이스를 파라미터로 받는다. 메소드를 호출할때 전달되는 파라미터도 args로 받는다. 다이나믹 프록시 오브젝트는 클라이언트의 모든 요청을 리플렉션 정보로 변환해서 InvocationHandler 구현 오브젝트의 invoke() 메소드로 넘기는 것이다.

`InvocationHandler`를 구현한 클래스 생성

---

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

```java
package org.springframework.beans.factory;

public interface FactoryBean<T> {
    T getObject() throws Exception; // 빈 오브젝트를 생성해서 돌려준다.
    Class<? extends T> getOBjectType(); // 생성되는 오브젝트의 타입을 알려준다.
    boolean isSingleton(); // getObject()가 돌려주는 오브젝트가 항상 같은 싱글톤 오브젝트인지 알려준다.
}
```

팩토리빈을 구현하면 TransactionHandler는 더이상 필요하지 않음

```java
public class TxProxyFactoryBean implements FactoryBean<Object> {
    Object target;
    PlatformTransactionManager transactionManager;
    String pattern;
    Class<?> serviceInterface;
    
    public void setTarget(Object targer) {
        this.target = targer;
    }
    
    public void setTransactionManager(PlatformTransactionManager transactionManager) {
        this.transactionManager = transactionManager;
    }
    
    public void setPattern(String pattern) {
        this.pattern = pattern;
    }
    
    public void setServiceInterface(Class<?> serviceInterface) {
        this.serviceInterface = serviceInterface;
    }
    
    //FactoryBean 인터페이스 구현 메소드
    public Object getObject() throws Exception {
        TransactionHandler txHandler = new TransactionHandler();
        txHandler.setTarget(targer);
        txHandler.setTransactionManager(transactionManager);
        txHandler.setPattern(pattern);
        return Proxy.newProxyInstance(
                getClass().getClassLoader(), new Class[] { serviceInterface },
                txHandler);
    }
    
    /*
	 * DI 받은 인터페이스 타입에 따라 팩토리 빈이 생성하는 오브젝트 타입이 달라진다.
	 * 다양한 프록시 오브젝트 생성을 위한 재사용 코드
	 */
    public Class<?> getObjectType() {
        return serviceInterface;
    }
    
    /*
     * 싱글톤 빈이 아니라는 뜻이 아니라
     * getObject()가 매번 같은 오브젝트를 리턴하지 않는다는 의미
     */
    public boolean isSingleton() {
        return false;
    }
}

```

![팩토리 빈을 이용한 트랜잭션 다이나믹 프록시 적용](https://github.com/young891221/blog/raw/master/images/Tobi/6.15.png)

```java
<bean id="userService" class="springbook.user.service.TxproxyFactoryBean">
	<property neme="taget" ref="userSerivceImpl" />	//다른 빈을 가리키는것은 ref
	<property neme="transactionManager" ref="transactionManager" />
	<property neme="pattern" value="upgradeLevels" />	//pattern은 string이니 값을 지정하는 value
	<property neme="serviceInterface" value="springbook.user.service.UserService" />
    // class타입인 serviceInterface는 value를 이용해 클래스 또는 인터페이스 이름을 넣어줌
</bean>
```

#### ` UserServiceTest` 테스트를 살펴보자

add()는 @Autowired로 가져온 userService 빈을 사용하기 때문에 TxProxyFactoryBean이 생성하는 다이나믹 프록시를 통해 UserService 기능을 사용하게 될것. 반면 upgradeLevels()와 mockUpgradeLevels()는 목 오브젝트를 이용해 비즈니스 로직에 대한 단위 테스트로 만들었으니 트랜잭션과 무관. 가장 중요한 트랜잭션 적용 기능을 확인하는 **`upgradeAllOrNothing()`의 경우는 수동 DI를 통해 직접 다이나믹 프록시를 만들어 사용하니 팩토리 빈이 적용 안됨.** 예외 발생 시 트랜잭션이 롤백됨을 확인하려면 비즈니스 로직 코드를 수정한 **TestUserService 오브젝트를 타깃 오브젝트로 대신 사용해야 한다**. 설정에는 정상적인 UserServiceImpl 오브젝트로 지정되어 있지만 테스트 메소드에서 TestUserService 오브젝트가 동작하도록 해야한다.

어떻게 해야할까? TestUserService를 사용하는 테스트용 설정을 별도로 만든다거나 프록시 팩토리 빈 코드를 확장한다거나 하는 방법도 가능하겠지만, 여기서는 가장 단순한 방법을 써보자. 어차피 TxProxyFactoryBean의 트랜잭션을 지원하는 프록시를 바르게 만들어 주는지를 확인하는게 목적이므로 빈으로 등록된 **TxProxyFactoryBean을 직접 가져와서 프록시를 만들어보면 된다.**

팩토리 빈은 내부에서 생성하는 오브젝트가 빈 오브젝트로 사용되지만, 원한다면 팩토리 빈 자체를 가져올 수도 있음. `TxProxyFactoryBean`을 가져와서 target 프로퍼티를 재구성해준 뒤 다시 프록시 오브젝트를 생성하도록 요청할 수 있다. (컨텍스트의 설정을 변경해버리기에 `@DirtiesContext` 등록)

```java
//...
TxProxyFactoryBean txPFB = context.getBean("&userService", TxProxyFactoryBean.class);
txPFB.setTarget(testUserService);
UserService txUserService = (UserService) txProxyFactoryBean.getObject();
//...
```

---

### 프록시 팩토리 빈의 한계

##### 장점 - 프록시 팩토리 빈의 재사용

​	TransactionHandler를 이용하는 다이나믹 프록시를 생성해주는 `TxProxyFactoryBean`은 코드 수정 없이도 다양한 클래스에 적용 가능. 타깃 오브젝트에 맞는 프로퍼티 정보를 설정해서 빈으로 등록해주기만 하면 된다. 하나 이상의 `TxProxyFactoryBean`을 동시에 빈으로 등록해도 상관없다. 팩토리 빈이기 때문에 각 빈의 타입은 타깃 인터페이스와 일치한다.

##### 한계점

​	하나의 클래스 안에 여러 개의 메소드 적용은 가능하지만 여러 개의 클래스에 공통적인 부가기능을 제공하는 일을 불가능

​	하나의 타깃에 보안, 트랜잭션 등 여러 개의 부가기능을 적용하려 할 때도 문제.

> 적용 대상인 서비스 클래스가 200개쯤 된다면 보통 하나당 3~4줄이면 되는 서비스 빈의 설정에 5~6줄씩 되는 프록시 팩토리 빈 설정이 부가기능의 개수만큼 따라 붙어야한다.

​	또 한가지 문제점 은 TransactionHandler 오브젝트가 프록시 팩토리 빈 개수만큼 만들어진다는 점.

​	그래도 코드 수정 없이 설정 파일 수정으로 그칠 수 있다는 사실에 감지덕지 해야할까?

---

### 스프링의 프록시 팩토리 빈

스프링은 매우 세련되고 깔끔한 방식으로 이런 문제 해법 제공

#### ProxyFactoryBean

기존에 만들었던 `TxProxyFactoryBean`과 달리, `ProxyFactoryBean`은 순수하게 프록시를 생성하는 작업만을 담당하고 프록시를 통해 제공해줄 **부가기능은 별도의 빈에 둘 수 있다.**

ProxyFactoryBean이 생성하는 프록시에서 사용할 부가기능은 `MethodInterceptor`인터페이스를 구현해서 만든다. `MethodInterceptor`는 `InvocationHandler`와 비슷하지만 한 가지 다른 점이 있다. InvocationHandler의 invoke() 메소드는 타깃 오브젝트에 대한 정보를 제공하지 않는다. 따라서 타깃은 InvocationHandler를 구현한 클래스가 직접 알고 있어야한다. **반면 MethodInterceptor의 inovoke()메소드는 ProxyFactoryBean으로부터 타깃 오브젝트에 대한 정보까지도 함께 제공받는다**. 그 차이 덕분에 MethodInterceptor는 타깃 오브젝트에 상관없이 독립적으로 만들어질 수 있다. 따라서 MethodInterceptor 오브젝트는 타깃이 다른 여러 프록시에서 함께 사용할 수 있고, 싱글톤 빈으로 등록 가능하다.

```java
// 스프링 ProxyFactoryBean을 이용한 다이나믹 프록시 테스트
public class DynamicProxyTest {
    @Test
    public void simpleProxy() {
        //JDK 다이나믹 프록시 생성
        Hello proxiedHello = (Hello) Proxy.newProxyInstance(
                getClass().getClassLoader(),
                new Class[] { Hello.class },
                new UppercaseHandler(new HelloTarget()));
       // ...
    }
    
    @Test
    public void proxyFactoryBean() {
        ProxyFactoryBean pfBean = new ProxyFactoryBean();
        pfBean.setTarget(new HelloTarget()); //타깃 설정
        pfBean.addAdvice(new UppercaseAdvice()); //부가기능을 다은 advice 추가 (여러개 가능)
        Hello proxiedHello = (Hello) pfBean.getObject(); //FacotryBean이므로 생성된 프록시를 가져온다.
        
        assertThat(ProxiedHello.sayHello("Havi"), is("Hello Havi"));
       // ...
    }
    
    static class UppercaseAdvice implements MethodInterceptor {
        public Object invoke(MethodInvocation invocation) throws Throwable {
            String ret = (String)invocation.proceed(); //타깃을 알고 있기에 타깃 오브젝트를 전달할 필요가 없다.
            return ret.toUpperCase(); //부가기능 적용
        }
    }
    
    static interface Hello {	// 타깃과 프록시가 구현할 인터페이스
        String sayHello(String name);
        String sayHi(String name);
        String sayThankYor(String name);
    }
    
    static class HelloTarget implements Hello {	//타깃 클래스
        public String sayHello(String name) { return "Hello" + name; }
        //...
    }
}

```

#### 어드바이스 :  타깃이 필요없는 순수한 부가기능

`MethodInvocation`은 일종의 콜백 오브젝트로, proceed() 메소드를 실행하면 타깃 오브젝트의 메소드를 내부적으로 실행해주는 기능이 있다. ProxyFactoryBean은 작은 단위의 템플릿/콜백 구조를 응용하여 적용하였기에 템플릿 역할을 하는 MethodInvocation을 싱글톤으로 두고 공유할수 있다. 바로 이 점이 JDK의 다이나믹 프록시를 직접 사용하는 코드와 스프링이 제공해주는 프록시 추상화 기능인 ProxyFactoryBean을 사용하는 코드의 가장 큰 차이점이자 ProxyFactoryBean의 장점이다. 마치 JdbcTemplate과 수많은 DAO메소드의 관계와 같다.

> MethodInterceptor는 Advice 인터페이스를 상속한다. 

**어드바이스는 MethodInvocation처럼 타깃 오브젝트에 적용하는부가기능을 담은 (타깃 오브젝트에 종속되지 않는) 오브젝트이다.**

스프링의 ProxyFactoryBean은 인터페이스 자동검출 기능을 사용해 타깃 오브젝트가 구현하고 있는 인터페이스 정보를 알아낸다. 그리고 알아낸 인터페이스를 모두 구현하는 프록시를 만들어준다. 

#### 포인트컷 : 부가기능 적용 대상 메소드 선정 방법

InvocationHandler를 직접 구현했을 때 메소드 이름 가지고 부가기능 적용 대상 메소드 선정이 가능했다. 그런데 스프링의 ProxyFactoryBean과 MethodInterceptor를 사용하는 방식에서는 불가능하다. MethodInterceptor는 싱글톤 빈으로 등록되어 여러 프록시가 공유하기에 특정 프록시에만 적용되는 패턴은 문제가 된다.

대신 프록시에 부가기능 적용 메소드를 선택하는 기능을 넣자. 물론 프록시의 핵심 가치는 타깃을 대신해서 클라이언트의 요청을 받아 처리하는 오브젝트로서의 존재 가치이므로, 메소드 선별기능도 분리하는게 낫다.

![img](https://t1.daumcdn.net/cfile/tistory/99E1D5465C42666524)

```java
@Test
public void pointcutAdvisor() {
    ProxyFactoryBean pfBean = new ProxyFactoryBean();
    pfBean.setTarget(new HelloTarget());
    
    NameMatchMethodPointcut pointcut = new NameMatchMethodPointcut(); //메소드이름을 비교대상으로 선정하는 포인트컷 생성
    pointcut.setMappedName("sayH*"); //이름 비교조건 설정(sayH로 시작하는 모든 메소드 선택)
    pfBean.addAdvisor(new DefaultPointcutAdvisor(pointcut, new UppercaseAdvice())); //포인트컷과 어드바이스를 Advisor로 묶어서 한 번에 추가
    
    Hello proxiedHello = (Hello) pfBean.getObject();
    assertThat(ProxiedHello.sayHello("Havi"), is("Hello Havi"));
    assertThat(ProxiedHello.sayThankYou("Havi"), is("Thank You Havi")); //적용 안됨
}
```

#### ProxyFactoryBean 적용

JDK 다이나믹 프록시의 구조를 그대로 이용해 만들었던 TxProxyFactoryBean을 이제 스프링이 제공하는 ProxyFactoryBean을 이용하도록 수정해보자.



```java
// 부가기능을 담당하는 어드바이스는 테스트에서 만들어본 것처럼 MethodInterceptor라는 Advice 서브인터페이스를 구현해서 만든다.
// 이전에 만든 TransactionHandler에서 타깃과 메소드 선정 부분을 제거해 주면 된다.
public class TransactionAdvice implements MethodInterceptor { //스프링 어드바이스 인터페이스 구현
    PlatformTransactionManager transactionManager;
    
    public void setTransactionManager(PlatformTransactionManager transactionManager) {
        this.transactionManager = transactionManager;
    }
    
    //타깃을 호출하는 기능을 가진 콜백 오브젝트를 프록시로부터 받는다. 덕분에 어드바이스는 특정 타깃에 의존하지 않고 재사용 가능하다.
    public Object invoke(MethodInvocation invocation) throws Throwable {
        TransactionStatus status = this.transactionManager.getTransaction(new DefaultTransactionDefinition());
        
        try {
            Object ret = invocation.proceed(); //타깃의 메소드 실행. 호출 전후로 부가기능을 추가할 수 있다.
            this.transactionManager.commit(status);
            return ret;
        } catch(RuntimeException e) {	//JDK 다이나믹 프록시가 제공하는 Method완 달리 스프링의 MethodInvocation을 통한 타깃 호출은 예외가 포장되지 않고 타깃에서 보낸 그대로 전달된다.
            this.transactionManager.rollback(status);
            throw e;
        }
    }
}
```

#### 스프링 XML 설정파일

남은건 설정파일

###### 트랜잭션 어드바이스 빈 설정

```xml
<bean id="transcationAdvice" class="springbook.user.service.TransactionAdvice">
	<property name="transactionManager" ref="transactionManager"/>
</bean>
```

###### 포인트컷 빈 설정

```xml
<bean id="transactionPointcut" class="org.springframework.aop.support.NameMatchMethodPointCut">
	<property name="mappedName" value="upgrade*" />
</bean>
```

###### 어드바이스와 포인트컷을 담을 어드바이저. 생성자도 가능

```xml
<bean id="transactionAdvisor" class="org.springframework.aop.support.DefaultPointcutAdvisor">
	<property name="advice" ref="transactionAdvice" />
	<property name="pointcut" ref="transactionPointcut" />
</bean>
```

###### ProxyFactoryBean 설정

```xml
<bean id="userService" class="org.springframework.aop.framework.ProxyFactoryBean">
	<property name="target" ref="userServiceImpl"/>
    <property name="interceptorNames">	//어드바이스와 어드바이저를 동시에 설정해줄 수 있는 프로퍼티
    	<list>
        	<value>transactionAdvisor</value>
        </list>
    </property>
</bean>
```

```java
@Test
@DirtiesContext
public void upgradeAllOrNothind() {
    TestUserService testUserService = new TestUserService(users.get(3).getId());
    testUserService.setUserDao(userDao);
    testUserService.setMailSender(mailSender);
    
    ProxyFactoryBean txProxyFactoryBean = context.getBean("&userService", ProxyFactoryBean.class);
    txProxyFactoryBean.setTarget(testUserService);
    UserService txUserService = (UserService) txProxyFactoryBean.getObject();
}
```

##### 어드바이스와 포인트컷의 재사용

---

## 스프링 AOP

#### 빈 후처리기를 이용한 자동 프록시 생성기

스프링은 컨테이너로서 제공되는 기능 중에서 변하지 않는 핵심적인 부분 외에는 대부분 확장 가능하도록 해준다. 그 중 중요한 확장 포인트는 BeanPostProcessor 인터페이스를 구현해서 만드는 빈 후처리기이다. 그 중 하나인 DefaultAdvisorAutoProxyCreator는 어드바이저를 이용한 자동 프록시 생성기이다. 빈 후처리기의 등록은 간단하게 후처리기 자체를 빈으로 등록하기만 하면 된다.

#### 확장된 포인트컷

포인트컷은 **ClassFilter**, MethodMatcher를 돌려주는 메소드 두개를 갖고 있다. DefaultAdvisorAutoProxyCreator는 클래스와 메소드 선정 알고리즘을 모두 갖고 있는 포인트컷이 필요하다. 정확히는 그런 포인트컷과 어드바이스가 결합되어 있는 어드바이저가 등록되어 있어야 한다.

##### 어드바이저를 이용하는 자동 프록시 생성기 등록

적용할 자동 프록시 생성기인 DefaultAdvisorAutoProxyCreator는 등록된 빈 중에서 Advisor 인터페이스를 구현한 것을 모두 찾는다. 그리고 생성되는 모든 빈에 대해 어드바이저의 포인트컷을 적용해보면서 프록시 적용 대상을 선정한다. 빈 클래스가 프록시 선정 대상이라면 프록시를 만들어 원래 빈 오브젝트와 바꿔치기한다. 원래 빈 오브젝트는 프록시 뒤에 연결돼서 프록시를 통해서만 접근 가능하게 바뀌는 것이다. *따라서 타깃 빈에 의존한다고 정의한 다른 빈들은 프록시 오브젝트를 대신 DI 받게 될 것이다.*

##### ProxyFactoryBean 제거와 서비스 빈의 원상복구



### DefaultAdvisorAutoProxyCreator의 적용

#### 클래스 필터를 적용한 포인트컷 작성

> NameMatchMethodPointcut을 상속해서 프로퍼티로 주어진 이름 패턴을 가지고 클래스 이름을 비교하는 ClassFilter를 추가

```java
public class NameMatchClassMethodPointcut extends NameMatchMethodPointcut {
    public void setMappedClassName(String mappedClassName) {
        this.setClassFilter(new SimpleClassFilter(mappedClassName));
    }
    static class SimpleClassFilter implements ClassFilter {
        String mappedName;
        private SimpleClassFilter(String mappedName){
            this.mappedName = mappedName;
        }
        public boolean matches(Class<?> clazz){
            return PatternMatchUtils.simpleMatch(mappedName, clazz.getSimpleName());
        }
    }
}
```

### 포인트컷 표현식을 이용한 포인트컷

---



### AOP 네임스페이스

스프링의 프록시 방식 AOP를 적용하려면 최소한 네 가지 빈을 등록해야 한다.

- 자동 프록시 생성기

  : DefaultAdvisorAutoProxyCreator 클래스를 빈으로 등록한다.

  - 빈으로 등록된 어드바이저를 이용해서 프록시를 자동으로 생성하는 기능을 담당한다.

- 어드바이스: 부가기능을 구현한 클래스를 빈으로 등록한다.

- 포인트컷: 스프링의 AspectJExpressionPointcut을 빈으로 등록하고 expression 프로퍼티에 포인트컷 표현식을 넣어주면 된다.

- 어드바이저

  : 스프링의 DefaultPointcutAdvisor 클래스를 빈으로 등록해서 사용한다.

  - 어드바이스와 포인트컷을 프로퍼티로 참조하는 것 외의 기능은 없다.
  - 자동 프록시 생성기에 의해 자동 검색된다.

###### aop 네임스페이스를 적용한 AOP 설정 빈

```xml
<!-- aop 설정을 담는 부모 태그. 필요에 따라 AspectJAdvisorAutoProxyCreator를 빈으로 등록 -->
<aop:config>
	<aop:pointcut id="transactionPointcut"
                  expression="execution(* *..*ServiceImpl.upgrade*(..))" />
	<aop:advisor advice-ref="transactionAdvice" pointcut-ref="transactionPointcut" />
</aop:config>

<!--어드바이저와 결합된 형태(더 간편하게 사용)-->
<aop:config>
	<aop:advisor advice-ref="transactionAdvice" pointcut-ref="execution(**..*ServiceImpl.upgrade*(..))" />
</aop:config>
```

`<aop:config>`, `<aop:pointcut>`, `<aop:advisor>` 세 가지 태그를 정의해두면 그에 따라 세 개의 빈이 자동으로 등록된다.