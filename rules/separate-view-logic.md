## Extract view logic from controller

### Description

Instead of having all the outlets and styling logic for the view in its View Controller, we extract all this logic to a different ```swift UIView``` subclass. This makes controllers smaller and is a much better way of dividing responsibilites.


### Rationale

A very common problem as applications grow over time is that controllers often end up being quite large and handling a lot of responsibilities. One approach we take at Wolox to avoid this is using the MVVM architecture, thus moving most of the presentation logic out of the View Controller. However, when the view itself is rcomplex, the controller ends up having outlet references to many interface elements and then having to bind them and update them throughout its logic. Therefore, we prefer to have a UIView subclass that has all these references and handles the view logic, together with the styling of the elements. The View Controller still has to pass the elements that it gets from its View Model, but as it only has a reference to the view, it saves it from having to style and manage all these interface elements.

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
