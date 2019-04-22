## Test 부분

#### 단위 테스트

테스트 검증의 자동화

#### JUnit 테스트로 전환

`@Test`

```java
assertThat(user2.getName(), is(user.getName())); // is는 matcher
```

```java
ppublic class UserDaoTest {
    @Test
    public void addAndGet() throws SQLException {
        ApplicationContext context = new GenericXmlApplicationContext("applicationContext.xml");
        UserDao dao = context.getBean("userDao", UserDao.class);
        User user = new User();
        user.setId("gyumee");
        user.setName("박성철");
        user.setPassword("springno1");
        
        dao.add(user);
        
        User user2 = dao.get(user.getId());
        
        assertThat(user2.getName(), is(user.getName()));
        assertThat(user2.getPassword(), is(user.getPassword()));
    }
}
```

테스트 시마다 DB를 초기화하기?

`deleteAll()` `getCount()` 추가하고 각각 `addAndGet()`에서 같이 검증

예외처리 검증

`@Test(expected=EmptyResultDataAccessException.class)`

> 테스트를 작성할 때 부정적인 케이스를 먼저 만드는 습관을 들이는게 좋다.

JUnit 기능

`@Before`

`setUp()` 메소드를 만들고 중복된 코드 삽입

#### 픽스처<sup>fixture</sup>

테스트를 수행하는데 필요한 정보나 오브젝트. @Before 메소드를 이용해 생성해 두면 편리.

```java
@RunWithSpringJUnit4ClassRunner.class)
@Contextconfiguration(locations="/applicationContext.xml")
public class UserDaoTest {
    @Autowired
    private Application context;
    ...
    @Befroe
        public void setUp(){
        this.dao = this.context.getBean("userDao",Userdao.class);
    }   
}
```

#### 테스트 클래스의 컨텍스트 공유

> 여러 개의 테스트 클래스가 있는데 모두 같은 설정파일을 가진 애플리케이션 컨텍스트를 사용한다면,  스프링은 테스트 클래스 사이에서도 애플리케이션 컨텍스트를 공유하게 해준다.

```java
@RunWithSpringJUnit4ClassRunner.class)
@ContextConfigurationi(locations="/applicationContext.xml")
public class UserDaoTest{}

@RunWithSpringJUnit4ClassRunner.class)
@ContextConfigurationi(locations="/applicationContext.xml")
public class GroupDaoTest{}
```



## 학습 테스트

> 개발자가 자신이 만든 코드가 아닌 다른 사람이 만든 코드와 기능에 대한 테스트를 작성할 필요가 있을까? 때로는 자신이 만들지 않은 프레임워크나 다른 개발팀에서 만들어서 제공한 라이브러리 등에 대해서도 테스트를 작성해야 한다. 이런 테스트를 학습 테스트<sup>learning test</sup>라고 한다.

- 장점

  다양한 조건에 따른 기능을 손쉽게 확인해볼 수 있다.

  학습 테스트 코드를 개발 중에 참고할 수 있다.

  프레임워크나 제품을 업그레이드할 때 호환성 검증을 도와준다.

  테스트 작성에 대한 좋은 훈련이 된다.

  새로운 기술을 공부하는 과정이 즐거워진다(?)