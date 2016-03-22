## Extract view logic from controller

### Reasoning

TODO

### Example

```swift
final class UserProfileView: UIView {

  @IBOutlet var nameTextField: UITextField!
  @IBOutlet var emailTextField: UITextField!
  @IBOutlet var avatarView: UIImageView!

  static func loadFromNib(bundle: NSBundle = NSBundle(forClass: UserProfileView.self)) -> UserProfileView {
    let nibName = String(self).componentsSeparatedByString(".").first!
    return bundle.loadNibNamed(name, owner: self, options: nil)[0] as! NibLoadableViewType
  }

}

final class UserProfileController: UIViewController {

  lazy var userProfileView: UserProfileView = UserProfileView.loadFromNib()

  override func loadView {
    view = userProfileView
  }

}
```
