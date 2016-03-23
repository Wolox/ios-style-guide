## Immutable controller

Controllers should be instantiated programmatically using the initializer that requires a view model. Storyboards should be avoided. The controller's
view model should be a private instance constant.

## Rationale

By avoiding storyboards and [separating view logic from the view controller](./rules/separate-view-logic.md) we have full control on how view controllers are instantiated without loosing the possibility to design our UI using Interface Builder. This allows us to declare a custom initializer making dependencies explicit.

If the view controller is well designed, meaning that it has a single responsibility, all business logic is extracted in services and the presentation logic is extracted in view models. Then the only dependency should be its view model.

By making the controller immutable we avoid having complex logic to keep
the internal state up-to-date. The view controller should only bind the properties of the view model with the view. Changes in the view model should be exposed using [Signal](https://github.com/ReactiveCocoa/ReactiveCocoa/blob/master/ReactiveCocoa/Swift/Signal.swift), [SignalProducer](https://github.com/ReactiveCocoa/ReactiveCocoa/blob/master/ReactiveCocoa/Swift/SignalProducer.swift) or any of the observable [properties](https://github.com/ReactiveCocoa/ReactiveCocoa/blob/master/ReactiveCocoa/Swift/Property.swift) from the [ReactiveCocoa](https://github.com/ReactiveCocoa/ReactiveCocoa) library.

The controller's responsibility gets reduced to coordinate the interaction of the view model with the view and handle events from the Cocoa framework.

## Example

```swift
final class UserProfileController: UIViewController {

  lazy var userProfileView: UserProfileView = UserProfileView.loadFromNib()

  private let _viewModel: UserProfileViewModel

  init(viewModel: UserProfileViewModel) {
    _viewModel = viewModel
    super.init(nibName: nil, bundle: nil)
  }

  required public init?(coder aDecoder: NSCoder) {
    fatalError("init(coder:) has not been implemented")
  }

  override func loadView {
    view = userProfileView
  }

  override func viewDidLoad {
    super.viewDidLoad()
    bindViewModel()
  }

}

private extension UserProfileController {

  var willDealloc: SignalProducer<(), NoError> {
    return rac_willDeallocSignal()
      .toSignalProducer()
      .flatMapError { _ in SignalProducer.empty }
      .map { _ in () }
  }

  func bindViewModel() {
    userProfileView.nameTextField.text = viewModel.name
    userProfileView.emailTextField.text = viewModel.email
    viewModel.fetchAvatar
      .takeUntil(willDealloc)
      .startWithNext { [unowned self] avatar in
        self.userProfileView.avatarView.image = avatar
      }
  }

}
```

Where some of the types that are being used in the `UserProfileController`
could look like:

```swift
struct User {

  let name: String
  let email: String
  let avatarURL: NSURL

}

enum ImageFetcherError {

  case InvalidImageFormat(NSData)
  case FetchError(NSError)

}

typealias ImageFetcher = NSURL -> SignalProducer<UIImage, ImageFetcherError>

final class UserProfileViewModel {

  var name: String { return _user.name }
  var email: String { return _user.email }
  var fetchAvatar: SignalProducer<UIImage, ImageFetcherError> {
    return _fetchImage(_user.avatarURL)
  }

  private let _user: User
  private let _fetchImage: ImageFetcher

  init(user: User, fetchImage: ImageFetcher) {
    _user = user
    _fetchImage = fetchImage
  }

}
```
