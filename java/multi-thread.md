# Multi Thread

## LinkedBlockingQueue
Java의 LinkedBlockingQueue는 BlockingQueue 인터페이스의 구현으로, thread-safe한 큐다. 이 큐는 연결 노드(linked nodes)를 사용하여 요소를 저장한다. 이로 인해 큐의 용량은 필요에 따라 늘어날 수 있으며, 정적 배열을 사용하는 ArrayBlockingQueue와 달리 초기에 크기를 정해놓지 않아도 된다.

LinkedBlockingQueue의 주요 특징은 다음과 같다.

- Thread-Safe: LinkedBlockingQueue는 내부적으로 락(locking)을 사용하여 여러 스레드에서 동시에 접근해도 데이터의 일관성이 유지된다.

- 블로킹(Blocking) 연산: 큐가 비어있을 때 요소를 꺼내려고 하면 스레드는 큐에 요소가 들어올 때까지 대기한다. 마찬가지로, 큐에 공간이 없을 때(설정된 용량에 도달한 경우) 요소를 넣으려고 하면 스레드는 큐에 공간이 생길 때까지 대기한다. 이러한 블로킹 특성 때문에 프로듀서-컨슈머 패턴에서 유용하게 사용된다.

- 옵션으로 용량 제한: LinkedBlockingQueue를 생성할 때, 큐의 최대 크기를 지정할 수 있다. 크기를 지정하지 않으면 Integer.MAX_VALUE가 기본값으로 사용되어 사실상 무제한 크기로 동작한다.

- FIFO (First-In-First-Out): LinkedBlockingQueue는 FIFO 데이터 구조

```Java
import java.util.concurrent.LinkedBlockingQueue;

public class Example {
    public static void main(String[] args) throws InterruptedException {
        LinkedBlockingQueue<String> queue = new LinkedBlockingQueue<>(3); // 용량을 3으로 설정

        // 큐에 요소 추가
        queue.put("A");
        queue.put("B");
        queue.put("C");

        // 큐에서 요소 제거
        String element = queue.take();
        System.out.println("Removed element: " + element);

        // 큐의 상태 출력
        System.out.println("Queue contents: " + queue);
    }
}
```


## Thread starvation or clock leap detected 
멀티 쓰레딩 작업 중에 아래와 같은 메시지가 발생했다.

`com.zaxxer.hikari.pool.HikariPool : HikariPool-1 - Thread starvation or clock leap detected (housekeeper delta=15m30s283ms).` 

찾아 보니, Thread starvation 현상이 발생한 것은 아니고, 로컬에서 실행하는 경우 PC가 sleep 에 빠진 경우 발생하는 증상이었다.

참고) https://stackoverflow.com/questions/38703876/log-warning-thread-starvation-or-clock-leap-detected-housekeeper-delta-springh
