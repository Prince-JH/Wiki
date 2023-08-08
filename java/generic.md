# generic

## generic and primitive type
제네릭에는 **primitive type을 할당할 수 없다.**

generic은 compile-time feature인데, 이는 compile 시점에 generic 타입 파라미터가 지워지고 Object 타입으로 변경되기 때문이다.

때문에 Object 클래스를 상속하는 클래스 타입만 사용이 가능하다.