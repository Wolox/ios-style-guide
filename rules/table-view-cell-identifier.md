## Table view cell identifiers should be inferred from the class name

### Rationale

TODO

### Example

```swift
protocol IdentifiableCell {
    
    static var cellIdentifier: String { get }
    
}

extension UITableViewCell: IdentifiableCell {
    
    static var cellIdentifier: String {
      return String(self).componentsSeparatedByString(".").first!
    }
    
}

extension UITableView {

  func registerCell(cell: IdentifiableCell) {
    registerNib(UINib(nibName: cell.cellIdentifier, bundle: nil), forCellReuseIdentifier: cell.cellIdentifier)
  }

}
```
