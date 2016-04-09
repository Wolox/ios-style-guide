# Wolox iOS style guide

  We follow [GitHub's Swift style guide](https://github.com/github/swift-style-guide) and we enforce it using [SwiftLint](https://github.com/realm/SwiftLint) and [Linterbot](https://github.com/guidomb/linterbot).

 * [Extract view logic from controller](./rules/separate-view-logic.md)
 * [Immutable controllers](./rules/immutable-controller.md)
 * [Use lazy actions](./rules/use-lazy-actions.md)
 * Extract presentation logic into view models
 * Use extensions when conforming to protocols
 * Use private extensions for private methods
 * [Use property hooks to configure outlets](./rules/property-hooks-for-outlets.md)
 * [Avoid structs that use closure that capture self](./rules/avoid-struct-closure-self.md)
 * Dependencies should always be protocols.
 * [Table view cell identifiers should be inferred from the class name.](./rules/table-view-cell-identifier.md)
 * Public properties should appear before private ones.
