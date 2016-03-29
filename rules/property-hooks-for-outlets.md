## Use property hooks to configure outlets

### Description

Instead of applying style to a `UIView` after the `awakeFromNib()` method, we do it in the `didSet` property of each outlet.

### Rationale

In the traditional way, we would have something like this:

```swift
public final class MyView: UIView {

  @IBOutlet weak var myLabel: UILabel!
  @IBOutlet weak var myButton: UIButton!

  public override func awakeFromNib() {
    super.awakeFromNib()
    setMyLabelStyle()
    setMyButtonStyle()
    ...
  }

}

private extension MyView {

  private func setMyLabelStyle() {
    myLabel.backgroundColor = UIColor.whiteColor()
  }

  private func setMyButtonStyle() {
    myButton.backgroundColor = UIColor.whiteColor()
    myButton.setTitleColor(UIColor.whiteColor(), forState: .Normal)
  }
  ...
}
```

We face some problems with this approach:

**1. We need to override the awakeFromNib() method.**

   This isn't really a problem, but it is usually not necessary.

**2. We need to add a private function for each `IBOutlet`.**

   It often happens that we have to change that outlet, for example `myLabel` to a `UITextField`. Now, we should also change the function `setMyLabelStyle()` to `setMyTextFieldStyle()`.

**3. The code setting the outlet style is visually far from the outlet itself.**

   It forces us to first find which outlet we want to change, then look for the private function and then change that function.

**4. The file gets bigger as we keep adding functions.**

### Example

```swift
public final class MyView: UIView {

  @IBOutlet weak var myLabel: UILabel! {
    didSet {
      myLabel.backgroundColor = UIColor.whiteColor()
    }
  }

  @IBOutlet weak var myButton: UIButton! {
    didSet {
      myButton.backgroundColor = UIColor.whiteColor()
      myButton.setTitleColor(UIColor.whiteColor(), forState: .Normal)
    }
  }
}

```