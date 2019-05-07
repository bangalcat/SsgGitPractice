# 13장 스프링 @MVC

## @RequestMapping 핸들러 매핑

### 클래스/메소드 결합 매핑정보

@RequestMapping은 타입 레벨 뿐 아니라 메소드 레벨도 붙일 수 있다.

```java
@RequestMapping("/admin/**/user")
@RequestMapping("/user/{userid}")
@RequestMapping({"/hello","/hi"})
```

- RequestMethod[] method() : HTTP 요청 메소드

```java
@RequestMapping(value="/user/add", method=RequestMethod.GET)
```



##### 제네릭스와 매핑정보 상속을 이용한 컨트롤러 작성

```java
public abstract class GenericController<T, P, S>{
 	S service;
    @RequestMapping("/add") public void add(T entity){}
    @RequestMapping("/update") public void update(T entity){}
    @RequestMapping("/view") public T view(P id){}
    @RequestMapping("/delete") public void delete(P id){}
    @RequestMapping("/list") public List<T> list(){}
}
@RequestMapping("/user")
public class UserController extends GenericController<User, Integer, UserService> {
    @RequestMapping("/login")
    public String login(String userId, String password) {}
}
```

> 스프링을 기반으로 제네릭스를 모든 계층에 적용해서 고속 개발이 가능하도록 만든 프레임워크를 살펴보고 싶다면 OSAF(http://whiteship.me/category/OSAF)의 소스코드를 참고해보기 바란다

## @Controller

다른 핸들러 어댑터들은 컨트롤러가 구현하고 있는 인터페이스의 실행 메소드를 알고 있기 때문에 이를 이용해 간단히 메소드를 호출할 수 있었다. 하지만 @MVC의 컨트롤러는 특정 인터페이스를 구현하지 않는다. 게다가 메소드의 이름이나 파라미터 개수와 타입, 리턴 타입도 정해져 있지 않다. 그렇다면 어떻게 AnnotationMethodHandlerAdapter는 이런 메소드를 컨트롤러 메소드로 사용할 수 있는 것일까?

```java
//복잡한 @Controller 메소드
@RequestMapping("/complex")
public String complex(@RequestParam("name") String name, @CookieValue("auth")String auth, ModelMap model){
    model.put("info", name+"/"+auth);
    return "myview";
}
```

리턴 타입이 스트링이면 스프링은 관례에 따라서 이 값을 뷰 이름으로 사용한다.

##### @PathVariable

@ReqeustMapping의 URL에 {}로 들어가는 패스 변수를 받는다. 요청 파라미터를 URL의 쿼리 스트링으로 보내는 대신 URL 패스로 풀어서 쓰는 방식을 쓰는 경우 매우 유용하다.

##### @RequestParam

단일 HTTP 요청 파라미터를 메소드 파라미터에 넣어주는 애노테이션이다.

##### @ModelAttribute

RequestParam은 하나씩 가져오게 된다. Map으로 가져올 수도 있지만 key값을 알아야한다. 하나의 객체로 그냥 파라미터들을 받을 수 있다면?

```java
@RequestMapping("/user/search")
public String search(@ModelAttribute UserSearch userSearch){
    List<User> list = userService.search(userSearch);
    model.addAtribute("userList", list);
    //...
}
```

### 리턴 타입의 종류

컨트롤러가 DispatcherServlet에 돌려줘야 하는 정보는 모델과 뷰다. 최종적으로 ModelAndView 타입으로 리턴값이 전달된다.

###### 자동 추가 모델 오브젝트와 자동생성 뷰 이름

- @ModelAttribute 모델 오브젝트 또는 커맨드 오브젝트
- Map, Model ModelMap 파라미터
- @ModelAttribute 메소드
- BindingResult