# 스프링 MVC 패턴의 흐름

![img](https://t1.daumcdn.net/cfile/tistory/2247684E565C021913)

> 웹브라우저 -> DispatcherServlet -> Controller -> Model and View -> Controller -> DispatcherServlet -> Web Browser


1. 웹브라우저에게 정보요청을 받은 디스패처서블릿은 어느 컨트롤러에 해당 요청을 전송할지 결정
2. 디스패처 서블릿은 핸들러 매핑에 어느 컨트롤러를 사용할건지 물어봄(URL로 링크)
3. 결정된 컨트롤러는 해당요청을 수행하게 됨
4. 해당요청을 처리한 컨트롤러는 디스패처서블릿에 결과를 보냄. 이 과정에서 Model이 생성되어 View(JSP)에서 같이 사용됨
5. Model And View는 실제 JSP 정보를 갖고 있지 않기 때문에 뷰리졸버가 실제 JSP이름으로 변환하여 해당 view를 검색함.
6. 검색한 결과를 View에 전송
7. View는 모든 과정에서 처리된 결과를 화면으로 표현함
8. 마지막으로 디스패처서블릿이 웹브라우저에 최종결과를 출력



## Spring DispatcherServlet

Spring은 이러한 MVC 패턴을 구현하기 위해 다른 웹 MVC프레임워크처럼 Front Controller 패턴을 사용하며 프레임워크의 여러 가지 기능을 제공하는 servlet 중심으로 설계되어 잇다. **스프링에서 Front Controller 역할을 하는것이 바로 DispatcherServlet이다.** 스프링의 DispatcherServlet은 단순히 Front Controller 기능만 하는 게 아니라 스프링 IoC 컨테이너와 완전히 통합되어 스프링이 가진 모든 다른 기능을 사용할 수 있게 한다. 

> FrontController 패턴이란? 모든 Web Application에 대한 요청들을 Front Controller로 받고 그 요청을 Controller로 분배해주는 패턴을 의미

