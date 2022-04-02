# RxSwift
> 비동기적으로 생기는 데이터를 completion과 같은 클로저가 아닌, (Observable이라는 클래스로 감싸서)리턴값으로 전달하기 위한 유틸리티

## Observable 생명주기

1. Create
2. Subscribe
3. onNext (데이터 전달)
4. onCompleted (전달하고 종료) / onError (에러나서 종료)
5. Disposed
- 참고
  - 4부터는 새로 subscribe 하기 전까지는 사용 불가
  - 새로 하는 subscribe와 2의 subscribe는 다르다
  - 4, 5에서 작업 끝나면서 클로저도 종료되고 RC 감소 (순환참조 문제 해결)

## 기본 사용법
* Observable로 비동기 데이터 전달하기
  - create, onNext

* Observable로 오는 데이터 처리하기
  - subscribe

* disposable
  - subscribe의 리턴값
  - 취소할 때 사용하므로 안 쓸 거면 와일드카드 패턴 사용

## sugar API - Operator
- reactivex.io - docs - operators에서 확인 가능
- Observable에서 subscribe로 데이터가 전달되는 도중에 받아서 처리하는 애들을 sugar api 중 operator라고 부름.
- map, filter 등도 오퍼레이터에 포함된다.


* just
  - 하나의 요소만을 방출하는 observable을 생성
  - 여러 개 보내려면 배열로 보내야 함 (한 번에 보내짐)

* from
  - 두 개 이상의 요소를 방출하는 observable을 생성
  - 하나씩 따로 보내짐 

* last
  - observerble이 completed 된 시점에 가장 마지막 요소를 방출

* observeOn
  - observeOn 다음부터의 작업을 수행할 스레드를 지정

  - ex) 메인스레드에서 데이터 처리시키기

        downloadJson(UrlList)
        .subscirbe(onNext: {json in
            DispatchQueue.main.async {
                self.textField.text = json
            }
        })

     위 코드 대신

        downloadJson(UrlList)
        .observeOn(MainScheduler.instance)
        .subscirbe(onNext: {json in
        self.textField.text = json
        })

     이렇게 사용 가능

* subscribeOn
  - (코드 작성 위치와 관계 없이)  시작 스레드를 지정

* buffer
  - 여러 요소를 하나로 묶어서 방출

* scan
  - scan((x, y) => x + y)

* merge
  - 여러 옵저버블의 모든 데이터를 하나로 묶어서 방출 (합집합)
  - 각 옵저버블의 데이터 타입이 같아야함

* zip
  - 두 옵저버블에서 데이터가 하나씩 생성되면 쌍으로 묶어서 방출
  - 쌍이기 때문에 한 옵저버블에서만 데이터가 생셩되면 방출 X

* combineLatest
  - zip과 동일하나, 한 옵저버블에서만 데이터가 생성되면 다른 옵저버블의 가장 최근 데이터와 쌍으로 묶어서 방출한다는 차이점이 있음

## DisposeBag
dispose를 담는 배열이라고 생각하면 됨
- 사용법
       
       var disposeBag = DisposeBag()
    
- 취소하려는 작업이 여러 개일 때 아래 코드로 disposeBag에 dispose를 담을 수 있다.

    
       disposeBag.insert(someDisposable)
    
       // 또는
    
       someDisposable.disposed(by: disposeBag)

- disposeBag은 저장속성(멤버변수)이므로 클래스 인스턴스(ex. VC)가 사라질 때 같이 사라지지만, 순환 참조로 RC가 0이 되지 않을 경우 아래 코드로 초기화 가능함
      
           override func didMove(toParentViewController parent: UIViewController?) {
               super.didMove(toParentViewController: parent)
               if parent == nil {    
                   disposeBag = DisposeBag()
               }
           }
    
## Subject
- 옵저버블은 create 할 때부터 내보낼 데이터가 이미 정해진 형태의 스트림으로, 런타임에 외부 컨트롤에 의해 데이터를 생성시킬 수 없다.
- 때문에 옵저버블 밖에서 데이터를 방출/구독을 모두 할 수 있는 양방향성을 지닌 개념이 필요하다.
- 이게 subject이다.


* PublishSubject
    - subscribe 시점부터 생성되는 데이터를 방출

* BehaviorSubject
    - 기본값을 가지고 시작하는 subject. subscribe하면 값이 생성되지 않아도 기본값이 먼저 방출

* ReplaySubject
    - subscribe하면 그전에 발생한 모든 데이터를 방출하고 이후 데이터가 새로 생성되면 걔네도 방출

* AsyncSubject
    - complete되는 시점에 가장 마지막 데이터를 방출


## RxCocoa
* UI 작업 특징
    1. UI 관련해서 에러가 발생해도 스트림이 끊기면 안 됨. 에러 발생 시 빈 문자열 표시해야함  
        
           .catchErrorJustReturn(“”)
    2. 메인스레드에서 실행돼야 함
    
           .observeOn(MainScheduler.instance)
    - 위 1, 2를 합친 것
    
          .asDriver(onErrorJustReturn: “”)
          .drive(someLabel.rx.text)
* Subject도 스트림이 끊어지지 않는 subject가 필요 => RxRelay
    
      import RxRelay

* BehaviorRelay (BehaviorSubject 대신)
    
      .accept (onNext 대신)
