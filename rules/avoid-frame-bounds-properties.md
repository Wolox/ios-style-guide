## Avoid using frame and bounds properties to add child views

When adding child view controller's views to the view hierarchy programmatically avoid using the container's bound or frame properties to
configure the contained view's frame.

## Rationale

When we define complex views, we generally use Interface Builder to layout all the view components and some times, for reusability or to separate responsibilities, we extract some sections of the view into their own view controllers. The proper way to this is to define a container view, where the child view controller's view will be added, and use interface builder to set its layout constrains. Then once our controller view has been loaded we can instantiate all the child view controllers and add their view to the view hierarchy by adding them as a sub view of their corresponding container view.

The idea is to leverage the power of auto-layouts to calculate the container view's size and position and then make the contained view fill the container view. A naive approach to implement this would be to use the container view's bounds to configure the contained view's frame. The problem with this approach is that, most probably, you will be adding the child component in the `loadView` or `viewDidLoad` method and at those moments the layout engine hasn't been run yet. Making the bounds property report wrong values.

A common trick to overcome this problem is to setup the child view in the `viewDidLayoutSubviews` method and use a flag to mark if the child view has been configured. Even though this works it forces to maintain more state in the view controller and makes the layout engine do another pass after the child view has added.

The proper way to do this would be use auto layout constrains to make the child view fill the content of the container view by pinning their edges.

## Example

The following example shows how to add a child view controller's view in the view hierarchy. In this case we have a parent controller (for example the one that is presented as the window's root view controller ). This controller could be as complex as you can imagine and it could have several UI components. For the sake of this example all those components have been omitted.

The important thing is that there is a `mapContainerView` that will be used (as its name suggests) to contain a map view from a child view controller. This container view's layout has been configured using auto-layouts with Interface Builder. The idea is that the map view fills the container view. The child view controller is added programmatically.

### Wrong way

The wrong way of doing this would be to use the `mapContainerView`'s bound as
the map view's frame.

```swift
public final class MyViewController: UIViewController {

  @IBOutlet weak var mapContainerView: UIView!

  private lazy var mapViewController: MapViewController = MapViewController()

  override public func viewDidLoad(animated: Bool) {
    super.viewDidLoad(animated)

    mapViewController.willMoveToParentViewController(self)
    addChildViewController(mapViewController)
    mapViewController.didMoveToParentViewController(self)

    mapViewController.view.frame = mapContainerView.bounds
    mapContainerView.addSubview(mapViewController.view)
  }

}
```

The example above will result in the map view configured with the wrong size. The next example works but requires to keep state in the view controller and forces another pass of the layout engine.

```swift
public final class MyViewController: UIViewController {

  @IBOutlet weak var mapContainerView: UIView!

  private lazy var mapViewController: MapViewController = MapViewController()
  private var mapViewConfigured = false

  override public func viewDidLayoutSubviews() {
    if (!mapViewConfigured) {
      mapViewController.willMoveToParentViewController(self)
      addChildViewController(mapViewController)
      mapViewController.didMoveToParentViewController(self)

      mapViewController.view.frame = mapContainerView.bounds
      mapContainerView.addSubview(mapViewController.view)

      mapViewConfigured = true
    }
  }

}
```

### Right way

The proper solution would be to use layout constrains to pin the map's view edges to the container's. A couple more of code lines but the map view will have the correct dimensions and its content will be able to adjust properly.

If you find yourself repeating this code you can extract the logic of pinning a view into a container view in an extension on `UIView`.

```swift
public final class MyViewController: UIViewController {

  @IBOutlet weak var mapContainerView: UIView!

  private lazy var mapViewController: MapViewController = MapViewController()

  override public func viewDidLoad(animated: Bool) {
    super.viewDidLoad(animated)

    mapViewController.willMoveToParentViewController(self)
    addChildViewController(mapViewController)
    mapViewController.didMoveToParentViewController(self)

    mapContainerView.addSubview(mapViewController.view)
    mapContainerView.translatesAutoresizingMaskIntoConstraints = false
    mapViewController.view.translatesAutoresizingMaskIntoConstraints = false

    // Pins map view into the container view on all edges
    mapContainerView.topAnchor.constraintEqualToAnchor(mapViewController.view.topAnchor).active = true
    mapContainerView.bottomAnchor.constraintEqualToAnchor(mapViewController.view.bottomAnchor).active = true
    mapContainerView.leadingAnchor.constraintEqualToAnchor(mapViewController.view.leadingAnchor).active = true
    mapContainerView.trailingAnchor.constraintEqualToAnchor(mapViewController.view.trailingAnchor).active = true
  }

}
```
