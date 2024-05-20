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

