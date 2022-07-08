# finalizer와 cleaner(소멸자) 사용을 피하라

### Finalize
> Object에 존재하는 finalize()를 의미한다. 클래스의 객체가 더 이상 사용되지 않으면 GC가 자동으로 호출
### Cleaner
> 별도의 쓰레드를 사용해서 finalizer보다는 덜 위험하지만, 여전히 예측불가하고, 느리며 불필요

- 즉시 수행 보장도 없음 
- 상태 영구적 수정 작업에서는 절대 finalize, cleaner 사용 X
- DB같은 공유자원의 영구 LOCK 해제를 finalizer와 cleaner 에 맡기면 분산 시스템 전체가 서서히 멈춤 

## AutoCloseable
- 클라이언트에서 인스턴스를 다 쓰고 나면 close()를 호출
- 예외가 발생해도 제대로 종료할 수 있게 처리해주는 try-with-resouces를 사용하긔
