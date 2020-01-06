---
layout: post
title:  "RxSwift를 활용한 MVVM 구현"
date:   2020-01-06
categories: RxSwift, MVVM
---

RxSwift를 활용한 MVVM 구현은 이미 좋은 예제들이 많이 있습니다. 또 다른 예제를 추가하기 보다는 기존의 예제를 활용하여 설명하는 것으로 대체하고자 합니다. 아래 예제는 [여기](link)를 참고하였습니다.

## 들어가면서
### MVVM vs MVC

MVVM 패턴을 적용하기에 앞서 왜 MVC에서 MVVM으로 페러다임이 이동하고 있는지가 궁금해졌습니다. 무조건 적으로 따르는 것은 제 성격과 맞지 않기에 페러다임의 변화의 이유에 대해서 간단히 살펴보겠습니다.

![MVC vs MVVM](https://www.simform.com/wp-content/uploads/2018/01/MVC-vs-MVP-vs-MVVM-for-ios-development-.png)
*MVVM vs MVC ([출처](https://www.simform.com/mvc-mvp-mvvm-ios-app-development/))*

MVC에서 View와 Model은 긴밀히 연결되어 있습니다. 보기 좋은 구조는 아니지요. View와 Model이 tightly coupled되어 있다는 의미는 view의 갯수에 따라 model의 갯수도 증가되어야 함을 의미합니다. 프로젝트가 커질수록 복잡도는 기하급수적으로 증가하게 됩니다.

MVVM은 view와 model간의 의존성을 완전히 끊어 버렸습니다. View와 Model간의 의존성도 data binding과 command pattern으로 끊을 수 있습니다. View와 bind되어 있는 view model도 최대한 추상화 하여 하나의 view model이 여러 view를 support할 수 있도록 구현해야 합니다.

개인적인 생각으로는 MVVM이 MVC보 낫다 아니다 라고 논쟁할 거리는 아닌 듯 합니다. 앱 사이즈가 작고 구현할 class가 적으면 MVVM구현 보다는 MVC로 구현하는 것이 더 수월할 수 있습니다. 필요에 의해서, 언젠가 MVC가 갖고 있는 단점을 극복하고 싶을 때 MVVM으로 넘어와도 늦지 않다는 생각 입니다.

### RxSwift
MVVM은 ReactiveX (이하 Rx)에 종속된 design pattern이 아닙니다. 따라서 충분히 Rx 없이도 MVVM pattern을 구현할 수 있습니다. ([RxSwift 없이 MVVM pattern 구현하기](https://www.raywenderlich.com/34-design-patterns-by-tutorials-mvvm)) 하지만 data binding에서 편리함을 제공하기에 좋은 기술을 굳이 마다할 이유는 없습니다.

아래는 Rx 쓰지 않았을 때의 data binding과 썼을 때의 data binding의 차이를 보여 줍니다.

우선 Rx를 쓰지 않았을 경우를 살펴 보도록 하겠습니다. ([참고 사이트](https://academy.realm.io/posts/slug-max-alexander-mvvm-rxswift/))

**View Model**
```swift
struct LoginViewModel {

    var username: String = "" {
        didSet {
            evaluateValidity()
        }
    }
    var password: String = "" {
        didSet { evaluateValidity()
        }
    }
    var isValid : Bool = "" {
        didSet {
            isValidCallback?(isValid: isValid)
        }
    }
    var isValidCallback : ((isValid: Bool) -> Void)?

    func attemptToLogin() {
          //truncated for space
    }

    private func evaluateValidity(){
      isValid = username.characters.count > 0
            && password.characters.count > 0
    }
}
```

*didSet* *property observer* 를 통해 username 및 password가 변경될 때 마다 validity를 체크하고(evaluateValidity) 해당 함수를 통해 *isValid*가 set되면 *isValidCallBack* callback block을 호출함으로서 View와 Model간 data binding을 완성시킵니다.

**View**
```
class LoginViewController {

    @IBOutlet var confirmButton: UIButton!
    var loginViewModel = LoginViewModel()

    override func viewDidLoad(){
        super.viewDidLoad()
        loginViewModel.isValidCallback = { [weak self] (isValid) in
            self?.confirmButton.isEnabled = isValid
        }
    }
}
```
View에서는 위와 같이 *isValidCallBack* block이 call되면 *isValid*값에 따라 confirmButton enable / disable 을 set하게 됩니다.

조금 복잡하죠? Rx로 쉽게 해결할 수 있습니다.

**View with Rx**
```
import RxSwift
import RxCocoa

class LoginViewController {

    var usernameTextField: UITextField!
    var passwordTextField: UITextField!
    var confirmButton: UIBUtton!

    var viewModel = LoginViewModel()

    var disposeBag = DisposeBag()

    override func viewDidLoad(){
        super.viewDidLoad()
        usernameTextField.rx.text.bindTo(viewModel.username).addTo(disposeBag)
        passwordTextField.rx.text.bindTo(viewModel.password).addTo(disposeBag)

        //from the viewModel
        viewModel.rx.isValid.map{ $0 }
            .bindTo(confirmButton.rx.isEnabled)
    }
}
```

**View Model with Rx**
```
import RxSwift

struct LoginViewModel {

    var username = Variable<String>("")
    var password = Variable<String>("")

    var isValid : Observable<Bool>{
        return Observable.combineLatest( self.username, self.password)
        { (username, password) in
            return username.characters.count > 0
                        && password.characters.count > 0
        }
    }
}

```
usernameTextField 및 passwordTextField는 Model의 username, password [subject](http://reactivex.io/documentation/subject.html)에 bind해 줍니다. 이렇게 하면 usernameTextField passwordTextField의 값이 변화될 때 마다 변화된 값이 Model의 username, password에 쉽게 말해 observe 가능한 형태로 저장됩니다.

밑에 Model에서 username과 password를 observing함으로서 해당 값의 valid 유무를 observable형태로 return하게 됩니다. (말이 좀 어렵죠? [RxSwift]()에 대한 지식이 좀 필요합니다.)

view는 isValid만 observing하면서 isValid 값에 따라서 enable / disable을 변경하면 되겠네요.

일단 들어가기 전 알아야 할 기본 지식에 대해 다루어봤습니다. 아직 본론으로 들어가지도 않았는데 벌써 지치네요. :)


## Deep dive into the code
