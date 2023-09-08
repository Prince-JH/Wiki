# Multi Thread

## Thread starvation or clock leap detected 
멀티 쓰레딩 작업 중에 아래와 같은 메시지가 발생했다.

`com.zaxxer.hikari.pool.HikariPool : HikariPool-1 - Thread starvation or clock leap detected (housekeeper delta=15m30s283ms).` 

찾아 보니, Thread starvation 현상이 발생한 것은 아니고, 로컬에서 실행하는 경우 PC가 sleep 에 빠진 경우 발생하는 증상이었다.

참고) https://stackoverflow.com/questions/38703876/log-warning-thread-starvation-or-clock-leap-detected-housekeeper-delta-springh
