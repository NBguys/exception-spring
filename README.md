# exception-spring
서블릿 예외 처리 - 오류 페이지 작동 원리
오류 페이지 요청 흐름
```
WAS `/error-page/500` 다시 요청 -> 필터 -> 서블릿 -> 인터셉터 -> 컨트롤러(/error-page/500) -> View
```

예외 발생과 오류 페이지 요청 흐름 
```
1. WAS(여기까지 전파) <- 필터 <- 서블릿 <- 인터셉터 <- 컨트롤러(예외발생)
2. WAS `/error-page/500` 다시 요청 -> 필터 -> 서블릿 -> 인터셉터 -> 컨트롤러(/error-page/500) -> View
```

DispatcherType
```
오류가 발생하면 오류 페이지를 출력하기 위해 WAS 내부에서 다시 한번 호출이 발생한다. 이때 필터, 서블릿, 인터셉터
도 모두 다시 호출된다. 불필요한 호출을 막기 위해 요청에 대한 구분을 알 수 있게 해주는 메소드
REQUEST : 클라이언트 요청
ERROR : 오류 요청
FORWARD : MVC에서 배웠던 서블릿에서 다른 서블릿이나 JSP를 호출할 때
RequestDispatcher.forward(request, response);
INCLUDE : 서블릿에서 다른 서블릿이나 JSP의 결과를 포함할 때
RequestDispatcher.include(request, response);
ASYNC : 서블릿 비동기 호출
```

스프링 부트 - 오류 페이지
```
지금까지 예외 처리 페이지를 만들기 위해서 다음과 같은 복잡한 과정을 거쳤다.
WebServerCustomizer 를 만들고
예외 종류에 따라서 ErrorPage 를 추가하고
예외 처리용 컨트롤러 ErrorPageController 를 만듬
스프링 부트는 이런 과정을 모두 기본으로 제공한다.
ErrorPage 를 자동으로 등록한다. 이때 /error 라는 경로로 기본 오류 페이지를 설정한다.
new ErrorPage("/error") , 상태코드와 예외를 설정하지 않으면 기본 오류 페이지로 사용된다.
서블릿 밖으로 예외가 발생하거나, response.sendError(...) 가 호출되면 모든 오류는 /error 를
호출하게 된다. 
BasicErrorController 라는 스프링 컨트롤러를 자동으로 등록한다.
ErrorPage 에서 등록한 /error 를 매핑해서 처리하는 컨트롤러다.
```
BasicErrorController 컨트롤러는 다음 정보를 model에 담아서 뷰에 전달한다. 뷰 템플릿은 이 값을 활용해서
출력할 수 있다
```
* timestamp: Fri Feb 05 00:00:00 KST 2021
* status: 400
* error: Bad Request
* exception: org.springframework.validation.BindException
* trace: 예외 trace
* message: Validation failed for object='data'. Error count: 1
* errors: Errors(BindingResult)
* path: 클라이언트 요청 경로 (`/hello`)
```
BasicErrorController 설정
```
application.properties
server.error.include-exception=false : exception 포함 여부( true , false )
server.error.include-message=never : message 포함 여부
server.error.include-stacktrace=never : trace 포함 여부
server.error.include-binding-errors=never : errors 포함 여부
```

기본 값이 never 인 부분은 다음 3가지 옵션을 사용할 수 있다
```
never : 사용하지 않음
always :항상 사용
on_param : 파라미터가 있을 때 사용
on_param 은 파라미터가 있으면 해당 정보를 노출한다. 디버그 시 문제를 확인하기 위해 사용할 수 있다. 그런데 이
부분도 개발 서버에서 사용할 수 있지만, 운영 서버에서는 권장하지 않는다.
on_param 으로 설정하고 다음과 같이 HTTP 요청시 파라미터를 전달하면 해당 정보들이 model 에 담겨서 뷰 템플
릿에서 출력된다.
ex) localhost:8080/error-ex?message=&errors=&trace=
```

스프링 부트 오류 관련 옵션
```
server.error.whitelabel.enabled=true : 오류 처리 화면을 못 찾을 시, 스프링 whitelabel 오류 페이지 적용
server.error.path=/error : 오류 페이지 경로, 스프링이 자동 등록하는 서블릿 글로벌 오류 페이지 경로
와 BasicErrorController 오류 컨트롤러 경로에 함께 사용된다
```