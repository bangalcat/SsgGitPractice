# IoC와 DI

스프링이 XML에 담긴 내용을 읽어서 설정 메타정보로 활용하는 건 사실이지만, 그렇다고 해서 스프링이 XML로 된 설정 메타정보를 가졌다는 말은 틀렸다.

## 웹 어플리케이션의 IoC 컨테이너 구성

서버에서 동작하는 애플리케이션에서 스프링 IoC 컨테이너를 사용하는 방법은 크게 세 가지다. 두 가지 방법은 웹 모듈 안에 컨테이너를 두는 것, 나머지 하나는 엔터프라이즈 애플리케이션 레벨에 두는 방법

> 몇 개의 서블릿이 중앙집중식으로 모든 요청을 다 받아서 처리하는 이런 방식을 **프론트 컨트롤러 패턴**이라고 합니다. 스프링 웹 애플리케이션에 사용되는 서블릿의 숫자는 하나이거나 많아야 두셋정도다.

#### 웹 애플리케이션의 컨텍스트 계층구조

- 서블릿 컨텍스트와 루트 애플리케이션 컨텍스트 계층구조

  - 스프링 웹 기술을 사용하는 경우 웹 관련 빈들은 서블릿의 컨텍스트에 두고 나머지는 루트 애플리케이션 컨텍스트에 등록

- 루트 애플리케이션 컨텍스트 단일구조

  - 스프링 웹 기술을 사용하지 않고 서드파티 웹 프레임워크나 서비스 엔진만을 이용해 프레젠테이션 계층을 만든다면 서블릿 애플리케이션 컨텍스트도 사용 않음. 이때는 루트 애플리케이션 컨텍스트만 등록

- 서블릿 컨텍스트 단일구조

  - 서블릿 안에 만들어지는 애플리케이션 컨텍스트가 부모 컨텍스트를 갖지 않기에 스스로 루트 컨텍스트가 된다.

  

##### 루트 애플리케이션 컨텍스트 등록

##### 서블릿 애플리케이션 컨텍스트 등록

스프링의 웹 기술을 지원하는 프론트 컨트롤러 서블릿은 DispatcherServlet이다. web.xml에 등록해서 사용할 수 있는 평범한 서블릿이다. 서블릿 이름을 다르게 지정해주면 하나의 웹 앱에 여러 개의 DispatcherServlet을 등록도 가능. 각 DispatcherServlet은 서블릿이 초기화될 때 자신만의 컨텍스트를 생성하고 초기화한다. 동시에 웹 애플리케이션 레벨에 등록된 루트 애플리케이션 컨텍스트를 찾아서 이를 자신의 부모 컨텍스트로 사용.

DispatcherServlet을 등록할 때 신경 써야 할 사항 두가지

1. `<servlet-name> `

   1. 독립적인 네임스페이스. 
   2. `.WEB-INF/` + `서블릿네임스페이스 + '.xml'

   루트 앱 컨테스트는 서비스 계층과 데이터 액세스 계층의 빈을 모두 포함하고 있고, 그 외에도 각종 기반 서비스와 기술 설정을 갖고 있다. 따라서 설정파일을 여러 개로 구분해두고 디폴트 설정파일 위치 대신 `<context-param>`으로 지정된 설정파일 위치를 사용하는 경우가 많다.

2. `<load-on-startup>` 

   서블릿 컨테이너가 등록된 서블릿을 언제 만들고 초기화할지, 또 그 순서는 어떻게 되는지를 지정하는 정수 값

## IoC/DI를 위한 빈 설정 메타정보 작성

컨테이너는 어떻게 자신이 만들 오브젝트가 무엇인지 알 수 있을까? 컨테이너는 빈 설정 메타정보를 통해 빈의 클래스와 이름을 제공받는다. 빈을 만들기 위한 설정 메타정보는 파일이나 애노테이션 같은 리소스로부터 전용 리더를 통해 읽혀서 `BeanDifinition`타입의 오브젝트로 변환된다. 이 BeanDifinition 정보를 IoC 컨테이너가 활용하는 것이다.

| 이름                      | 내용                                                         | 디폴트값       |
| ------------------------- | ------------------------------------------------------------ | -------------- |
| beanClassName             | 빈 오브젝트의 클래스 이름                                    | 없음. 필수항목 |
| parentName                | 빈 메타정보를 상속받을 부모 BeanDefinition의 이름            |                |
| factoryBeanName           | 팩토리 역할을 하는 빈을 이용해 빈 오브젝트를 생성하는 경우에 팩토리 빈의 이름을 지정한다. |                |
| factoryMethodName         | 다른 빈 또는 클래스의 메소드를 통해 빈 오브젝트를 생성하는 경우 그 메소드 이름 지정 |                |
| scope                     | 생명주기                                                     | 싱글톤         |
| lazyInit                  | 빈 오브젝트의 생성을 최대한 지연할 것인지                    | false          |
| dependsOn                 |                                                              |                |
| autowireCandidate         |                                                              | true           |
| primary                   | 자동 와이어링 중 DI 대상 후보 여러개일 경우 최종선택 우선권 부여 | false          |
| abstract                  | 메타정보 상속에만 사용할 추상 빈으로 만들지의 여부           | false          |
| autowireMode              | 오토와이어링 전략. 이름, 타입, 생성자, 자동인식 등의 방법    |                |
| dependencyCheck           |                                                              |                |
| initMethod                |                                                              |                |
| destoryMethod             |                                                              |                |
| propertyValues            |                                                              |                |
| constructorArgumentValues |                                                              |                |
| annotationMetadata        |                                                              |                |

### 빈 등록 방법

스프링에서 자주 사용되는 빈의 등록방법은 크게 5가지가 있다.

1. XML : <bean> 태그

2. XML : 네임스페이스와 전용 태그

   ```xml
   <aop:pointcut id="mypointcut" expression="execution(* *..*ServiceImpl.upgrade*)(..)>"/>
   ```

   ​	aop 스키마가 있기 때문에 XML 문서 편집 중에도 즉시 애트리뷰트의 타입과 필수 사용 여부 등을 검증할 수 있다.

   ​	전용 태그 하나로 동시에 여러 개의 빈을 만들 수 있다는 장점도 있다. `<context:annotation-config>`

3. 자동인식을 이용한 빈 등록 : 스테레오타입 애노테이션과 빈 스캐너

   빈 스캐너에 내장된 디폴트 필터는 `@Component`와 같은 **스테레오타입<sup>stereotype</sup>** 애노테이션이 부여된 클래스를 선택하도록 되어있다. 

   ##### XML을 이용한 빈 스캐너 등록

   ​	XML 설정 파일 안에 context 스키마의 전용 태그 `<context:component-scan>`를 넣어서 간단히 빈 스캐너 등록

```xml
<context:component-scan base-package="ssg.framework.support">
	<context:exclude-filter type="annotation" expression="ssg.framework.support.spring.annotation.LazyBinding"/>
</context:component-scan>
```

##### 		빈 스캐너를 내장한 애플리케이션 컨텍스트 사용

​			XML에 빈 스캐너를 지정하는 대신 아예 빈 스캐너를 내장한 컨텍스트를 사용하는 방법

```xml
<context-param>
	<param-name>contextClass</param-name>
	<param-value>org.springframework.web.context.support.AnnotationConfigWebApplicationContext</param-value>
</context-param>
```

- 빈 클래스 자동인식에는 `@Component` 외에도 여러 스테레오타입 애노테이션 사용 가능. 여러 애노테이션을 사용하는 이유는 계층별로 빈의 특성이나 종류를 나타내려는 목적도 있고, AOP의 적용 대상 그룹을 만들기 위해서이기도 하다. AOP 포인트컷 표현식을 사용하면 특정 애노테이션이 달린 클래스만을 선정 가능. 특정 계층 빈에 부가기능 부여 가능

| 스테레오타입 애노테이션 | 적용 대상                                                    |
| ----------------------- | ------------------------------------------------------------ |
| `@Repository`           | 데이터 액세스 계층의 DAO 또는 Repository 클래스에 사용된다. DataAccessException 자동변환과 같은 AOP의 적용 대상을 선정하기 위해서도 사용된다. |
| `@Service`              | 서비스 계층의 클래스에 사용                                  |
| `@Controller`           | 프레젠테이션 계층의 MVC컨트롤러에 사용된다. 스프링 웹 서블릿에 의해 웹 요청을 처리하는 컨트롤러 빈으로 선정된다. |

##### 		자바 코드에 의한 빈 등록 : @Configuration 클래스의 @Bean 메소드

```java
@Configuration
public class ServiceConfig{
    @Bean
    public DataSource dataSource(){
        SimpleDriverDataSource dataSource = new SimpleDriverDataSource();
        dataSource.setDriverClass(com.mysql.jdbc.Driver.class);
        dataSource.setUrl("jdbc:mysql://localhost/testdb");
        
        return dataSource;
    }
    //...
}
```

@Configuration이 아닌 일반 빈 크래스에서 @Bean 하면 싱글톤이 안됨

### 빈 의존관계 설정 방법

