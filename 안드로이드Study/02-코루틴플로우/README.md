# 2주차 Coroutine Flow

[flow-공식문서](https://developer.android.com/kotlin/flow?hl=ko)

### Coroutine의 Flow
- 데이터 스트림 (Data Stream)
- 코루틴 상에서 `리액티브 프로그래밍`을 지원하기 위한 구성요소

- **Reactive Programming**
    - 데이터가 변경될 때 이벤트를 발생시켜서 데이터를 계속해서 전달하도록 하는 프로그래밍 방식
    - ↔ 명령형 프로그래밍과 대응되는 개념
        
        [[RxJava] 반응형 프로그래밍 이해하기](https://kotlinworld.com/125)
        
    - `기존의 명령형 프로그래밍` : 데이터가 필요한 Consumer는 데이터를 요청한 후 받은 결과값을 일회성으로 수신 → 결론적으로 데이터가 필요할 때마다 결과값을 매번 요청해야 한다는 비효율성 존재
    - `리액티브 프로그래밍` : 데이터를 발행하는 Producer가 있고, 해당 Producer는 데이터가 필요한 Consumer가 Producer에게 구독 요청(Subscribe)을 함
    - Producer는 새로운 데이터가 들어오면 Consumer에게 지속적으로 데이터를 발행함
    - 즉, Producer가 Consumer에게 **지속적으로 데이터를 전달**하는데, 이것을 **Data Stream**이라고 함
    
    
    - Flow를 이용한 리액티브 프로그래밍
      > 코루틴에서 데이터 스트림을 구현하기 위해서는 Flow를 사용해야 함
      
   
 ### Data Stream의 구성요소
![image](https://user-images.githubusercontent.com/85485290/185614939-bb3e6d0c-e0e0-4c21-bfc8-bd9e6bc2d504.png)

- `Producer` (생산자, 발행자)
- `Intermediary` (중간 연산자)
- `Consumer` (소비자)

*→ flow의 세가지 핵심 구성요소*

#### Producer

![image](https://user-images.githubusercontent.com/85485290/185615097-e66a0ec2-a6f1-48ee-8dd3-03c800e4dab2.png)



#### Intermediary

![image](https://user-images.githubusercontent.com/85485290/185615138-475e41d7-1df5-44b9-936b-f932b3f1d028.png)



#### Consumer
