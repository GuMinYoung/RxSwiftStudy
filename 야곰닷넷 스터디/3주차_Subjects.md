# Subject
> Subject는 옵저버+옵저버블이다.   
      - 옵저버 - 관찰자   
      - 옵저버블 - 방출

### behaviorSubject
* 초기값 존재
* 초기값을 주면서 `value:` 에서 타입 추론이 가능하므로 publishSubject 만들 때 썼던 <> 생략 가능
* 마지막 `.next` 및 종료 이벤트 `.completed` 또는 `.error` 를 새로운 구독자에게 반복한다는 점만 빼면 PublishSubject와 유사하다.
