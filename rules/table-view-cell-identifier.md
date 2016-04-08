## Table view cell identifiers should be inferred from the class name

### Rationale

TODO

### Example

```swift
protocol IdentifiableCell {
    
  static var cellIdentifier: String { get }
    
}

extension UITableViewCell: IdentifiableCell {
    
  static var cellIdentifier: String { return String(self) }
    
}

extension UITableView {
    
  func registerCell(cellType: IdentifiableCell.Type) {
    registerNib(UINib(nibName: cellType.cellIdentifier, bundle: nil), forCellReuseIdentifier: cellType.cellIdentifier)
  }
    
}
```
