## Use lazy actions

### Rationale

TODO

### Example

```swift
final class LoginViewModel {

  public let email = MutableProperty("")
  public let password = MutableProperty("")

  private let _credentialsAreValid: AnyProperty<Bool>
  private lazy var logIn: Action<AnyObject, User, SessionServiceError> = {
    return Action(enabledIf: self._credentialsAreValid) { [unowned self] _ in
      if let email = Email(raw: self.email.value) {
        let password = self.password.value
        return self._sessionService.logIn(email,password).observeOn(UIScheduler())
      } else {
        return SignalProducer(error: .InvalidCredentials(.None))
      }
    }
  }()

  init(sessionService: SessionService, credentialsValidator: LoginCredentialsValidator = LoginCredentialsValidator()) {
    let isValidEmail = email.signal.map { validateEmail($0) }
    let isValidPassword = password.signal.map { validatePassword($0) }
    _credentialsAreValid = AnyProperty(
      initialValue: false,
      signal: combineLatest(isValidEmail, isValidPassword).map { $0 && $1 }
    )    
  }

}
```
