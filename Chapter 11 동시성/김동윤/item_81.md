# wait와 notify보다는 java.util.concurrent의 동시성 유틸리티를 애용하라
- wait와 notify
wait와 notify는 올바르게 사용하기가 아주 까다로우니, java.util.concurrent의 고수준 동시성 유틸리티를 사용

## wait(), notify(), notifyAll()
- synchronized로 동기화해서 공유 데이터를 보호할때 특정 스레드가 객체의 락을 가진 상태로 오랜 시간을 보내지 않도록 하는것도 중요
- 락을 오랜시간 보유하게되면, 다른 스레드들은 모두 해당 객체의 락을 기다리느라 다른 작업들도 원할히 진행 X
- 이러한 상황을 위해 고안된 것이 wait), notify()

### wait()
- 동기화된 임계 영역의 코드를 수행하다가 더이상 작업을 진행할 상황이 아니라면, 일단 wait()을 호출하여 스레드가 락을 반납하고 기다리게 해
- 그러면 다른 스레드가 락을 얻어 해당 객체에 대한 작업을 수행

> wait()은 notify() 또는 notifyAll()이 호출될때까지 기다린다.

### notify()
- 나중에 작업을 진행할 수 있는 상황이 되면 notify()를 호출해서, 작업을 중단했던 스레드가 다시 락을 얻어 작업을 진행할 수 있게해


## 동기화 장치(synchronizer)
- 동기화 장치는 스레드가 다른 스레드를 기다릴 수 있게 하여 서로 작업을 조율할 수 있게 해준다.
(ex) CountDownLatch, Semaphore, CyclicBarrier, Exchanger, Phaser

### CountDownLatch
- CountDownLatch는 하나 이상의 스레드가 또 다른 하나 이상의 스레드 작업이 끝날 때까지 기다리게 한다.

## wait와 notify를 사용해야할 때
- 동시성 유틸리티를 사용하는게 옳지만, 어쩔 수 없이 레거시 코드를 다뤄야할 때도 있다.
- wait 메소드를 사용할 땐 반드시 대기 반복문(wait loop) 관용구를 사용하고 반복문 밖에서는 절대 호출하지 말자.
   - 반복문은 wait 호출 전후로 조건이 만족하는지 검사하는 역할을 한다.
 
### notify vs notifyAll

- notify는 스레드 하나만 깨우며, notifyAll은 모든 스레드를 깨운다. 일반적으론 언제나 notifyAll을 사용하는게 낫다. 깨어나야 하는 모든 스레드가 깨어남을 보장하니 항상 정확한 결과를 얻을 수 있을 것이고, 다른 스레드까지 깨어날 수도 있긴 하지만 정확성에는 영향을 주지 않는다. 위에서 했던 것처럼 기다리던 조건이 충족되었는지 확인해 충족되지 않았다면 다시 대기할테니 말이다.
- 모든 스레드가 같은 조건을 기다리고, 조건이 한 번 충족될 때마다 단 하나의 스레드만 헤택을 받는다면 notify를 사용해 최적화할 수도 있지만 외부로 공개된 객체에 대해 실수 혹은 악의적으로 notify를 호출하는 상황 대비를 위해 wait를 반복문 안에서 호출했듯, notifyAll을 사용하면 관련없는 스레드에서 실수 혹은 악의적으로 wait를 호출하는 공격으로부터 보호할 수 있다

### reference
https://ktaes.tistory.com/87
https://devfunny.tistory.com/854?category=895441