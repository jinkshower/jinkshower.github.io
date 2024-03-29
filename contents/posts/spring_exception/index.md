---
title: "AOP를 통한 Spring 예외처리"
description: "Spring의 AOP 예외 처리로 코드를 개선한 기록"
date: 2024-01-24
update: 2024-01-24
tags:
  - spring
  - exception
  - exceptionhandler
series: "spring"
---

개인과제를 진행하며 Spring의 AOP 예외 처리로 코드를 개선한 기록.

## 과제에서 마주한 문제

개인 과제를 진행하는 도중 Entity를 수정할때 비밀번호가 다를 경우 예외를 던지는 코드를 작성하게 되었다.
```java
@Transactional
public ScheduleResponseDto updateSchedule(Long id, ScheduleRequestDto scheduleRequestDto) {
    Schedule schedule = findSchedule(id);
    validatePassword(schedule.getPassword(), scheduleRequestDto.getPassword());

    schedule.update(scheduleRequestDto);
    return new ScheduleResponseDto(schedule);
}

private void validatePassword(String origin, String input) {  
    if(!origin.equals(input)) {  
        throw new IllegalArgumentException("[ERROR] 패스워드가 다릅니다");  
    }  
}
```
이 애플리케이션이 Java로 돌아가는 커맨드라인 프로그램이었다면 종료되었을 것이다.

하지만 Spring에서 별다른 예외처리를 하지 않고 그냥 throw로 던지는 예외의 경우 500상태코드를 반환하게 된다. 

`사용자` 가 알맞은 비밀번호를 입력하지 않았는데, 500 상태코드가 반환된다.
서버 에러 메시지를 보고 계속 같은 요청을 하는 사용자도 생길테고, 이는 전반적인 사용자 경험에 큰 악영향을 미친다
또한 사용자는 자신의 요청에 대한 피드백을 제대로 받지 못한다.

## 첫 시도, Early Return

```java
@Transactional  
public ResponseEntity<String> updateSchedule(Long id, ScheduleRequestDto requestDto) {  
    Schedule schedule = findSchedule(id);  
    if (!schedule.getPassword().equals(requestDto.getPassword())) {  
        return ResponseEntity.badRequest().body("비밀번호가 다릅니다");  
    }   // early return

    schedule.update(requestDto);  
    return ResponseEntity.ok().body("수정 완료!");  
}
```
수정하는 메서드에서 검증을 진행하고, 비밀번호가 다를 경우 상태코드와 함께 메시지를 전달하게 바꾸어 보았다.

하지만 해당 방법은 
1. 검증이 필요한 모든 메서드에 early return을 적용해야 하고,
2. 검증 로직에 변경이 있을 경우 응답객체의 생성방식도 변경해야 한다.
3. 더불어 Service에서 응답객체를 만들기 때문에 Controller의 역할이 희미해진다 (싱크홀 안티패턴의 가능성)

## 두번째 시도, ResponseStatusException

```java
private void validatePassword(String origin, String input) {  
    if(!origin.equals(input)) {  
        throw new ResponseStatusException(HttpStatus.BAD_REQUEST, "[ERROR] 패스워드가 다릅니다");
    }  
}
```
Spring에서 제공하는 `ResponseStatusException`을 이용해 보았다. 
파라미터로 HttpStatus와 String의 메시지를 넣을 수 있으니 500 상태코드 대신 내가 원하는 코드와 메시지를 전달할 수 있다.

하지만 위의 방법으로는 내가 직접 예외처리 코드를 작성하지 않은 다른 예외는 처리할 수 없다. 

(내 의견으로는) 500 상태 코드는 정말 서버내부에서 크리티컬한 문제가 났을 때만 보여줘야한다고 생각한다. 

즉, 내가 해당방법으로 하나하나 예외처리 해준 케이스 이외에 사용자의 잘못된 요청으로 예외가 발생하면 여전히 500 상태코드가 보여진다. 

## 세번째 시도, @ExceptionHandler

```java
@ExceptionHandler(RuntimeException.class)  
public ResponseEntity<String> handleException() {  
    //..  
    return ResponseEntity.badRequest().body("[ERROR] 잘못된 입력입니다");  
}
```
해당 코드를 @Controller에 적용했다.

@ExceptionHandler에 예외클래스를 명시한 후 @Controller에서 지정된 예외 클래스 하위의 예외가 발생시 어노테이션이 붙은 메서드를 실행하게 했다. 

이를 통해 직접 예외처리한 로직 이외에도 예외처리를 할 수 있게 되었고
무엇보다도 500상태코드를 반환할 예외를 직접 지정할수 있게되었다. 

이 코드를 좀 더 확장성있게 쓸수 있지 않을까? 

## 확장하기, @ContollerAdvice

```java
@ControllerAdvice  
public class ControllerAdvice {  
    @ExceptionHandler  
    public ResponseEntity<String> handleException(Exception e) {  
        return ResponseEntity.internalServerError().body(e.getMessage());  
    }  
@ExceptionHandler(IllegalArgumentException.class)  
    public ResponseEntity<String> handleIllegalArgumentException(IllegalArgumentException e) {  
        return ResponseEntity.badRequest().body(e.getMessage());  
    }  
}
```

@ControllerAdvice를 어노테이션으로 가지는 새로운 클래스를 작성했다.

해당 어노테이션은 AOP(Aspect Oriented Programming)의 방식으로 예외처리를 많은 컨트롤러에서 `공통`으로 처리해야할 요소로 보고 @Controller어노테이션을 가진 모든 컨트롤러에서 발생하는 예외에 앞서 적용한 @ExceptionHandler의 예외처리 로직을 적용해 준다.

코드를 보면 알수 있듯이 여러가지 Exception에 대해 어떤 응답을 보내줄지 메서드를 추가해서 확장할 수 있으므로 우리는 Java코드를 작성하는 것처럼 예외처리를 할 수 있게 되었고, 커스텀예외를 추가하는 것도 물론 가능하다. 


---
참고

https://mangkyu.tistory.com/204

https://tecoble.techcourse.co.kr/post/2020-07-28-global-exception-handler/ 

