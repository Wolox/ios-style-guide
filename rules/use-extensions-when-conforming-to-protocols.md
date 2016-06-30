## Use extensions when conforming to protocols

### Description

When a class must conform to a protocol, for example with UI classes' delegates, we declare all the properties and functions needed in a separate extension which implements that protocol.



### Rationale
Even if the class is following all other rules and the Single Responsibility Principle, having all its functions declared inside the class make this confusing and difficult to find. The class can have many private and public functions, but if the class must conform to a protocol, this sure is a particular task. When those protocol conforming functions are extracted into a separate extension, whenever someone wants to review how the class manages that task, the only search needed is looking for the extension that implements the protocol. This prevents the person from having to first, search which are the functions the protocol requires and second, search through all the set of functions the class has for those wanted ones. Moreover, it helps break the class's responsabilities into smaller "tasks", making it easier to follow and understand.

### Example

```swift
public final class SignUpController: UIViewController {

    private lazy var signUpView: SignUpView = .loadFromNib()

    private let _viewModel: SignUpViewModel

    public init(viewModel: SignUpViewModel) {
        _viewModel = viewModel
        super.init(nibName: nil, bundle: nil)
    }

    public override func viewDidLoad() {
        super.viewDidLoad()
        bindViewModel()
    }

    public bindViewModel() {
        signUpView.emailTextField.delegate = self
        _viewModel.email <~ signUpView.emailTextField.rex_textSignal
        signUpView.passwordTextField.delegate = self
        _viewModel.password <~ signUpView.passwordTextField.rex_textSignal
        
        bindButtons()
        ...
    }

    ...

}

extension SignUpController: UITextFieldDelegate {

    public func textFieldShouldReturn(textField: UITextField) -> Bool {
        guard validCredetentials() else {
            textField.resignFirstResponder()
            return true
        }
        if textField == signUpView.emailTextField {
            signUpView.passwordTextField.becomeFirstResponder()
        } else {
            signUpView.emailTextField.becomeFirstResponder()
        }
        return true
    }

}

```
