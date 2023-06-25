# python의 GC
cpython 기준의 설명이다.

 python의 GC는 기본적으로 2가지 측면이 있다.
 1. Reference counting
 2. Generational garbage collection

## Reference counting
python 에서는 객체를 만들 때마다 reference count가 생성된다. 해당 객체가 참조될 때마다 count가 증가하고, 참조가 해제될 때 감소한다. 객체의 reference count 가 0이 되면 객체의 메모리 할당이 해제된다.

그러나 reference count 갱신이 어려운 경우가 있다. 
```python
a = []
a.append(a)
del a
```
위 코드와 같이 동적 객체가 서로를 참조 하고 있다면, a의 reference count 는 1이지만 해당 객체는 더 이상 접근할 수 없다. 이러한 객체들은 GC 에 의해 정리가 된다.

## Generational garbage collection
GC는 메모리의 모든 객체를 추적한다. python 역시 *가설은 대부분의 객체는 빠르게 사용되지 않게 되며, 오래된 객체 중 일부만이 계속해서 사용된다는 것을 전제*로 하는 Generational Hypothesis를 기반으로 GC를 수행한다.

python 은 generation 을 3세대로 분리하는데, 새로 생성된 객체는 0세대에 배치되고 GC에서 살아남은 객체는 다음 세대로 한 칸씩 이동하며 2세대가 가장 오래된 세대이다. 낮은 세대일 수록 GC가 더 빈번하게 발생한다. 

### 그럼 각 세대 별로 GC가 언제 발생한다는 것일까?

GC가 발생하는 기준은 threshold와 관련이 있다.
```python
>>> gc.get_threshold()
(700, 10, 10)
```
각각 `threshold 0`, `threshold 1`, `threshold 2` 를 의미한다. (값은 당연히 변경이 가능하다.)

0세대의 경우 메모리에 객체가 할당된 횟수에서 해제된 횟수를 뺀 값, 즉 객체 수가 `threshold 0`을 초과하면 실행된다. 다만 그 이후 세대부터는 조금 다른데 0세대 GC가 일어난 후 0세대 객체를 1세대로 이동시킨 후 카운터를 1 증가시킨다. 이 1세대 카운터가 `threshold 1`을 초과하면 그때 1세대 GC가 일어난다.

위에서 GC가 발생하면 순환 참조를 탐지하여 메모리를 정지한다고 했다. 그러면 어떻게 순환 참조를 탐지할 수 있을까?

순환 참조는 컨테이너 객체(e.g. tuple, list, set, dict, class)에 의해서만 발생한다. 컨테이너 객체들만이 다른 객체의 참조를 보유할 수 있기 때문이다.

순환 참조 해결을 위해 아래의 순서로 동작한다.

1. 각 객체의 `gc_refs` 필드를 레퍼런스 카운트와 같게 설정
2. 각 객체에서 참조하고 있는 다른 컨테이너 객체를 찾고, 참조되는 컨테이너의 `gc_refs`를 감소시킨다.
3. 어느 객체의 `gc_refs가` 0이 되면, 그 객체는 컨테이너 집합 내부에서 자기들끼리 참조하고 있다는 뜻이다.
4. 해당 객체를 메모리에서 해제한다.

이런 식으로 **모든** 객체에 대해서 reference graph를 그리며 접근하는 방식은 성능이 느릴 수 밖에 없기 때문에 GC의 threshold를 조정하는 방식으로 튜닝하며 사용할 수 있다. Instagram 에서는 Python Django 서버에서 GC를 비활성화 하며 운영하기도 한 만큼, GC를 컨트롤 하는 것은 하나의 옵션으로 존재한다.

### python 의 멀티 쓰레드
python 에는 GIL 이라는 일종의 mutex 가 존재하여, 여러 쓰레드가 CPU bound task를 수행하는 것을 막아 두었다. 이는 cpython이 메모리를 관리하는 방식인 reference counting과 관련이 있는데, 위에서 언급한 것 처럼 모든 객체에 대해 참조 횟수를 tracking 해야 하는데, 여러 개의 쓰레드가 인터프리터를 동시에 실행하면 Race Condition이 발생할 수 있어, 모든 객체에 대해 Lock 을 걸어야 한다. 이런 비효율을 피하기 위해 GIL이라는 녀석을 두어 아예 하나의 쓰레드만 실행이 가능하도록 해 둔 것이다.