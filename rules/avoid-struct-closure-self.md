## Avoid structs that use closure that capture self

### Description

It is not advisable to have value types (`struct`) containing reference types (`class`). Furthermore, `structs` that use a closure that captures `self` must not be used, as it will lead to memory issues.

Quoting Andy Matushack on [classes-vs-structs](https://www.objc.io/issues/16-swift/swift-classes-vs-structs/):

> Value types containing code that executes without being called by its owner are often unpredictable and should generally be avoided. For example: a struct initializer might call dispatch_after to schedule some work. But passing an instance of this struct to a function would duplicate the scheduled effect, inexplicitly, since a copy would be made. Value types should be inert.

> Value types containing references are not necessarily isolated and should generally be avoided: they carry a dependency on all other owners of that referent. These value types are also not readily interchangeable, since that external reference might be connected to the rest of your system in some complex way.

### Rationale

This is because `structs`, as being value types, have value semantics, meaning that they will be copied on assign. Therefore, when we have a `struct` and we try to capture it via `self` (for example for an async operation) we are capturing a copy to the `struct` instead.

Take a look at this:

```swift
public struct Foo {

   private var _number: Int = 0

   public init(signal: Signal<String, NoError>) {
      signal.observeNext {
          print($0)
          self._number = 1
      }
   }
}
```
While this code seems safe, it is creating a new copy of `Foo` with a new value of `_number` whenever the signal emits a new value.

To solve this problem, we should change the `struct` to a `class`.

```swift
public class Foo {

   private var _number: Int = 0

   public init(signal: Signal<String, NoError>) {
      signal.observeNext {
          print($0)
          self._number = 1
      }
   }
}
```

### Example

```swift
public struct Foo {

    private let signal: Signal<Int, NoError>

    public let observer: Observer<Int, NoError>

    public var text: String = ""

    public var disposable: Disposable!

    public init() {
        (signal, observer) = Signal.pipe()
        signal.observeNext {
            self.text = "\($0)"
        }
    }

}

let foo = Foo()

foo.observer.sendNext(4)
print(foo.text) // prints("")
```
It prints "" because `foo.text` equals "", we changed the text of the copy created by `self` on the closure but not the `text` value of `foo`.

####Right way:

```swift
public class Foo {

    private let signal: Signal<Int, NoError>

    public let observer: Observer<Int, NoError>

    public var text: String = ""

    public var disposable: Disposable!

    public init() {
        (signal, observer) = Signal.pipe()
        signal.observeNext { [unowned self] in
            self.text = "\($0)"
        }
    }

}



let foo = Foo()

foo.observer.sendNext(4)
print(foo.text) // prints("4")
```

It now prints 4, because it is a class and we changed the text that `foo` was pointing to. Note that we had to use `[unowned self]` inside the closure because `foo` was already retained by signal (because signal is a property of foo).

### Further information

https://www.objc.io/issues/16-swift/swift-classes-vs-structs/