## Immutable controller

## Reasoning

TODO

## Example

```swift
struct User {

  let name: String
  let email: String
  let avatarURL: NSURL

}

enum ImageFetcherError {

  case InvalidImageFormat
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
