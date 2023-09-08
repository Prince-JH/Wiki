## ConcurrentHashMap

ConcurrentHashMap은 HashTable과 유사하게, thread-safe 하다는 성질을 갖고 있다.

그러나 HashTable은 전체 맵에 락(lock)을 거는 반면, ConcurrentHashMap은 특정 세그먼트에만 락을 걸어 동시성을 제공하면서도 우수한 성능을 가진다.

```Java
import java.util.concurrent.ConcurrentHashMap;

public class ConcurrentHashMapExample {
    public static void main(String[] args) {
        ConcurrentHashMap<String, Integer> map = new ConcurrentHashMap<>();

        map.put("one", 1);
        map.put("two", 2);
        map.put("three", 3);

        // 값 획득
        int value = map.get("one");

        // 값 제거
        map.remove("one");

        // 값 변경
        map.replace("two", 22);
    }
}
```