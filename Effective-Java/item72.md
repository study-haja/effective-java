# Item 72 : 표준 exception을 선호하라.

### 표준 exception 사용의 장점

1. 표준이기때문에, 다른 사람이 API를 익히기 쉽다.

2. Exception 클래스의 종류가 적어지면 memory footprint가 줄어들고 클래스를 로딩하는데 시간이 덜 걸린다.

   

### 주의할점

1. ```Exception```, ```RuntimeException```, ```Throwable```, ```Error``` 클래스들은 다른 exception들의 super class 이기때문에 직접 사용해서는 안된다.
2. 표준 exception에서 추가로 구현을 하고 싶다면 상속을 받아서 구현을 해도 된다. 
3. Exception은 serializable 이기 때문에 꼭 필요한 경우가 아니면, 새로운 exception 클래스를 구현하지 말아야한다.
4. ```IllegalArgumentException``` 과 ```IllegalStateException``` 중 무엇을 사용할지 해깔릴때가 있을 수 있는데, 모든 Argument가 비정상이면 ```IllegalStateException``` 을 사용하고, 하나의 argument만 비정상이면 ```IllegalArgumentException``` 을 사용하라.

