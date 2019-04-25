---
title: Swift Style Guide
tags: engineering
---

This document captures the style guidelines for Swift code at Omni. New Swift code should be written according to the current guidelines. As existing code is edited, it should generally be migrated toward the current guidelines within reason and professional judgment. 

This is intended to be a living document. As the language and our experience with it evolve, we're likely to conclude that some decisions recorded here need to be refined or changed. The Engineers chat room is a good place to start discussions of proposed changes, and the weekly Engineering meeting is the likely place to seek consensus before updating the document.

Much of this document was guided by and derived from *Swift Style: An Opinionated Guide to an Opinionated Language* by Erica Sadun, and many of the code examples are hers. We thank Erica for her contributions to the Swift community.

## Whitespace and Layout

### Indentation

For consistency with our existing code base and portability between editors, indent using spaces with four spaces per indent level. This is *not* the Xcode default but can be configured in the *Text Editing* pane of Xcode's preferences.

> Prefer indent using: Spaces
>
> Indent width: 4 spaces

### Braces

Use One True Brace Style (1TBS) with few exceptions, noted below. In 1TBS opening braces appear at the end of any statement that establishes a clause. The `else` keyword in an `if` statement appears between a closing brace and the next open brace. For example:

```swift
func setGlowing(_ isGlowing: Bool, animated: Bool) {
    if animated {
        UIView.animate(withDuration: 0.1, animations: { 
            self.isGlowing = isGlowing
        })
    } else {
        self.isGlowing = isGlowing
    }
}
```

There are two exceptions to the 1TBS guideline: hard-wrapped headers and co-linear braces.

#### Hard-Wrapped Headers

It is OK to not use 1TBS is in function declarations that have hard wrapping in their headers. In this case, move the opening brace of the block to the following line ("Allman style"):

```swift
public init<C : BidirectionalCollection>(_ base: C)
    where C.Iterator.Element == Element,
    C.SubSequence : BidirectionalCollection,
    C.SubSequence.Iterator.Element == Element,
    C.SubSequence.Index == C.Index,
    C.SubSequence.Indices : BidirectionalCollection,
    C.SubSequence.Indices.Iterator.Element == C.Index,
    C.SubSequence.Indices.Index == C.Index,
    C.SubSequence.Indices.SubSequence == C.SubSequence.Indices,
    C.SubSequence.SubSequence == C.SubSequence,
    C.Indices : BidirectionalCollection,
    C.Indices.Iterator.Element == C.Index,
    C.Indices.Index == C.Index,
    C.Indices.SubSequence == C.Indices 
{
    self.base = base
}
```

Note, however, that we avoid hard wrapping except when absolutely necessary for readability or to document `where` clauses or default arguments. Techniques for reducing the need for hard wrapping include configuration objects and the Builder pattern to shorten parameter lists. Type aliases can sometimes shorten `where` clauses.

#### Co-linear Braces

There are a few circumstances where it makes sense to put an entire block, including the opening and closing braces, on a single line. The term of art for this is *co-linear braces*, like so:

```swift
guard let context = editingContext else { return }
```

When using co-linear braces, follow the Rule of Joshua (Weinberg): *braces are stand-offish*. That is, do this:

```swift
let result = threes.map { "#\($0)" }
```

not this:

```swift
let result = threes.map {"#\($0)"} // Nope. I'm squishing your code.
```

Favor co-linear braces for:

- guard statements where the body is `return`, `return nil`, `return true`, `return []`, `break`, or `continue`; and
- very simple blocks where it's reasonable to not name the parameter, e.g.,

```swift
    let threes = (1 ... 15).filter { $0 % 3 == 0 }
```

### Parentheses

Don't be afraid to add parentheses to expressions to improve clarity. We don't assume that everyone has memorized the precedence rules. However, if the number of parentheses start to obscure the meaning, break expressions into named subexpressions to improve clarity.

Parentheses are friendly sorts. They should hug the code near them:

```swift
let paddedCount = String(count).pad(to: 8) // yes
let paddedCount = String( count ).pad( to: 8 ) // no
```

Like declarations, we try to avoid hard wrapping in method and function calls. If you decide to hard wrap a call for clarity, the trailing parenthesis should hug the last argument and each argument should be on its own line. 

One place where we favor wrapping at call sites is when calls take closures as arguments. Use trailing closure syntax except when the expression includes multiple closures. For example, for a simple animation you should use:

```swift
UIView.animate(withDuration: 0.2) {
	self.view.alpha = 0
}
```

But if you're using the form with a second block for completion, write:

```swift
UIView.animate(withDuration: 0.2, animations: {
	self.view.alpha = 0
}, completion: { _ in
	self.view.removeFromSuperview()
})
```

### Aligning with Whitespace

Except in very narrow cases, don't use whitespace to try to align text. This looks nice, but is a maintenance headache and doesn't improve scan-ability:

```swift
// Don't do this:
let userKey     = "User Name"
let passwordKey = "User Password"
```

An exception to this rule is when scanning down multiple lines is helpful in identifying skips in a sequence or other errors of omission. OptionSet declarations are a common example of this:

```swift
// OK:
struct Conditions: OptionSet {
	let rawValue: Int
	static let isEditing                 = Conditions(rawValue: 1 << 0)
	static let isInMultipleSelectionMode = Conditions(rawValue: 1 << 1)
}
```

### Wrapping Collections

Although we generally recommend against hard wrapping, there are some collection literals that call for wrapping. Wrap a collection when it's:

- likely to grow, 
- likely to be reordered,
- requires in-line documentation, and/or 
- begs comparison of the elements.

When wrapping, put a trailing comma after the last item and put the closing bracket on the following line. This makes it easier to add or comment out items and simplifies diffs.

```swift
let items = ["cat", "banana", "pony"]
var primes: Set<Int> = [1, 2, 3, 5, 7, 11]
var counts = ["red": 3, "blue": 6, "green": 1]
let optionalAddOperations: [BarButtonController.Button?] = [
        includeNewAction ? .newAction(enabled: true) : nil,
        includeNewProject ? .newProject(enabled: true) : nil,
        includeNewTag ? .newTag(enabled: true) : nil,
        includeNewFolder ? .newFolder(enabled: true) : nil,
        ]
```

### Spacing and Punctuation

Regarding spaces around punctuation, our general guidelines are to follow the rules of written English. That is, no space before a punctuation symbol and a single space after. This is sometimes called *left-magnetic* spacing. Some specifics:

- Commas: left-magnetic, including in generic parameters

```swift
func blarg<Left, Right>() -> (Left, Right) {
	…
	return (left, right)
}
…
(x, y) = blarg()
```

- Colons: left-magnetic, except for ternaries and operator declaration, which should have a space on either side. This is sometimes called the *Mike Ash Rule*. Sometimes Xcode will insert extra spaces. Don't fight Xcode when it does.

```swift
let dict: [String: Int]
dict = ["x": 13, "y": 12]
let z = dict["x"] > 10 ? "Foo" : "Bar"
infix operator +- : AdditionPrecedence
func +-(lhs: Double, rhs: Double) -> Range<Double> { … }
```

- Semicolons: don't use 'em

```swift
// Nope:
x = 12; y = 13

// Yep:
x = 12
y = 13
```

- Angle-brackets: hug

```swift
func blarg<Left, Right>() -> (Left, Right) { … }
func flarg<Value>() -> Value { … }
```

- Binary operators: do what seems most readable in the circumstance
  
  For example `a + b^c` is probably better than `a + b ^ c`, but the choice between `a + (b * c)`, `a + b * c`, and `a + b*c` depends on the actual expressions plugged into `a`, `b`, and `c`. Don't be afraid to introduce local variables to simplify expressions.

- Return arrow tokens: put a space on either side

```swift
// Yep
func f() -> Int

// Nope
func f()-> Int

// What kind of monster are you?
func f()->Int
```

- Ranges: closed ranges are clingy and half-open ranges are standoffish

```swift
for i in 1...3 { … }
for j in 1 ..< 3 { … }
```

- Empty constructs: no internal spaces

```swift
[], [:], {}, f()
```

- Comments: add single-space padding between comment delimiters and text

```swift
// blah
/// blah
//: blah
/* blah */
/** blah */
/*: blah */
```

### Maximum Line width

We do not set a maximum line width limit. Xcode is happy to wrap lines.

### Attributes

Attributes on declarations should each go on their own line above the declaration except for shorter attributes (like `@objc` and `@IBOutlet`), which are left to your discretion.

These are all OK:

```swift
@objc internal var count: Int

@objc
internal var count: Int

@objc(objectAtIndex:) 
internal func objectAt(_ index: Int) -> AnyObject { … }

@available(*, unavailable, renamed: "Iterator")
public typealias Generator = Iterator

@discardableResult
func addTask(named name: String, at index: Int) -> OFMTask
```

When a type is be bridged using an `@objc` attribute, make sure its bridged name
is appropriate for our [Objective-C style](/coding-style-guidelines). (This
includes adding prefixes for bridged types in frameworks.)

### Number literals

Number literals in Swift can have intermediate underscores to enhance readability:

```swift
let largeInteger = 1_732_500 // decimal
let largeInteger = 0xff_ff_ff // hex
let b2 = 0b0111_0001_1001_1111 // binary
let o2 = 0o71_24_32_26 // octal
```

This is kind of cool, but we don't take a position on whether this is proper style. In general we shouldn't have many literals in our code.

Swift allows zero padding, both leading and trailing. The former is technically allowed because, unlike C, an extra leading 0 doesn't signify an octal literal.

```swift
// Nope
let value1 = 005.0300
let value2 = 123.0000
let value3 = 000.0520
let value4 = 010.1621
```

Though this is allowed by Swift, don't do this. The extra zeros interfere with readability. Note, however, for code that depends on differences between the numbers, it's OK to align the decimal points if that makes the code easier to scan:

```swift
// OK to decimal-align if scanning is necessary
let value1 =   5.03
let value2 = 123.0
let value3 =   0.052
let value4 =  10.1621
```

## Language Feature Use

### Closures

Swift encourages the use of closures. A full-fledged closure has a very busy declaration:

```swift
let closure = { [weak helper, unowned self] (x: Int, y: Int) -> Vector in
	…
}
```

All of these declaration elements are sometimes optional, depending on whether the argument types and return type can be inferred among other things. We follow Erica Sadun's guidance: "Add declaration elements reluctantly, preferring to include just those items required to make your closure compile and function properly."

#### Shorthand $0, …

When the types can be inferred, Swift lets us omit parameter names and use shorthand `$0`, `$1`, … placeholders in the body of the closure. Reserve closure shorthand for short and simple statements. When your closure extends beyond a line or two, establish argument names. Here “short and simple” means a single expression with just a single use of any shorthand argument, and short enough to be written with co-linear braces.

#### Placement of `in`

When a closure declaration requires the `in` keyword, `in` goes with the rest of the declaration. Declarations go with the opening `{`:

```swift
// Nope:
let _ = array.withUnsafeMutableBufferPointer({
        (arrayPtr: inout UnsafeMutableBufferPointer<Int16>)
        in
        source.copyBytes(to: arrayPtr)
})

// Yep:
let _ = array.withUnsafeMutableBufferPointer { (arrayPtr: inout UnsafeMutableBufferPointer<Int16>) in
        source.copyBytes(to: arrayPtr)
})
```

#### Chaining Closures

It's sometimes OK to chain multiple blocks when they're very simple. When doing so, put each chained function on a separate line and use parentheses. The parentheses are required for all but the last function, so using them on all functions lends consistency:

```swift
let activeProjects = objects
    .flatMap({ $0 as? OFMProjectInfo })
    .filter({ $0.status == .active })
```

Note, however, that it's generally better for readability, maintenance, and debugging to name intermediate results. So the above might be better written:

```swift
let projects = objects.flatMap { $0 as? OFMProjectInfo }
let activeProjects = projects.filter { $0.status == .active }
```

This example also lets us eliminate the parentheses.

#### Implicit `return`

The preceding example uses implicit `return` statements. That is, it could equivalently be written:

```swift
// not as nice
let projects = objects.flatMap { 
        return $0 as? OFMProjectInfo
    }
let activeProjects = projects.filter { 
        return $0.status == .active
    }
```

Use implicit `return` only for co-linear closures and explicit return otherwise.

#### Autoclosure

Don’t use `@autoclosure` to optimize out closure braces in the calling syntax. Restrict autoclosure for things like logging and assertions where short-circuiting is needed.

#### Trailing Closures

As mentioned above, favor trailing closure syntax with two exceptions.

First, wrap closure arguments in parentheses when chaining blocks.

Second, when an expression contains multiple blocks prefer to use labels for all of them. For example:

```swift
// Yes
UIView.animate(withDuration: 0.2, animations: {
    self.view.alpha = 0
}, completion: { _ in
    self.view.removeFromSuperview()
})

// No
UIView.animate(withDuration: 0.2, animations: {
    self.view.alpha = 0
}) { _ in
    self.view.removeFromSuperview()
}
```

This avoids privileging the final block over the others and avoids the run-on punctuation in the middle: `}) { _`.

This isn't an ironclad rule. Trailing closure syntax may make sense if the last block is clearly primary. For example, this might make sense:

```swift
database.fetch(matching: { $0.status == .active }) { activeItems in 
    … do some things with the matched items …
}
```

We explicitly reject the [Rule of Kevin](http://ericasadun.com/2015/11/17/a-handful-of-swift-style-rules-swiftlang/). Whether or not to use trailing closure syntax is a presentation decision, not one based on the return type of the block. Changing a return type shouldn't require re-punctuating your code.

#### Escaping Closures

Prefer nonescaping closures (which are the default anyways).

### Guard Statements

We prefer to check for exceptional or trivially handled conditions and early-out from methods, functions, and other scopes rather than deeply nesting blocks. Swift `guard` statements are the preferred mechanism for doing early-outs. They set two important expectations for the reader: 

- the body of the guard handles exceptional or trivially handled conditions, and thus can be skimmed when one is looking for the main behavior of the enclosing block
- unlike an `if` statement, a guard is guaranteed to exit the enclosing block

To keep the conditions in `guard`s easily scannable, we prefer to break the conditions into separate `guard` statements when that's reasonable. 

Seek to simplify guard conditions. For example, introducing local variables can produce a more readable `guard` than using a single, more complex expression.

```swift
// not great
guard !(arguments.last?.isEmpty ?? true) else { return }

// better
guard let path = arguments.last, !path.isEmpty else { return }
```

As with declaration headers, we *avoid hard wrapping* the conditions in a `guard` statement except when necessary for readability or to document the individual conditions. *If* the conditions are hard wrapped:

- the first condition goes on the same line as the `guard` keyword,
- let Xcode handle the indentation,
- the `else {` goes on the same line as the last condition, and
- a co-linear block is right out.

```swift
// ok, particularly if chunky and monkey are only used to calculate banana
guard let chunky = …,
    let monkey = …,
    let banana = … else {
        return
}

// preferred
guard let chunky = … else { return }
guard let monkey = … else { return }
guard let banana = … else { return }
```

`guard` statements are not a substitute for `if` statements. Guards establish the conditions necessary for successful execution of subsequent code. They communicate an exceptional condition.

Because of their role as handlers of exceptional conditions and early exits, a `guard` statement with a large `else` block may be a code smell, especially if the code following the guard is substantially smaller than the `guard` itself. 

For example, here the `guard` does all the work and the subsequent code is trivial.

```swift
// Problematic use of guard
func truncateIfNecessary(_ string: String) -> String {
    guard string.count < maxChars else {
        let reversedPrefix = string.prefix(maxChars).unicodeScalars.reversed()
        let truncatedReversedPrefix = reversedPrefix.drop(while: { !CharacterSet.whitespaces.contains($0) }).dropFirst()
        let interestingPrefix = truncatedReversedPrefix.reversed()
        var result: String
        if interestingPrefix.isEmpty {
            result = String(string.prefix(maxChars))
        } else {
            result = String(String.UnicodeScalarView(interestingPrefix))
        }
        result.append("…")
        return result
    }
    
    return string
}

// Better: guard is concise
func truncateIfNecessary(_ string: String) -> String {
    guard string.count >= maxChars else { return string }
    
    let reversedPrefix = string.prefix(maxChars).unicodeScalars.reversed()
    let truncatedReversedPrefix = reversedPrefix.drop(while: { !CharacterSet.whitespaces.contains($0) }).dropFirst()
    let interestingPrefix = truncatedReversedPrefix.reversed()
    var result: String
    if interestingPrefix.isEmpty {
        result = String(string.prefix(maxChars))
    } else {
        result = String(String.UnicodeScalarView(interestingPrefix))
    }
    result.append("…")
    return result
}

```

Generally if the block of a `guard` becomes long consider alternative approaches, like:

- using a `switch` or `if-else` to perform case analysis
- reversing the sense of the method so that the exceptional case is the simpler one
- throwing from the `guard` or otherwise consolidating the exceptional case, e.g., by calling a helper method

Keep in mind that given the expectations a `guard` statement sets for the reader, it may still be the appropriate choice.

### Ternaries

Ternaries are OK, but keep them simple enough that they don't need to wrap. 

```swift
// Nope    
return style == .left
    ? padString + self
	: self + padString

// Yep
let prefix = (style == .left ? "" : padString)
```

When possible, prefer nil-coalescing to ternary or cascaded `if`s

```swift
return optValue == nil ? fallback : optValue! // no
return optValue ?? fallback // yes
```

In limited cases, cascaded nil-coalescing is OK, but exercise restraint.

```swift
// OK
return specificValue() ?? fallbackValue() ?? finalDefault
```

### Type Inference and Declarations

Swift will automatically interpret integer literals as floating point in a floating point context, like `let x: CGFloat = 1`. We take advantage of this. However, when a method call has multiple arguments of the same type, don't mix literal representations:

```swift
// Nope
let frame = CGRect(x: 0, y: 0, width: 400.0, height: 300.0)
let red = UIColor(red: 1, green: 0.231, blue: 0.65, alpha: 1)

// Yep
let frame = CGRect(x: 0, y: 0, width: 400, height: 300)
let color = Color(red: 0.0, green: 0.5, blue: 0.5, alpha: 1.0)
```

In general, prefer to omit redundant type information. Inference of literal types is type information. For example:

```swift
public struct MyStruct2 {
    public var count = 32 // yes, inferred to be Int, so no type decl is required
    
    public let name: String // yes, type declaration needed since there is no initializer expression
    
    let myPoint1: CGPoint = CGPoint(x: 10.0, y: 30.0) // no, redundant `CGPoint`
    let myPoint2 = CGPoint(x: 10.0, y: 30.0) // yes
	
    var myShape: Shape = Square() // yes, not redundant, initializer is subtype of property's type
    
    public var countAsDouble: Double { // yes, required by the compiler
        get { 
        	return Double(count)
        }
        set { 
        	count = lrint(newValue)
        }
    }
}
```

Keep type names (when needed) close to variable names:

```swift
let freezingPoint: CGFloat = 32.0  // yes
let freezingPoint = 32.0 as CGFloat // no
let freezingPoint = CGFloat(32.0) // no
```

Use the same syntax when referencing values that are enum cases or type
properties:

```swift
let direction: Direction = .west // yes
let highlightColor: NSColor = .red // yes
let mainBundle: NSBundle = .main // yes

let text = NSLocalizedString("foo", table: "Localizable", bundle: .main, comment: "foo") // yes
```

### Extended Literals

Swift and Xcode collaborate to compile and render special literals for colors, images, and files. In raw source code, these look like:

```swift
let color = #colorLiteral(red: 0.9994240403, green: 0.9855536819, blue: 0, alpha: 1)
let image = #imageLiteral(resourceName: "flourish.png")
let file = #fileLiteral(resourceName: "README.txt")
```

Swift infers the type of these expressions and even handles platform differences. For example `#colorLiteral`s are inferred to be of type NSColor or UIColor as appropriate. However, given that there's no way to turn off the feature where Xcode renders these as glyphs instead of text, *don't use extended literals in production code*.

### Collection Literals

Thanks to `ExpressibleByArrayLiteral` and `ExpressibleByDictionaryLiteral`, many Swift collections can be created using literal expressions. Prefer to use this mechanism. For example:

```swift
let array: [String] = [] // yes
let dictionary: [AnyHashable: Any] = [:] // yes
let aSet: Set<String> = [] // yes

let array = [String]() // no
let array = Array<String>() // no
let dictionary = [AnyHashable: Any]() // no
let dictionary = Dictionary<AnyHashable: Any>() // no
let aSet = Set<String>() // no
```

As seen above, Swift provides a shorthand for expressing the types of arrays and dictionaries. It can be tempting to ignore the shorthand when commingling these types with less elegant collections, like so:

```swift
var arrayOfStrings: Array<String> = ["a", "b", "c"] // Nope
var setOfStrings:   Set<String>   = ["a", "b", "c"]
```

This wastes time and causes the reader to ponder why the standard `[String]` shorthand or inference wasn't used. Instead, write this example according to guidelines above:

```swift
var arrayOfStrings = ["a", "b", "c"] // inferred as [String]
var setOfStrings: Set<String> = ["a", "b", "c"] // a set, thanks to ExpressibleByArrayLiteral
```

It's possible to provoke Swift into inferring the type of a collection by explicitly typing the first item in a collection literal. Avoid that. Instead explicitly declare the type of the entire collection when explicit typing is required. Introduce a local variable if necessary.

```swift
let array: [Character] = ["a", "b", "c"] // yes
let array: [NSColor] = [.blue, .red, .green] // yes

let array = ["a" as Character, "b", "c"] // no
let array = ["a", "b", "c"] as [Character] // no
let array = [NSColor.blue, .red, .green] // no
let array = [Character("a"), "b", "c"] // no

for color in [NSColor.blue, .red, .green] { … } // nope
for color in ([.blue, .red, .green] as [NSColor]) { … } // no
for char in ["a" as Character, "b", "c"] { … } // nope
for char in (["a", "b", "c"] as [Character]) { … } // no

// yes:
let primaries: [NSColor] = [.blue, .red, .green]
for color in primaries  { … }
```

### Initializing `OptionSet`s

Use array literal shorthand for expressions that yield `OptionSet`s, including single and empty options. This makes it easier for the reader to recognize the option set and to add options. Don't use the `rawValue` initializer directly. That's both too clever and fragile to API changes.

```swift
// yes
if let json = try JSONSerialization.jsonObject(with: data, options: [.mutableContainers, .mutableLeaves]) …
// no
if let json = try JSONSerialization.jsonObject(with: data, options: JSONSerialization.ReadingOptions(rawValue: 3)) …
		
// yes
if let json = try JSONSerialization.jsonObject(with: data, options: []) …
// no
if let json = try JSONSerialization.jsonObject(with: data, options: JSONSerialization.ReadingOptions(rawValue: 0)) …
		
accessQueue.sync(flags: [.barrier]) // yes
accessQueue.sync(flags: .barrier) // no
```

When options commonly occur together, it's helpful to declare additional static presets:

```swift
struct LaundryOptions: OptionSet {
    var rawValue: Int
    
    init(rawValue: Int) {
        self.rawValue = rawValue
    }
    
    static let coldWater = LaundryOptions(rawValue: 1 << 0)
    static let hotWater  = LaundryOptions(rawValue: 1 << 1)
    static let lowHeat   = LaundryOptions(rawValue: 1 << 2)
    
    // yes, common combination
    static let energyStar: LaundryOptions = [.coldWater, .lowHeat]
}

// handy:
runningShirt.launder(with: [.energyStar])
```

### Optional Sugar

Because optionals in swift are enums with associated values under the covers, it's possible to use pattern matching against them. Don't. Instead prefer the binding `?` shorthand:

```swift
switch (firstOptional , secondOptional) {
case let (.some(first), .some(second)): ... // nope
case (.none, .none): ... // nope
...
default: break
}

switch (firstOptional , secondOptional) {
case (.some(let first), .some(let second)): ... // again, no
...
default: break
}

switch (firstOptional , secondOptional) {
case let (first?, second?): ...  // yes? yes!
case (nil, nil): ... // yeah

...
default: break
}
```

### Tuples

Swift's tuples can provide a very quick way of associating values, particularly when elements of the tuple are named.

```swift
// OK
func fetchResult(from query: Query) -> (result: Result, continuation: () -> Void) { 
    …
    return (result: result, continuation: { self.reset() })
}
```

While this is a great way to begin developing an abstration, in most cases, we'll tend to replace tuples used in this way with lightweight `struct`s. Lightweight structs concentrate the syntactic noise at the declaration site and simplify the other references. Keep in mind that names are part of the UI for programmers, so tend towards named types for code that has more clients (e.g., public APIs).


```swift
// Eventually
struct QueryResult {
    let result: Result
    let continuation: () -> Void
}

func fetchResult(from query: Query) -> QueryResult { 
    …
    return QueryResult(result: result) { self.reset() }
}
```

Tuples can also be used to group parallel assignments. This isn't lovely, but is OK for things like swapping the value of variables.

```swift
var x = 20 // fine
var y = 50
	
var (x, y) = (20, 50) // meh

var x = 20, y = 50 // nope
var x = 20; var y = 50 // no way!

(x, y) = (y, x) // yes, swapping
```

It's also possible to use tuples in initializers and for extracting pieces of structs. Avoid this in general.

```swift
public init (x: T, y: U) {
    (self.x, self.y) = (x, y) // too clever, separate lines are more readable and less syntax
    …
    self.x = x // better
    self.y = y
}
```

```swift
// OK if the components really must be extracted
let (w, h) = (size.width, size.height)
let (r, g, b) = (color.red, color.blue, color.green)
```

In Swift, it's often more readable to access the properties at point of use rather than extracting. The optimizer will typically make multiple accesses as efficient as extraction. (There are exceptions, such as computed properties, so use your best judgment and knowledge of the value being accessed.)

Avoid mixing `inout` with return values. Prefer a result enumeration or a tuple instead.

### Assertions

Swift assertions come in two flavors:

- `assert(conditionExpression, "message")` and `assertionFailure("message")` --- used to catch unexpected conditions in debug builds
- `precondition(conditionExpression, "message")` and `preconditionFailure("message")` --- used in cases where continuing might potentially cause additional data loss

The "assertion" flavor crashes in debug builds but do nothing in release builds. The "precondition" flavor crashes in both debug and release builds. As such, `precondition` should be used in cases where continuing might potentially cause additional data loss. That is, use `precondition` where it's safer to crash than go on.

Both flavors are primarily intended to help us catch problems during development. For handling problems in the field, `throw` and `do-try-catch` are the right mechanisms.

#### General guidelines for assertions

- Never introduce a side effect in an assertion.
- Assertions shouldn't check multiple conditions. Instead, break the conditions down into  subexpressions in separate assertions. This makes it easier to pinpoint what went wrong.
- Always add the second message argument to assertions unless it would merely repeat the condition. Prefer to use the message to log values of relevant variables and properties in case the problem proves hard to reproduce. When applicable, use the format “expected: *blah*, got: *baz*" for the message.
- <a name="forceInLieuOfPrecondition">&nbsp;</a>In some circumstances it's legitimate to use force unwrapping (`!`) in lieu of a precondition. For example, when there is no additional information that would be provided in an assertion message beyond what is provided by the system.

```swift
// nope
assert(isFinishingFrobulating && frobulationCount == 0)

// better
assert(isFinishingFrobulating)
assert(frobulationCount == 0, "Expected no frobulations, got \(frobulationCount)")
```

```swift
// nope, no message in assertion
guard !frobulations.isEmpty else {
    assertionFailure() 
    return
}

// better
guard !frobulations.isEmpty else {
    assertionFailure("Expected frobulations.isEmpty, instead was: \(frobulations)")
    return
}
```

### Optionals

When a variable's value may be undefined at any point during its lifetime, represent it with an optional type (though see the subsection on the force operator `!` below).

```swift
class ActionCell: UITableViewCell {
	weak var delegate: Delegate? // must be optional, since it's weak
	var viewModel: ViewModel? // optional since this is only set when cell is dequeued
}
```

Errors and optionals play distinct roles in Swift. Use optionals to represent the absence of a value. Throw an error to explain why something went wrong.

```swift
// Nope! Don't use optionals to indicate failure.
func save() -> URL? {
	...
	guard saveSucceeded else { return nil }
	...
}

// Better. 
func save() throws -> URL {
	...
	guard saveSucceeded else { 
	    throw SaveError(/* interesting error state here */)
	}
	...
}
```

When using conditional binding, prefer to use the same name for the non-optional variable, shadowing the optional one. This is both conventional and avoids the work of coming up with a second reasonable name, at least one of which won’t be the best name.

```swift
guard let handler = handler … // yes
guard let resultHandler = handler … // no

if let completion = completion { // yes
    /* more code here */
    completion()
}

// also OK if no more code
completion?()
```

#### Nil Coalescing

The nil coalescing operator can be handy for providing default values.

```swift
let count = viewModel?.itemCount ?? 0
```

Never use nil coalescing to control side effects. Who would do that? Swift isn't Ruby.

```swift
popover?.popup() ?? presentNewPopup() // no

if popover?.popup() == nil { // yes
    presentNewPopup()
}
```

#### The Force Operator `!`

As mentioned [above](#forceInLieuOfPrecondition), avoid force unwrapping and forced casts except in cases where failure is a programming error and the generated trap would be as informative as an explicit precondition failure. For example:

```swift
let image = UIImage(named: "someAssetThatMustExist")!
```

Often force operators can be eliminated using other Swift features. For example, consider this loop:

```swift
var viewController: UIViewController? = arg
while viewController != nil {
    if someTest {
        doSometing(viewController!)
    }
    viewController = viewController.parent
}
```

It could be rewritten in Swift like so:

```swift
var viewController: UIViewController? = arg
while let existingParent = viewController {
    if someTest {
        doSometing(existingParent)
    }
    viewController = existingParent.parent
}
```

#### Implicitly Unwrapped Optionals

Use implicitly unwrapped optionals (IUO) *only* for properties that will be assigned to after object initialization but before use. For example, in a `UITableViewHeaderFooterView`, subviews must be added to the `contentView`, but the `contentView` is not available until after the superclasses initializer has been called. 

```swift
class SectionHeaderView: UITableViewHeaderFooterView {
	let title: UILabel // no need for IUO; we can set before super init
	var leadingIndent: NSLayoutConstraint! // appropriate use of IUO
	...
	
	init() {
		title = UILabel()
		super.init(reuseIdentifier: String(describing: SectionHeaderView.self))
		
		// contentView is available now
		contentView.addSubview(title)
		leadingIndent = title.leadingAnchor.constraint(equalTo: contentView.leadingAnchor, constant: 32)
		...
	}
}
```

#### Collection Lookup

In most cases, subscripting into a collection in Swift returns an optional value. For example, in:

```swift
let table: [String: Date] = ...
let key: String = ...

... table[key] ...
```

the type of `table[key]` is `Date?`.

`Array`s and `ArraySlice`s are exceptions to this. If you subscript one of these with an out-of-bounds index, your program will crash. (There are two main arguments for this design. First, the domain of most collections is sparse, but the domain of an array is dense---there are no gaps between the start and end indexes. Second, unchecked access is faster and arrays are somehow more "primitive" than other collections.)

In circumstances where an index is known to be valid, or we should fail fast if it isn't, it's appropriate to use direct array subscripting without checking the index. For example, in a table view data source, we shouldn't be asked for an out-of-bounds index.

```swift
override func tableView(_ tableView: UITableView, numberOfRowsInSection section: Int) -> Int {
	return data.count
}

override func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath) -> UITableViewCell {
	let itemData = data[indexPath.row] // no bounds check needed in general
	...
}
```

In a situation where you do need to validate an index there are several useful options in Swift.

```swift
// use the collection's `indices` property
guard data.indices.contains(index) else { ... }

// use a range check
if (0 ..< itemCount).contains(index) { ... }

// use "safe" subscripting, implemented in OmniFoundation as an extension on Collection
if let item = data[safe: index] { ... }
```

#### Conditional Casting

Prefer conditional casting, `as?`, coupled with `guard let` or `if let` binding when downcasting is needed.

```swift
// Nope
if cell is ActionCell {
    (cell as! ActionCell).delegate = self
}

// Yes
if let actionCell = cell as? ActionCell {
    actionCell.delegate = self
}
```

Although `map` can be used to conditionally apply a block to an optional, this style fails the "too clever" test. Use optional chaining or local variables and binding.

```swift
let table: [String: Date] = ...
let key: String = ...

// Nope
let seconds =  table[key].map({ $0.timeIntervalSinceReferenceDate }) ?? 0

// Yep
let seconds = table[key]?.timeIntervalSinceReferenceDate ?? 0

// Nope
table[key].map { date in
    /* complicated
       multi-line block
       that only runs if we
       got a date from the table */
}

// Yep
if let date = table[key] {
    /* complicated
       multi-line block
       that only runs if we
       got a date from the table */
}
```

#### Binding Optional Values for Mutation

If you need to mutate a value type, it must be bound to a variable not a constant (i.e., `var` not `let`). If the original is `let` bound *and* optional, then you have several choices:

```swift
func something(x: …?)

…

// leave optional and use optional chaining
var x = x
x?.mutate() // fine for single use

// var bind in a guard
guard var x = x else { … }
…
x.mutate() // excellent if early out is needed
…

// var bind in an if
if var x = x { // excellent if scoped binding is needed
	…
	x.mutate()
	…
}

// let bind, then var bind, nope!
guard let x = x else { … }
var x = x
…
x.mutate() // excellent if early out is needed
…

// also nope
if let x = x { 
	var x = x
	…
	x.mutate()
	…
}
```

#### Iterating Collections of Optionals

There are several clean ways to deal with collections whose elements are of an optional type.

You can use `case-let` to bind just the non-nil elements, like so:

```swift
let array: [SomeType?] = …
for case let item? in array {  // yes
	// iterates through each non-nil item in array
	…
}
```

Prefer this `case-let` approach to separate iteration and binding. Don't do this:

```swift
for item in array { // no, requires an extra scope
	if let item = item {
		// iterates through each non-nil item in array
		…
	}
}
```

To filter a collection of optionals to just the non-optional elements, use `compactMap`. 

```swift
let nonNilArray = array.compactMap { $0 } 
```

If the collection itself is of an optional type, prefer nil coalescing over introducing a new scope:

```swift
let items: Set<SomeType>? = …

// nope
if let items = items {
	for item in items { … }
}

// yep
for item in items ?? [] { … }
```

### Chaining Expressions

It's fine to chain expressions, like `image.size.width`

Combine optional chaining and nil coalescing when it makes sense to avoid optional types for variables and constants:

```swift
let size = image?.size.scaled(by: 2.0) ?? .zero
```

If a chain of expressions is so long that you feel compelled to add line breaks before dots for readability, consider whether the reader would be better served if you introduced a local variable.

### Non-catching `try`

In addition to the normal `do-catch` statement with `try` expressions inside, Swift also provides `try!`, which converts a thrown error into a crash, and `try?`, which converts a thrown error into an optional. In general:

- Don't use `try!` in production code.
- Avoid `try?` in production code, though there may be some few cases where nothing sensible can be done with an error. Note that logging is often a sensible thing to do. In some cases `guard-try?` is a good pattern:

```swift
guard let myFile = try? getTheFile() else {
    // log that we couldn't get the thing
    return
}
```


- When catching an error, prefer handling it over just logging it, but prefer logging it over just discarding it. 

### Enums and `Result` Types
 
Prefer enums when only a subset of possible arguments or results are valid. For example, if a handler can receive either `Data` or an `Error`, rather than passing separate parameters of type `Data?` and `Error?`, use the standard library's `Result` type. Similarly, if a payload can only be one of a few known types, define those types in an enum instead of using a single more general superclass, `Any`, or the originating `Data`.

### `for-in` vs. `forEach`

Prefer `for-in` to `forEach` except when the body of the `forEach` is reasonably written with co-linear braces and the $0 shortcut per our earlier guidelines.

```swift
let draggables: [Draggable] = …

// ok
draggables.forEach { infoCenter.stoppedDraggingObject($0) }

// also, ok
let fromulate: (Draggable) -> Void
… // pick a fromulation and assign to `fromulate`
draggables.forEach(body: fromulate)

// nope
draggables.forEach { draggable in
	draggable.embiggen()
	infoCenter.stoppedDraggingObject(draggable)
	if draggable.isCromulent {
		return // in a block, so this is logically a `continue`, not an early out!
	}
}

// Yep
for draggable in draggables {
	draggable.embiggen()
	infoCenter.stoppedDraggingObject(draggable)
	if draggable.isCromulent {
		break
	}
}

```

### `for-in` and `where`

The form of `for` loops with a `where` clause to filter the elements is only rarely useful. This is because the `where` clause doesn't bind a downcast element. For example:

```swift
let ofmObjects: [OFMObject] = …
for task in ofmObjects where task is OFMTask { 
	// `task` is still just an OFMObject here
	let task = task as! OFMTask // always succeeds, but ick
	…
}
```

The `case let` style of bindings works with `for` loops and is a better solution to this problem:

```swift
for case let task as OFMTask in arrayOfOFMObjects {
	// `task` is an OFMTask here, other objects are skipped
	…
}
```

It's sometimes OK to use the `where` clause on a `for` loop when it's strictly to filter the elements processed:

```swift
for task in tasks where !task.isCompleted {
	task.complete()
}
```

though consider whether a filter would be clearer:

```swift
for task in tasks.filter({ !$0.isCompleted }) {
	task.complete()
}
```


### Iterating and Enumerating Collections

Check `isEmpty` rather than `count == 0`. It's often faster, and more readable. 

Avoid indices when you can iterate over the elements directly.

```swift
for i in (0 ..< tasks).count { let t = tasks[0] } // avoid
for task in tasks { … } // better
```

Iterating dictionaries with a tuple is quite stylish. 

```swift
for (term, definition) in glossary {
	print(term, definition)
}

```

In the event that you need both indices and elements from a collection, approach the loop with caution. Slice indices are quirky. `enumerated()` looks really tempting, but probably doesn't do what you expect.

```swift
for i in (0 ..< tasks).count // spans 0 through the number of items in tasks
for i in slice.indices // spans from slice.startIndex to slice.endIndex

let letters = ["a", "b", "c", "d", "e"]
let slice = letters[2 ..< 4]

print(slice.indices) // prints "2 ..< 4" - does NOT start at 0
for (i, letter) in slice.enumerated() {
    print("\(i) => \(letter)") // prints 0=>c and 1=>d - DOES start at 0
}
```

Don't zip a collection with its indices. It's slow, and you could usually get the same thing by iterating over the indices, and using those to look up the members.

```swift
for (index, item) in zip(collection, collection.indices) { // no
	...
} 

for index in collection.indices { // yes, if you need the indices
	let item = collection[index]
	...
}
```

### Switch Statements

Prefer switch statements over complex `if` statements. Series of conditions based on the same variables are often clearer with a switch. 

```swift
// avoid
if droidType == .artoo {...}
else if droidType == .bb {...}
else if droidType == .humanoid {...}

// easier to skim, compiler can enforce all cases covered
switch droidType {
case .artoo:
case .bb:
case .humanoid:
}

// silly, just use `if-else`
switch droid.speaksEnglish {
case true:
case false:
}
```

Unlike Objective-C, Swift cases `break` by default. Don't specify the break if there's other code in the case. If a case is deliberately empty, use `break` to reassure the compiler and future readers. 
```swift
switch situation {
case .inABox: // yes, break implied
	print("green eggs and ham")
case .withAFox: // yes, explicit break
	break
case .inTheRain: // won't compile without code
case .onATrain: // no, no, no
	() 
}
```

Prefer reorganizing your logic, or calling the same helper method from multiple cases, over `fallthrough`.

```swift
switch situation { // no
case .here:
	fallthrough
case .there:
	return "I will not eat them, Sam I Am."
...
}

switch situation { // yes
case .here, .there:
	return "I will not eat them, Sam I Am."
...
}

switch situation { // no
case .withAFox:
	sayNot(.withAFox)
	fallthrough
case .here, .there:
	sayNot(.here)
	sayNot(.there)
	sayNotAnywhere()
	sayNoToSamIAm()
}

// yes
func sayFinalLines() {
		sayNot(.here)
	sayNot(.there)
	sayNotAnywhere()
	sayNoToSamIAm()	
}

switch situation { 
case .withAFox:
	sayNot(.withAFox)
	sayFinalLines()
case .here, .there:
	sayFinalLines()
}
```

In a situation that doesn't *quite* fit a tidy switch, a `where` clause might be the tidiest accommodation.

```swift
func buyDroid(jawa: Jawa, d: Droid) {
    switch d {
    case .bb:
    	wrongPlanet()
    case .dalek:
    	wrongGalaxy()
    case .humanoid where d.speaksBinaryLanguageOfMoistureVaporators(), .artoo where !d.hasBadMotivator():
        giveSomeMoney(jawa)
        tellLukeToCleanUp(d)
    default:
        tell(jawa, "what are you trying to pull on us?")
    }
}
```

Prefer specifying every case, unless several can be handled in `default`. If you know your cases are exhaustive, but the compiler doesn’t, use a well-stated assertion failure to maintain this.

Don't use parentheses on `if` and `switch` conditions unless they're actually needed.

### Declaring Number Constants and Variables

Prefer `Int` over explicitly sized integer types.

Prefer `Int` over `UInt`, even when you're sure they're all positive. 

Prefer `Double` over `Float`.

### Implementing Getters and Setters

Let `get` be inferred. 

```swift
var area: Double { // yes
	return width * height
}

var area: Double { // no
	get { 
		return width * height 
	}
}
```

Prefer multi-line computed properties for consistency. They're also less likely to be mistaken for declarations. 
```swift
var area: Double { // yes
	return width * height
}

var area: Double { return width * height } // no
```

Prefer the default `newValue` and `oldValue` over custom names. Do not explicitly declare `newValue` or `oldValue`. 

```swift
set { // yes
	if (newValue > oldValue) {...}
}
willSet {...} // yes (magically gets newValue)
didSet {...} // yes (magically gets oldValue)

set(newValue) {...} // no, dangerous

set(temperature) {...} // tolerable, if you have a good reason
```

### Case-Binding Syntax

Consider a Value enum with cases of different numbers of payload values:

```swift
enum Value<T> {
    case single(T)
    case double(T, T)
    case triple(T, T, T)
}
```

When binding variables to enum case payloads, place the `let` or `var` *inside*
the parentheses.

```swift
if case .double(let first, let second) = value { … } // yes
if case let .double(first, second) = value { … } // no

switch value {
    case .single(let associated): … // yes
    case let .triple(a, b, c): … // no
}
```

This allows for existing variables to be used in a match:

```swift
let constant = 1

if case .double(let bound, constant) = value { … } // yes - matches against constant
if case let .double(bound, constant) = value { … } // no - shadows constant
```

It also allows for mixing of `let` and `var` bindings:

```swift
if case .double(let immutable, var mutable) = value { … } // yes

if case let .double(immutable, _mutable) = value { // no
    var mutable = _mutable // absolutely not
}
```

When not binding *any* values out of an enum case, drop its payload entirely:

```swift
if case .double = value { … } // yes
if case .double(_, _) = value { … } // no
```

#### If-Case vs. Switch

In all of these different situations, a `switch` statement is always acceptable
instead of `if case`. On the other hand, an `if case` is only acceptable for a
single pattern match:

```swift
// yes
switch value {
    case .double: …
    default: …
}

// yes
if case .double = value { … }

// yes
switch value {
    case .single: …
    case .double: …
    case .triple: …
}

// no
if case .single = value { …
} else if case .double = value { …
} else if case .triple = value { …
}
```

### Using If/Guard-Case

Checking cases out of an enum is a special case of **pattern matching**.
Performing a pattern match in an `if` or `guard` can be accomplished two ways:
with `if case` and `guard case`, or with the `~=` operator.

Consider a basic Result enum with no success payload:

```swift
enum Result {
    case success
    case failure(Error)
}
```

Prefer to use the `if case`/`guard case` variant when matching enum types. The
word "case" lends familiarity with other enum uses, and is readable even when
not binding values:

```swift
if case .success = result { … } // yes
if case .failure(let e) = result { … } // yes

if .success ~= result { … } // no
// and it's impossible to bind values using ~=
```

Never explicitly call the `~=` operator. Instead, use more verbose API for all
other types, such as Ranges:

```swift
let range = 1 ..< 10
let five = 5

if range.contains(five) { … } // yes
if range ~= five { … } // no
if case range = five { … } // no
```

Override the `~=` operator only in rare cases where additional pattern matching
support inside `switch`, `if case`, or `catch` clauses is important.

### Void and ()

Use `Void` for type names.
Use `()` for argument signatures and values.
Use inferred void for method and function return types.
Use explicit `Void` when currying and typing closures. 

```swift
func f<T>(label: T) // yes
func f(action: (T) -> Void) // yes
func f(S) -> (T, U) -> (V) -> Void // yes

func f() -> () // no
func f<T>(label: T) -> Void // no
```

If you're never going to return, use `Never`.


### Grouping Initializers

When there's configuration that needs to happen after an initializer, but before the instance is truly ready for use, do *something* with the structure of your code to communicate that.

_Swift Style_ proposes three different approaches, but we felt each of them was problematic. Comments and/or blank lines can communicate this just fine.


### Using Call Site Inferencing

Prefer to omit types whenever you can do so without confounding the compiler. 

Design APIs that support and reinforce type inferencing.

```swift
// noisy
let constraint = NSLayoutConstraint(item: view1, 
									attribute: NSLayoutAttribute.leading, 
									relatedBy: NSLayoutRelation.equal, 
									toItem: view2, 
									attribute: NSLayoutAttribute.leading, 
									multiplier: 1.0, 
									constant: 0.0)

// easier to find the information, and would fit on fewer 
// lines if it weren't hard-wrapped.
let constraint = NSLayoutConstraint(item: view1, 
									attribute: .leading, 
									relatedBy: .equal, 
									toItem: view2, 
									attribute: .leading, 
									multiplier: 1.0, 
									constant: 0.0)

```

### Choosing Capture Modifiers

Create a `strong` reference within a closure, rather than referring to the `weak` one repeatedly. When capturing `self`, shadow the name `self` in Swift 4.2, or use `strongSelf` in older Swift. Never use `strelf` or `this`.

```swift
// yes:
[weak self] in 
guard let self = self else { return } // Swift 4.2 or newer
self.foo()
self.bar()

// no:
[weak self] in
self?.foo()
self?.bar()
```

`-Warc-repeated-use-of-weak` enforces this guideline in Objective-C, but doesn't work in Swift (yet?).

Don't use `unowned` unless you really need it. (`unowned` is equivalent to Objective-C's `unsafe-unretained`.)

## API

### Adopting Access Control

#### Further Reading

[Official Swift reference](https://developer.apple.com/library/content/documentation/Swift/Conceptual/Swift_Programming_Language/AccessControl.html)

[SE-117 introduced `open` in Swift 3](https://github.com/apple/swift-evolution/blob/master/proposals/0117-non-public-subclassable-by-default.md)

#### Subclassing

- Prefer `public` over `open` until you make an intentional choice to allow subclassing. 
- Prefer `static` members to `class` members, unless you mean to allow overrides.
- Protocols are often a better option than subclasses. 

Let Swift infer `final` for performance benefits. This may happen if things are `internal`, `fileprivate`, or `private`, but NOT `public`. If you want to communicate that a class is not designed for subclassing, go the extra mile and state `final` explicitly.

#### Placing Access Modifiers

When the default access modifier is what you want, just omit it.

#### Leading with Access

Access control must come before `let` and `var`. 
For consistency and scan-ability, place it before `static` and `class` declarations as well. 

```swift
fileprivate let y = 20 // compiles
let fileprivate x = 10 // error

open class MyClass { // (class open also won't compile)
	open class func foo() {} // yes
	class open func bar() {} // no, even though it compiles
	
	private static func baz() {} // yes
	static private func qux() {} // no, even though it compiles
}
```

#### Privacy Hints (Underscore)

Don't use underscore to name private entities. The `private` or `internal` qualifier should be sufficient. 


Sometimes we have to set a more permissive access control than we really mean. For this overstated scoping, you may use an underscore. But prefer to use lots of comments and/or an extension instead.

#### Exposing APIs

Start out exposing as little as possible. Prefer to solve real problems, not theoretical ones. Justify everything you include. Emphasize safety, speed, and expressiveness.

You can add more details, or give them more permissive access, but it's painful to consumers to go the other way. If you must go more restrictive, document and explain your deprecations.

### Avoiding Global Symbols

Namespace constants into a type where they naturally fit. (Remember that you can extend Apple-supplied types to do this.)

Namespace functions into the types and protocols they service. If a static function takes an instance of the type as its first argument, consider making it an instance method instead.

Choose wisely between properties and methods that take no arguments. Properties express instance characteristics and should not have side effects.

Move implementations of existing operators into the types they service, so you reduce the number of operators implemented in the global namespace. (Custom operators are [their own thing](#customOperators).)

Don't namespace anything with Objective-C style prefixes. Prefer to namespace by embedding things into static type members.

### Nesting Functions

Consider embedding functions that will be called by only a single method client into that method, rather than create a second method that exists to serve only one parent. However, beware of implicit retain cycles if calling the embedded function asynchronously.

```swift
func foo() { // yes, non-escaping
    func bar(x: X) -> y: Y {
        // something
    }
    let ys = xs.map(bar)
}

func foo() { // no, cycle of sadness
    func bar() {
        // something
    }
    DispatchQueue.main.async {
        bar()
    }
}
```


### Nesting Types

Nest types rather than add items to the global namespace that are of interest only within a parent type.

A hint that it's time to do this is when you're about to give a type a name that contains another type's name. 
`AnnotatedGraph` and `AnnotatedGraphAnnotation` suggests maybe the annotation type should be nested as `AnnotatedGraph.Annotation`.

You can even:

* nest types inside functions
* use `NS_REFINED_FOR_SWIFT` to make top-level Objective-C types appear nested in Swift


### Designing Singletons

Swift makes it pretty easy to code a singleton:

```swift
public final class Singleton {
	public static let shared = Singleton()
	private init() { }
}
```

Use `shared` for the instance and make the class `final`.

Beware of thread safety issues.

### Adding Custom Operators

<a name="customOperators">&nbsp;</a>Use them extremely sparingly. They're hard to recall when trying to use them and hard to recognize when trying to read code that uses them. Even symbols that have a standardized math meaning can be harder to use than a named method like `formUnion(with:)`.

Choose the operator's precedence wisely.

### Overloading Existing Operators

Overloading existing operators with consistent meanings is more often a good idea. These should generally be defined within the type they apply to. 

```swift
// no
struct Foo: Equatable {
    var bar: String
}
func ==(lhs: Foo, rhs: Foo) -> Bool {
    return lhs.bar == rhs.bar
}

// yes
struct Foo: Equatable {
    var bar: String
    static func ==(lhs: Foo, rhs: Foo) -> Bool {
        return lhs.bar == rhs.bar
    }
}
```

### Naming Generic Parameters

Use semantically appropriate single capital letters for generics. Use whole words in UpperCamelCase for associated types. 

```swift
func foo<T: Sequence, U: Hashable>(seq: T, key: U) { … } // no
func foo<SequenceType: Sequence, HashableType: Hashable>(seq: SequenceType, key: HashableType) { … } // no
func foo<S: Sequence, H: Hashable>(seq: S, key: H) { … } // yes
```

```swift
protocol CustomIndexable {
    associatedtype T // no
    associatedtype I // no
    associatedtype Index // maybe
    associatedtype IndexType // yes
}

protocol CustomIterable {
    associatedtype T: IteratorProtocol // no
    associatedtype I: IteratorProtocol // no
    associatedtype Iterator: IteratorProtocol // ok
    associatedtype IteratorType: IteratorProtocol // yes
}
```

### Naming Symbols

#### Casing

Use lowerCamelCase for variables, constants, and type members (including enumerations) and UpperCamelCase for types and protocols. Notably, top-level constants are also lowerCamelCase.

#### Concision

* Prefer shorter names for symbols you use often and those with limited scope. 
* Prefer longer, more explanatory names for things with broader scope. 
* Mark complicated or nontrivial methods and functions with long names.
* Leverage context and types to shorten names.

```swift
for i in indexes { ... } // ok
for index in indexes { ... } // better
for element in list { ... } // yes

struct I { // no
	func i() -> T { ... } // no
}

struct MemberQueue {
	var latency: T // yes
	func makeIterator() -> T { ... } // yes
}

// At the top level...
var count = 0 // no, broad scope
var databaseAccessCount = 0 // yes, broad scope
```

Prefer names that are clear at the point of use over those that describe the implementation.

```swift
struct MyOrderedDictionary<KeyType: Hashable, ObjectType> {
    private var orderedKeys: [KeyType] = []
    private var objects: [KeyType: ObjectType] = [:]

    func reorderKeys(using comparator: ((KeyType, KeyType) -> Bool)) { … } // no, reflects internal storage details
    func sort(by: ((KeyType, KeyType) -> Bool)) { … } // yes, only exposes conceptual information
}
```

#### Categorizations

Don't use the word `Type` or other categorizations in concrete symbol names. Focus on the construct's role, not its implementation details. (One exception is `associatedtype`, as detailed above.)

Names of derived types should reflect their ancestry and inherited capabilities.  

```swift
class Login: UIViewController // no
class LoginViewController: UIViewController // yes
```

#### Preserving Concepts

Do not use the name of another type as a symbol. 

```swift
let url = "http://www.apple.com" // no
let urlString = "http://www.apple.com" // yes, or
let url = URL(string: "http://www.apple.com")! // yes

let personImage = UIImageView() // no
let personImage = UIImage(named: "person") // yes
```

#### Acronyms and Initialisms

An acronym should be capitalized unless it appears at the start of a lowerCamel symbol.

```swift
let URLString = "http://www.apple.com" // no
let urlString = "http://www.apple.com" // yes

let storeurlString = "http://store.apple.com" // no
let storeUrlString = "http://store.apple.com" // no
let storeURLString = "http://store.apple.com" // yes
```

Lowercase all letters of a mixed-case brandname or acronym at the start of a lowerCamel symbol and capitalize the first letter for an UpperCamel symbol. 

```swift
let ipadIcon = ... // yes
protocol IPadRepresentable { ... } // yes
```

#### Terms of Art

Some problem domains have unusual but well-understood initialisms or abbreviations. When these arise, treat them as single words, and let them trump normal rules about labels and casing.

```swift
struct Transform {
    let dX, dY: Float // no
    let dx, dy: Float // yes
}
```

#### Labeling Enumeration Associated Values

Choose the presence or absence of labels based on readability. It's often easy to infer a single associated value from the case name; with multiple associated values, labels can help improve readability.

```swift
enum DropTarget {
    case before(String) // ok
    case after(identifier: String) // also ok
    case between(first: String, second: String) // labels slightly preferred
}
```

Further reading: [SE-0111: Remove type system significance of function argument labels](https://github.com/apple/swift-evolution/blob/master/proposals/0111-remove-arg-label-type-significance.md) (adopted in Swift 3).

### Plurality

Protocols are singular.

Enumerations are singular. 

Classes and Structures are singular. Instances should be singular unless there's a compelling reason. 

Collections and Sequences are a bit of a special case. The type should still be singular, but prefer plural words for collection instances. Also pluralize collection labels in method and function names.

```swift
let mixedItems: [AnyObject] = [s, v] // yes
let mixedArray: [AnyObject] = [s, v] // less desirable
let array: [AnyObject] = [s, v] // much less desirable

let numbers: [Int] = [1, 5] // yes
let numberArray: [Int] = [1, 5] // less desirable
let words = WordSequence(length: .random) // yes
```

### Choosing Label Names

Use external labels to support call sites by introducing the parameter. Use the internal name to describes the parameter's role for its implementation. Do not include the type of the parameter in its name(s).

When working with type-erased types (Any, AnyHashable, AnyObject) or root types (NSObject), add nouns that describe the argument's role using verbNoun form. For example, in `addObserver(_ observer: NSObject)`, "observer" supplements "add".

#### First Labels

Prefer `animate(withDuration:completion:)` to `animateWithDuration(_:, completion:)`. Swift prefers explicit first labels that describe the argument role.

By contrast, prefer `addSubview(_:)` over `add(subview view:)`. The "duration" argument is a tuning parameter to animation; the "subview" is a core part of the semantics of an "add subview" method. (Consider that UIView has both `addSubview(_:)` and `addGestureRecognizer(_:)`, which would be inferior if they were both `add(…)`.)

This is a gray area, but some ways you might distinguish:

* If a parameter could have a default argument, it should probably go inside the parentheses
* If a method name is too ambiguous to be useful without a label, the label should probably go outside the parentheses

#### Conventions

Choose names that follow the conventions of the Objective-C translator, considering [SE-0005](https://github.com/apple/swift-evolution/blob/master/proposals/0005-objective-c-name-translation.md).

#### Indistinguishable Arguments

Omit labels for indistinguishable arguments with identical roles. Labels aren't required for `min(number1, number2)`, `zip(sequence1, sequence2)`, or `sum(value1, value2, value3, …)`.

`min`, `zip`, `sum` are terms of art. Your method names will almost always be longer.

Incorporate variadics to accept an arbitrary number of indistinguishable arguments. Place variadic arguments as close to the end of the signature as possible, all the way up to your defaulted arguments. (It's a little strange to place anything after a variadic argument, so exercise this option sparingly.)

#### Prepositions

Move prepositions out of a call rather than using it to balance the call:

```swift
moveTo(x:, y:) // yes
move(toX:, y:) // unbalanced
move(toX:, toY:) // no
```

#### Meaning

Labels are especially important when their absence conveys incoherent thoughts. This is nigh-mandatory for booleans: don't leave them unlabeled.

```swift
words.split(12) // no — what does the 12 mean?
words.split(maxSplits: 12) // yes
dismiss(true) // no
dismiss(animated: true) // yes
```

#### Subscripting

Label names can describe a subscript's role. For collections, lists, and sequences, this role should distinguish a labeled call from standard label-free usage.

```swift
objects[2] // yes, standard
objects[name: "xyzzy"] // yes — label distinguishes behavior from standard key lookup
```

There's no harm in adding labels to subscript calls with nonstandard or custom semantics, especially for extensions on collection types. However, avoid labels when implementing conventional subscripting lookups. Prefer `customCollection["key"]` to `customCollection[key: "key"]`.

Types that fall outside the bounds of collections, lists, and sequences may also implement subscripts. The best candidates match a type that uses an underlying collection, list, or sequence in its implementation with a subscript API.

```swift
class ViewCache {
    private var viewsByIdentifier: [String: UIView] = [:]

    // yes
    subscript(_ identifier: String) -> UIView? {
        return viewsByIdentifier[identifier]
    }
}
```

Avoid getter subscript syntax for non-lookup behavior; write that behavior a function instead, or implement a setter subscript.

```swift
extension ViewCache {
    // no — does creation, not lookup
    subscript(new identifier: String) -> UIView {
        let view = CustomView()
        viewsByIdentifier[identifier] = view
        return view
    }
}
```

Avoid implementing multiple custom subscripts on an object that return results from different collections.

```swift
extension ViewCache {
    // sure — this object is a collection of views
    subscript(view identifier: String) -> NSView {
        return viewsByIdentifier[identifier]
    }

    // no — looks at a different collection and returns a different type
    subscript(gestureRecognizer identifier: String) -> NSGestureRecognizer { … }
}
```

### Initializers

Don't try to turn initializers into sentences or grammatically correct phrases. They won't read well at call sites.

```swift
class Gizmo {
    init(with widget: Widget, and sprocket: Sprocket) // no — callers type Gizmo(with:and:)
    init(widget: Widget, sprocket: Sprocket) // yes — callers type Gizmo(widget:sprocket:)
}
```

If the initializer *both* converts a type and is lossless, omit the first argument label:

```swift
struct UInt64 {
    init(_ uint32: UInt32) // yes
}
UInt64(some32BitValue) // yes
```

If the initializer is not lossless, use a first argument label that warns of the danger:

```swift
struct UInt32 {
    init(truncating uint64: UInt64) // yes
}
UInt32(truncating: some64BitValue) // yes
```

Include exact member names for memberwise initializers, even if the initializer does not cover every member of the initialized type.

```swift
struct Rect {
    var x, y, width, height: Float

    init(x: Float, y: Float, width: Float, height: Float) // yes
    init(x: Float, y: Float) // yes
    init(originX: Float, originY: Float, wide: Float, tall: Float) // no
}
```

#### Convenience Initializers

Prefer convenience initializers whenever there's a simpler way to build new instances than memberwise. Convenience initializers should not hide API details, obscure intent, or confuse callers. Prefer semantic meaning and clarity over implementation details.

```swift
class Filter<T: Equatable> {
    init(predicate: NSPredicate)

    convenience init() // no — hides details. What would the result do? 
    convenience init(object: T) // no — obscures intent. Does this filter the given object, or everything else?
    convenience init(predicateFormat: String, argumentArray: [Any]) // no — too close to the implementation.

    convenience init(constantValue: Bool) // probably ok, but consider init(predicate: NSPredicate(constantValue:))
    convenience init(matching object: T) // yes
    convenience init(block: ((T) -> Bool)) // yes
}
```

Use labels in convenience initializers to describe roles of arguments. Avoid inconsistency or imbalance in these labels; pass more appropriate types, or break down arguments, to clearly describe each role.

```swift
class Transform {
    convenience init(translationX: Float, y: Float) // no — "translation" is not balanced
    convenience init(scaleX: Float, y: Float) // no — "scale" is not balanced
    convenience init(xScale: Float, yScale: Float) // yes — balanced and defines roles

    convenience init(translate: CGSize) // no — type is unclear given the label
    convenience init(translate: CGVector) // yes

    convenience init(translate: (dx: Float, dy: Float)) // no — we avoid tuples in initializers in general
}
```

Prefer defining initializers over factory methods or global functions. If you think you need a factory method, try a convenience initializer.

### Default Arguments

Default parameters support concision at call sites. Add defaults for arguments that are typical, obvious, or irrelevant in a majority of use cases. Good candidates are:

* Conventional settings (`delay: 0`, `repeats: false`)
* Nullable items (`nil`)
* Trailing closures (`{}`)
* Option sets  (`[]`)
* Dictionaries (`options: [:]` or `attributes: [:]`)
* Arrays (`options: []` or `attributes: []`)
* Configuration types with singleton defaults (e.g. NSURLSession with the default configuration, or NSOperationQueue arguments that take `.main`)

Use default arguments instead of establishing method families. Move defaulted arguments to the end of the signature, but keep it before any trailing closures.

Default arguments are usually a bigger usability win than a performance cost.

### Protocols

Name protocols using:

* Nouns for "architectural" protocols that define what something is, or
* Adjectives ("-able") for protocols that define what something can do

### Generic "Beautification"

Move functions that are generic in a type's associated types to an extension on the top-level type.

```swift
// no
func finalIndex<T: Collection>(in collection: T, where predicate: (T.Iterator.Element) -> Bool) -> T.Index

// yes
extension Collection {
    func finalIndex(where predicate: (Iterator.Element) -> Bool) -> Index
}
```

### Typealiases

Add aliases for types that are used multiple times throughout a project, where the alias is shorter and cleaner than the type it replaces. Candidates are complex types (often closures):

```swift
typealias Completion = () -> Void // probably not
typealias Completion = (Result, Error) -> Void // yes
typealias ModelFilter = (ModelObject) -> Bool // yes
```

Names that establish new roles for existing simple types:

```swift
typealias IntegerType = Int // no — only renaming
typealias Magnitude = Double // yes — defines a new role
```

And simplified elements from protocols or nested generics:

```swift
typealias Index = Base.Index // yes
```

### Errors

Keep the word "Error" in error type definitions.

```
File.fileNotFound // no
FileError.fileNotFound // better
FileError.notFound // yes
```

## Comments & Documentation

Add header documentation for types and methods if it isn't obvious from the declaration what purpose a given type or method serves. Keep in mind that "non-obvious" can include hidden side effects.

Use Xcode's "Add Documentation" command to produce structured markup for this documentation.

Don't hard wrap comments; let Xcode soft wrap long lines. Use vertical whitespace to help with readability and break up "walls of text." For especially long comments, try to summarize at the beginning.

Explain "what" using code. Explain "why" using comments (and commit messages).

### Jump List

Use `MARK:` comments to group related pieces of code. Avoid using the hyphen (`MARK: -`) where it would interrupt hierarchy inside a scope.

```
extension NSTextField {
    // MARK: Bonuses  // yes
    // MARK: - Bonuses  // no
}
```

### Member Ordering

Try to arrange and group members of a type in a predictable order. While some cases may vary for readability if there is a strong semantic reason, we generally prefer the following order:

* Class or static properties and functions
* Stored properties
* Initializers
* Computed properties and functions ("API")
* Superclass overrides
* Protocol conformances (if required in the main body)
* Private helpers

Properties are the most flexible in this order; for some types, it may be more appropriate to group computed properties together with stored properties, or to otherwise rearrange them throughout the scope. Use your best judgment.

Try to break up implementations using extensions, rather than grouping everything into a single scope. Always pull out protocol conformances to an extension if possible, unless the conformance is a key part of the nature of the type (e.g. implementing a Sequence). Consider breaking out logically grouped functions and properties as well.

## Miscellaneous

Prefer `let` to `var`.

Avoid complex string interpolation. Find ways to break it into multiple statements. 

Place declarations at their first point of use, rather than putting them all at the start of scope.

Don't wrap return values in parentheses.

Embrace `Set`.

Embrace functional programming methods, like `map`, `filter`, and `reduce`.
