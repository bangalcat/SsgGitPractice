# Spring - properties

## property 설정 파일 (.properties와 xml)

### 0. 들어가기 전에

.properties 같은 파일을 이용해서 데이터를 이용하는 경우는 많이 있습니다. 이것과 비슷하게 xml을 이용해서도 데이터를 이용하는 경우도 많이 있습니다. 어떤 방법이 더 좋거나 큰 차이가 있는건 아니지만, 서로 변환이 가능하기에 한번 해보았습니다. jdbc 파일을 보다가 생각난 것이므로 예제도 jdbc로 해봤습니다.

### 1. properties

```properties
 mysql.jdbc.driver = com.mysql.jdbc.Driver
 mysql.jdbc.url = jdbc:mysql://127.0.0.1:3306/test
 mysql.jdbc.username = user
 mysql.jdbc.password = test123
```

### 2. xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE properties SYSTEM "http://java.sun.com/dtd/properties.dtd" >

<properties>
 <comment>jdbc</comment>
 <entry key="mysql.jdbc.driver">com.mysql.jdbc.Driver</entry>
 <entry key="mysql.jdbc.url">jdbc:mysql://127.0.0.1:3306/test</entry>
 <entry key="mysql.jdbc.username">user</entry>
 <entry key="mysql.jdbc.password">test123</entry>
</properties>
```


xml로 전환하기 위해서는 document 를 선언해 주어야 하며 comment와 entry key를 필수적으로 써주어야 합니다.

1, 2번의 모양대로 하면 properties 와 xml은 서로 변환이 가능합니다.



**[출처]** [[Spring\] properties 파일을 xml로 변경하는 방법](http://blog.naver.com/platinasnow/220261460882)|**작성자** [심해펭귄](http://blog.naver.com/platinasnow)



## Spring 프로젝트에서 Property를 이용할 수 있는 방법

### 1.  xml

property-placeholder를 사용

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
   xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
   xmlns:context="http://www.springframework.org/schema/context"
   xsi:schemaLocation="
      http://www.springframework.org/schema/beans
      http://www.springframework.org/schema/beans/spring-beans-4.2.xsd
      http://www.springframework.org/schema/context
      http://www.springframework.org/schema/context/spring-context-4.2.xsd">

      <context:property-placeholder location="classpath:foo.properties" />

</beans>
```

다수의 property-placeholder를 선언할 경우 order속성을 지정하여 스프링에 의해 처리되는 순서를 지정할 필요가 있다. 또한 마지막 속성을 제외한 모든 속성의 순서를 지정하고 `ignore-unresolvable="true"`를 함께 작성해야만 예외를 던지지 않는다.

Java Annotation을 사용한 property 등록

```java
@Configuration
@PropertySource("classpath:foo.properties")
public class PropertiesWithJavaConfig {

   @Bean
   public static PropertySourcesPlaceholderConfigurer
     propertySourcesPlaceholderConfigurer() {
      return new PropertySourcesPlaceholderConfigurer();
   }
}
```

