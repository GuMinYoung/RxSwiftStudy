# Filtering Operators

### 1. ignoreElements

* .next 이벤트를 무시하고 completed나 .error 같은 정지 이벤트는 허용

      // 1
      let strikes = PublishSubject<String>()
      let disposeBag = DisposeBag()
     
      // 2
      strikes
         .ignoreElements()
         .subscribe({ _ in
             print("You're out!")
         })
         .disposed(by: disposeBag)
     
      // 3
      strikes.onNext("X")        // 무시
      strikes.onNext("X")        // 무시
      strikes.onNext("X")        // 무시
     
      // 4
      strikes.onCompleted()      // 방출
     
* 출력 결과


  `You're out!`
      
 

### 2. elementAt

* n번째 요소만 방출

      // 1
      let strikes = PublishSubject<String>()
      let disposeBag = DisposeBag()

      // 2
      strikes
          .elementAt(2)
          .subscribe(onNext: { event in
              print(event)
              print("You're out!")
          })
          .disposed(by: disposeBag)

      // 3
      strikes.onNext("X1")    // 0번째
      strikes.onNext("X2")    // 1번째
      strikes.onNext("X3")    // 2번째 - 이거만 출력

* 출력 결과


    `X3`
 
 
    `You're out!`



### 3. filter

* 조건에 부합하는 요소만 방출


      let disposeBag = DisposeBag()

      // 1
      Observable.of(1,2,3,4,5,6)
      // 2
      .filter({ (int) -> Bool in
          int % 2 == 0
      })
      // 3
      .subscribe(onNext: {
          print($0)
      })
      .disposed(by: disposeBag)


* 출력 결과


   `2 4 6`
   
   
### 4. skip

* n번째 요소까지 스킵

      let disposeBag = DisposeBag()
     
      // 1
      Observable.of("A", "B", "C", "D", "E", "F")
      // 2
      .skip(3)
      .subscribe(onNext: {
          print($0)
      })
      .disposed(by: disposeBag)


### 5. skip(while:)

* 특정 조건에 부합하는 요소는 스킵하되, false인 요소가 나오면 그 때 부터 모든 값을 취함

 
      let disposeBag = DisposeBag()
     
       // 1
       Observable.of(2, 2, 3, 4, 4, 2)
       //2
       .skip(while: { (int) -> Bool in
             int % 2 == 0
             // 짝수이면 방출하다가 홀수가 나올 때부터 모든 값을 방출
       })
       .subscribe(onNext: {
           print($0)
       })
       .disposed(by: disposeBag)


* 출력 결과

   `3 4 4 2`

### 6. skip(until:)

* 특정 Observable의 onNext가 발생할 때 까지 스킵. onError, onCompleted는 해당 X!

      let disposeBag = DisposeBag()

      // 1
      let subject = PublishSubject<String>()
      let trigger = PublishSubject<String>()
      
      // 2
      subject
          .skip(until: trigger)
          .subscribe(onNext: {
              print($0)
          })
          .disposed(by: disposeBag)
      
      trigger
          .subscribe(onNext: {
          print($0)
      })
      
      // 3
      subject.onNext("A")
      subject.onNext("B")
      
      // 4
      trigger.onNext("X")
      
      // 5
      subject.onNext("C")


* 출력 결과

   `X C`



### 7. take

* n개의 요소를 취함. (<-> skip)

      let disposeBag = DisposeBag()
      
      // 1
      Observable.of(1,2,3,4,5,6)
      // 2
          .take(3)
          .subscribe(onNext: {
              print($0)
          })
          .disposed(by: disposeBag)
      
* 출력 결과

  `1 2 3`




### 8. take(while:)
* 특정 조건에 부합하는 요소는 취하되, false인 요소가 나오면 그 때 부터 모든 값을 스킵

      let disposeBag = DisposeBag()
      
      // 1
      Observable.of(2,2,4,4,6,6)
      // 2
          .enumerated()
      // 3
          .takeWhile({ index, value in
              // 4
              value % 2 == 0 && index < 3
          })
      // 5
          .map { $0.element }
      // 6
          .subscribe(onNext: {
              print($0)
          })
          .disposed(by: disposeBag)

* 출력 결과

  `2 2 4`




### 9. take(until:)
* trigger가 되는 Observable의 onNext(onError, onCompleted는 해당 X)가 발생하기 전까지의 이벤트 값만 받음.
* trigger Observable의 onNext가 발생하면 기존 Observable은 completed 된다.


      let disposeBag = DisposeBag()
      
      // 1
      let subject = PublishSubject<String>()
      let trigger = PublishSubject<String>()
      
      // 2
      subject
      .take(until: trigger)
      .subscribe{
            print($0)
      }.disposed(by: disposeBag)
      
      // 3
      subject.onNext("1")
      
      // 4
      trigger.onNext("X")    // subject가 completed 되는 시점
      
      // 5
      subject.onNext("2")

* 출력 결과
 
  `next(1)`
 
  `completed`



### 10. enumerated

* 방출된 요소의 index를 알고 싶을 때 사용

      let disposeBag = DisposeBag()
      
      // 1
      Observable.of(2,2,4,4,6,6)
      // 2
          .enumerated()
      // 3
          .take(while: { index, value in
              // 4
              value % 2 == 0 && index < 3
          })
      // 5
          .map { $0.element }
      // 6
          .subscribe(onNext: {
              print($0)
          })
          .disposed(by: disposeBag)

* 출력 결과
  
  `2 2 4`

