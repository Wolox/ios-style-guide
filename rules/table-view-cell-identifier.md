## Table view cell identifiers should be inferred from the class name

### Rationale

TODO

### Example

```swift
protocol IdentifiableCell {

  var cellIdentifier: String { get }

}

extension IdentifiableCell where Self: UITableViewCell {

  var cellIdentifier: String {
    return String(self.dynamicType)
  }

}

final class MyCustomCell: UITableViewCell, IdentifiableCell {

}

extension UITableView {

  func registerCell(cell: IdentifiableCell) {
    registerNib(UINib(nibName: cell.cellIdentifier, bundle: nil), forCellReuseIdentifier: cell.cellIdentifier)
  }

}
```
