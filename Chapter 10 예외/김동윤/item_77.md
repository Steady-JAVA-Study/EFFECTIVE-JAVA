# 예외를 무시하지 말라

```java
// catch 블록을 비워두면 예외가 무시된다. 
try {
    ...
}catch(SomeException e){
 
}
```
- 예외 처리란, 예외 상황에 적절한 처리를 하기 위해 존재 → 무시하지 말자.
- catch 블록 비워두기
try { 
- 위 코드처럼 catch 블록에서 아무일도 하지 않으면 예외가 무시 
- 예외는 문제 상황에 잘 대처하기 위해 존재하는데 catch 블록을 비워두면 예외가 존재할 이유가 없어


### 예외 무시 
- 다만 FileInputStream 을 닫을 때는 입력 전용 스트림이므로 파일의 상태를 변경하지 않았으니 복구할 것이 없으며, 스트림을 닫는다는 건 필요한 정보는 이미 다 읽었다는 뜻이므로 남은 작업을 중단할 이유도 없다. 
=> 이럴 경우 예외를 무시해야 한다.
- 예외를 무시하기로 했다면 catch 블록 안에 이유를 주석으로 남기고 예외 변수의 이름을 ignored로 바꿔라


```java
Future<Integer> f = exec.submit(planarMap::chromatioNumber);
int numColors = 4; // 기본값. 어떤 지도라도 이 값이면 충분하다.
try {
    numColors = f.get(1L, TimeUnit.SECONDS);
} catch(TimeoutException | ExecutionException ignored) {
    // 기본 값을 사용한다.
}

```

### 예외를 무시해도 되는 경우
- FileInputStream을 닫는 경우
   - 읽기 전용이고 파일의 상태를 변경하지 않았으니 복구할 것이 존재하지 않는다.
   - 스트림을 닫는 것은 필요한 정보를 다 읽었다는 뜻이니 작업을 중단할 이유도 없다.
   - 무시하기로 했다면 catch 블록 안에 이유를 명시하고 예외 변수의 이름도 ignored로 변경해야 한다.


## 결론

- 예외를 무시하는 경우가 저 예시 밖에 없는 걸 보아하니 예외는 절대 무시하면 안된다.
- 예외가 발생하면 이유와 함께 던지도록 하고 상황에 따라 추상화하여 던져야 한다.
- 최소한 발생한 예외를 전파되도록 코드를 작성해야 한다.








## reference
https://greeng00se.tistory.com/m/102
https://codingwell.tistory.com/160




