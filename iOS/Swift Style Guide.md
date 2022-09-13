This style guide is a compilation from Github, Intrepid and Linkedin Style guides. It is a living document which will evolve as Swift does.

# Swift Style Guide
Attempt to encourage patterns that accomplish the following goals:
1. Increased rigor, and decreased likelihood of programmer error
2. Increased clarity of intent
3. Aesthetic consistency
## Coding Style
- Prefer the composition of `map`, `filter`, `reduce`, etc. over iterating when transforming from one collection to another. Make sure to avoid using closures that have side effects when using these methods.
```
// PREFERRED
let stringOfInts = [1, 2, 3].flatMap { String($0) }
// ["1", "2", "3"]
// NOT PREFERRED
var stringOfInts: [String] = []
for integer in [1, 2, 3] {
 stringOfInts.append(String(integer))
}
// PREFERRED
let evenNumbers = [4, 8, 15, 16, 23, 42].filter { $0 % 2 == 0 }
// [4, 8, 16, 42]
// NOT PREFERRED
var evenNumbers: [Int] = []
for integer in [4, 8, 15, 16, 23, 42] {
 if integer % 2 == 0 {
 evenNumbers.append(integer)
 }
}
```
- If a function returns multiple values, prefer returning a tuple to using `inout` arguments (it’s best to use labeled tuples for clarity on what you’re returning if it is not otherwise obvious). If you use a certain tuple more than once, consider using a `typealias`. If you’re returning 3 or more items in a tuple, consider using a `struct` or `class`instead.
```
func pirateName() -> (firstName: String, lastName: String) {
 return ("Guybrush", "Threepwood")
}
let name = pirateName()
let firstName = name.firstName
let lastName = name.lastName
```
Be wary of retain cycles when creating delegates/protocols for your classes; typically, these properties should be declared `weak`
- Avoid writing out an `enum` type where possible - use shorthand.
```
// PREFERRED
imageView.setImageWithURL(url, type: .person)
// NOT PREFERRED
imageView.setImageWithURL(url, type: AsyncImageView.Type.person)
```
- When writing methods, keep in mind whether the method is intended to be overridden or not. If not, mark it as `final`, though keep in mind that this will prevent the method from being overwritten for testing purposes. In general, `final` methods result in improved compilation times, so it is good to use this when applicable. Be particularly careful, however, when applying the `final` keyword in a library since it is non-trivial to change something to be non-`final`in a library as opposed to have changing something to be non-`final` in your local project.
- Prefer `static` to `class` when declaring a function or property that is associated with a class as opposed to an instance of that class. Only use `class` if you specifically need the functionality of overriding that function or property in a subclass, though consider using a `protocol` to achieve this instead.
- If you have a function that takes no arguments, has no side effects, and returns some object or value, prefer using a computed property instead.
### Access Modifiers
- Write the access modifier keyword first if it is needed.
```
// PREFERRED
private static let myPrivateNumber: Int
// NOT PREFERRED
static private let myPrivateNumber: Int
```
- In general, do not write the `internal` access modifier keyword since it is the default.
- If a property needs to be accessed by unit tests, you will have to make it `internal` to use `@testable import ModuleName`. If a property *should* be private, but you declare it to be `internal` for the purposes of unit testing, make sure you add an appropriate bit of documentation commenting that explains this. You can make use of the `- warning:` markup syntax for clarity as shown below.
```
/**
 This property defines the pirate's name.
 - warning: Not `private` for `@testable`.
 */
let pirateName = "LeChuck"
```
- When choosing between `public` and `open`, prefer `open` if you intend for something to be subclassable outside of a given module and `public` otherwise. Note that anything `internal` and above can be subclassed in tests by using `@testable import`, so this shouldn't be a reason to use `open`. In general, lean towards being a bit more liberal with using `open` when it comes to libraries, but a bit more conservative when it comes to modules in a codebase such as an app where it is easy to change things in multiple modules simultaneously.
### Switch Statements and `enum`s
- When using a switch statement that has a finite set of possibilities (`enum`), do *NOT* include a `default` case. Instead, place unused cases at the bottom and use the `break` keyword to prevent execution.
- Since `switch` cases in Swift break by default, do not include the `break` keyword if it is not needed.
- The `case` statements should line up with the `switch` statement itself as per default Swift standards.
- When defining a case that has an associated value, make sure that this value is appropriately labeled as opposed to just types (e.g. `case Hunger(hungerLevel: Int)` instead of `case Hunger(Int)`).
```
enum Problem {
 case attitude
 case hair
 case hunger(hungerLevel: Int)
}
func handleProblem(problem: Problem) {
 switch problem {
 case .attitude:
 print("At least I don't have a hair problem.")
 case .hair:
 print("Your barber didn't know when to stop.")
 case .hunger(let hungerLevel):
 print("The hunger level is \(hungerLevel).")
 }
}
```
- If you have a default case that shouldn't be reached, preferably throw an error (or handle it some other similar way such as asserting).
```
func handleDigit(_ digit: Int) throws {
 switch digit {
 case 0, 1, 2, 3, 4, 5, 6, 7, 8, 9:
 print("Yes, \(digit) is a digit!")
 default:
 throw Error(message: "The given number was not a digit.")
 }
}
```
- If you don't plan on actually using the value stored in an optional, but need to determine whether or not this value is `nil`, explicitly check this value against `nil` as opposed to using `if let` syntax.
```
// PREFERERED
if someOptional != nil {
 // do something
}
// NOT PREFERRED
if let _ = someOptional {
 // do something
}
```
- When unwrapping optionals, use the same name for the unwrapped constant or variable where appropriate.
```
guard let myValue = myValue else {
 return
}
```
### Using Ternary Operators
The Ternary operator, ?: , should only be used when it increases clarity or code neatness. A single condition is usually all that should be evaluated. Evaluating multiple conditions is usually more understandable as an if statement, or refactored into instance variables. In general, the best use of the ternary operator is during assignment of a variable and deciding which value to use.
```
// PREFERERED
let value = 5
result = (value != 0 ? x : y)
let isHorizontal = true
result = (isHorizontal ? x : y)
// NOT PREFERRED
result = a > b ? x = c > d ? c : d : y
```
### Using `guard` Statements
- When deciding between using an `if` statement or a `guard` statement when unwrapping optionals is *not* involved, the most important thing to keep in mind is the readability of the code. There are many possible cases here, such as depending on two different booleans, a complicated logical statement involving multiple comparisons, etc., so in general, use your best judgement to write code that is readable and consistent. If you are unsure whether `guard` or `if` is more readable or they seem equally readable, prefer using `guard`.
```
// an `if` statement is readable here
if operationFailed {
 return
}
// a `guard` statement is readable here
guard isSuccessful else {
 return
}
// double negative logic like this can get hard to read - i.e. don't do this
guard !operationFailed else {
 return
}
```
Guard statements are meant to be used as early return logic only. They should not be used for regular control flow in place of a traditional control flow statement.
##### Single assignment `guard`
```
guard let value = someMethodThatReturnsOptional() else { return nil }
```
##### Multi assignment `guard`
```
guard
 let self = self,
 let foo = self.editing
 else { return }
```
##### Complex Returns `guard`
This is any situation where you'd want to do more work in the guard than just return or throw.
```
guard let value = someMethodThatReturnsOptional() else {
 doSomeNecessaryThing()
 return nil
}
```
##### Complex Returns (Multi-assignment) `guard`
```
guard
 let self = self,
 let foo = self.editing
 else {
 doSomeNecessaryThing()
 throw Error.FooUnknown
 }
```
##### Error Log Returns  `guard`
```
guard
 let self = self,
 let foo = self.editing
 else {
 doSomeNecessaryThing()
 return(log.error("foo is nil"))
 }
```
**Preferred:**
```
return(log.error("foo is nil"))
```
**Not Preferred:**
```
return log.error("foo is nil")
```
### Code Grouping
Code should strive to be separated into meaningful chunks of functionality. These larger chunks should be indicated by using the `// MARK:` keyword.
When grouping protocol conformance, always use the name of the protocol and only the name of the protocol
**Like this:**
```
// MARK: UITableViewDelegate
```
**Not this:**
```
// MARK: UITableViewDelegate Methods
```
**-- or --**
```
// MARK: Table View Delegate
```
### Magic Numbers/Strings
Do not use magic numbers or Strings in your code. Instead, create variables or constants which accurately explain the value's purpose.
**Like this:**
```
private let labelAnimationDuration = 0.25
private let maxCharacterLimit = 140
func doSomething() {
 guard text.characters.count <= maxCharacterLimit else { return }
 UIView.animateWithDuration(labelAnimationDuration) {
 //do something
 }
}
```
**Not this:**
```
func doSomething() {
 guard text.characters.count <= 140 else { return }
 UIView.animateWithDuration(0.25) {
 //do something
 }
}
```
When performing calculations (especially constraint/view layouts), do not hardcode values, but instead, create mathematical expressions which are flexible, allowing for adjustments to be made without having to manually recalcuate and hardcoded values.
**Like this:**
```
button.layer.cornerRadius = button.frame.height / 2
```
**Not this:**
```
button.layer.cornerRadius = 15.0
```
**Like this:**
```
let coachPageView = "Viewed Coach Page"
tag.event(name: coachPageView)
var pageViewName: String {
 switch self
 case .a, .b, .c: return "Coaching"
 default: return String() //empty Strings should be returned like this
}
```
**Not This:**
```
tag.event(name: "Viewed Coach Page")
func pageViewName() -> String {
 switch self
 case .a, .b, .c: return "Coaching"
 default: return ""
}
```
## Code Organization
Use extensions to organize your code into logical blocks of functionality. Each extension should be set off with a `// MARK: -` comment to keep things well-organized.
### Protocol Conformance
In particular, when adding protocol conformance to a model, prefer adding a separate extension for the protocol methods. This keeps the related methods grouped together with the protocol and can simplify instructions to add a protocol to a class with its associated methods.
**Preferred:**
```
class MyViewController: UIViewController {
 // class stuff here
}
// MARK: - UITableViewDataSource
extension MyViewController: UITableViewDataSource {
 // table view data source methods
}
// MARK: - UIScrollViewDelegate
extension MyViewController: UIScrollViewDelegate {
 // scroll view delegate methods
}
```
**Not Preferred:**
```
class MyViewController: UIViewController, UITableViewDataSource, UIScrollViewDelegate {
 // all methods
}
```
Since the compiler does not allow you to re-declare protocol conformance in a derived class, it is not always required to replicate the extension groups of the base class. This is especially true if the derived class is a terminal class and a small number of methods are being overridden. When to preserve the extension groups is left to the discretion of the author.
For UIKit view controllers, consider grouping lifecycle, custom accessors, and IBAction in separate class extensions.
## Spacing
- Method braces and other braces (`if`/`else`/`switch`/`while` etc.) always open on the same line as the statement but close on a new line.
- Tip: You can re-indent by selecting some code (or ⌘A to select all) and then Control-I (or Editor\Structure\Re-Indent in the menu). Some of the Xcode template code will have 4-space tabs hard coded, so this is a good way to fix that.
**Preferred:**
```
if user.isHappy {
 // Do something
} else {
 // Do something else
}
```
**Not Preferred:**
```
if user.isHappy
{
 // Do something
}
else {
 // Do something else
}
```
- There should be exactly one blank line between methods to aid in visual clarity and organization. Whitespace within methods should separate functionality, but having too many sections in a method often means you should refactor into several methods.
- Colons always have no space on the left and one space on the right. Exceptions are the ternary operator `? :`, empty dictionary `[:]` and `#selector` syntax for unnamed parameters `(_:)`.
**Preferred:**
```
class TestDatabase: Database {
 var data: [String: CGFloat] = ["A": 1.2, "B": 3.2]
}
```
**Not Preferred:**
```
class TestDatabase : Database {
 var data :[String:CGFloat] = ["A" : 1.2, "B":3.2]
}
```
- Long lines should be wrapped at around 70 characters. A hard limit is intentionally not specified.
- Avoid trailing whitespaces at the ends of lines.
- Add a single newline character at the end of each file.
#### Type Annotation for Empty Arrays and Dictionaries
For empty arrays and dictionaries, use type annotation. (For an array or dictionary assigned to a large, multi-line literal, use type annotation.)
**Preferred:**
```
var names: [String] = []
var lookup: [String: Int] = [:]
```
**Not Preferred:**
```
var names = [String]()
var lookup = [String: Int]()
```
**NOTE**: Following this guideline means picking descriptive names is even more important than before.
### Syntactic Sugar
Prefer the shortcut versions of type declarations over the full generics syntax.
**Preferred:**
```
var deviceModels: [String]
var employees: [Int: String]
var faxNumber: Int?
```
**Not Preferred:**
```
var deviceModels: Array<String>
var employees: Dictionary<Int, String>
var faxNumber: Optional<Int>
```
### Lazy Initialization
Consider using lazy initialization for finer grain control over object lifetime. This is especially true for `UIViewController`that loads views lazily. You can either use a closure that is immediately called `{ }()` or call a private factory method. Example:
```
lazy var locationManager: CLLocationManager = self.makeLocationManager()
private func makeLocationManager() -> CLLocationManager {
 let manager = CLLocationManager()
 manager.desiredAccuracy = kCLLocationAccuracyBest
 manager.delegate = self
 manager.requestAlwaysAuthorization()
 return manager
}
```
**Notes:**
- `[unowned self]` is not required here. A retain cycle is not created.
- Location manager has a side-effect for popping up UI to ask the user for permission so fine grain control makes sense here.
## Golden Path
When coding with conditionals, the left-hand margin of the code should be the "golden" or "happy" path. That is, don't nest `if` statements. Multiple return statements are OK. The `guard` statement is built for this.
**Preferred:**
```
func computeFFT(context: Context?, inputData: InputData?) throws -> Frequencies {
 guard let context = context else {
 throw FFTError.noContext
 }
 guard let inputData = inputData else {
 throw FFTError.noInputData
 }
 // use context and input to compute the frequencies
 return frequencies
}
```
**Not Preferred:**
```
func computeFFT(context: Context?, inputData: InputData?) throws -> Frequencies {
 if let context = context {
 if let inputData = inputData {
 // use context and input to compute the frequencies
 return frequencies
 } else {
 throw FFTError.noInputData
 }
 } else {
 throw FFTError.noContext
 }
}
```
When multiple optionals are unwrapped either with `guard` or `if let`, minimize nesting by using the compound version when possible. Example:
**Preferred:**
```
guard let number1 = number1,
 let number2 = number2,
 let number3 = number3 else {
 fatalError("impossible")
}
// do something with numbers
```
**Not Preferred:**
```
if let number1 = number1 {
 if let number2 = number2 {
 if let number3 = number3 {
 // do something with numbers
 } else {
 fatalError("impossible")
 }
 } else {
 fatalError("impossible")
 }
} else {
 fatalError("impossible")
}
```
## Classes, Structs, and Protocols
### Structs vs Classes
Unless you require functionality that can only be provided by a class (like identity or deinitializers), implement a struct instead.
Note that inheritance is (by itself) usually *not* a good reason to use classes, because polymorphism can be provided by protocols, and implementation reuse can be provided through composition.
For example, this class hierarchy:
```
class Vehicle {
 let numberOfWheels: Int
 init(numberOfWheels: Int) {
 self.numberOfWheels = numberOfWheels
 }
 func maximumTotalTirePressure(pressurePerWheel: Float) -> Float {
 return pressurePerWheel * Float(numberOfWheels)
 }
}
class Bicycle: Vehicle {
 init() {
 super.init(numberOfWheels: 2)
 }
}
class Car: Vehicle {
 init() {
 super.init(numberOfWheels: 4)
 }
}
```
could be refactored into these definitions:
```
protocol Vehicle {
 var numberOfWheels: Int { get }
}
extension Vehicle {
 func maximumTotalTirePressure(pressurePerWheel: Float) -> Float {
 return pressurePerWheel * Float(numberOfWheels)
 } 
}
struct Bicycle: Vehicle {
 let numberOfWheels = 2
}
struct Car: Vehicle {
 let numberOfWheels = 4
}
```
**Rationale:** Value types are simpler, easier to reason about, and behave as expected with the `let` keyword.
#### Protocol Naming
Protocols that describe *what something is* should be named as nouns. As per [Apple's API Design Guidelines](https://swift.org/documentation/api-design-guidelines/), a `protocol` should be named as nouns if they describe what something is doing (e.g. `Collection`) and using the suffixes `able`, `ible`, or `ing` if it describes a capability (e.g. `Equatable`, `ProgressReporting`). If neither of those options makes sense for your use case, you can add a `Protocol` suffix to the protocol's name as well. Some example `protocol`s are below.
```
Collection, ViewDelegate, etc.
```
Protocols that describe the *capability* of something should be named using the suffixes `able`, `ible`, or `ing`.
```
Equatable, Reporting, Sustainable, etc.
```
```
// here, the name is a noun that describes what the protocol does
protocol TableViewSectionProvider {
 func rowHeight(at row: Int) -> CGFloat
 var numberOfRows: Int { get }
 /* ... */
}
// here, the protocol is a capability, and we name it appropriately
protocol Loggable {
 func logCurrentState()
 /* ... */
}
// suppose we have an `InputTextView` class, but we also want a protocol
// to generalize some of the functionality - it might be appropriate to
// use the `Protocol` suffix here
protocol InputTextViewProtocol {
 func sendTrackingEvent()
 func inputText() -> String
 /* ... */
}
```
#### Protocol Implementation
When implementing protocols, there are two ways of organizing your code:
1. Using `// MARK:` comments to separate your protocol implementation from the rest of your code
2. Using an extension outside your `class`/`struct` implementation code, but in the same source file
Keep in mind that when using an extension, however, the methods in the extension can't be overridden by a subclass, which can make testing difficult. If this is a common use case, it might be better to stick with method #1 for consistency. Otherwise, method #2 allows for cleaner separation of concerns.
Even when using method #2, add `// MARK:` statements anyway for easier readability in Xcode's method/property/class/etc. list UI.
## Types
### Type Specifications
When specifying the type of an identifier, always put the colon immediately after the identifier, followed by a space and then the type name.
```
class SmallBatchSustainableFairtrade: Coffee { ... }
let timeToCoffee: NSTimeInterval = 2
func makeCoffee(type: CoffeeType) -> Coffee { ... }
func swap<T: Swappable>(inout a: T, inout b: T) { ... }
```
### Let vs Var
Prefer `let`-bindings over `var`-bindings wherever possible
Use `let foo = …` over `var foo = …` wherever possible (and when in doubt). Only use `var` if you absolutely have to (i.e. you *know* that the value might change, e.g. when using the `weak` storage modifier).
**Rationale:** The intent and meaning of both keywords is clear, but *let-by-default* results in safer and clearer code.
A `let`-binding guarantees and *clearly signals to the programmer* that its value is supposed to and will never change. Subsequent code can thus make stronger assumptions about its usage.
It becomes easier to reason about code. Had you used `var` while still making the assumption that the value never changed, you would have to manually check that.
Accordingly, whenever you see a `var` identifier being used, assume that it will change and ask yourself why.
### Parameterized Types
Methods of parameterized types can omit type parameters on the receiving type when they’re identical to the receiver’s.
**Like this:**
```
struct Composite<T> {
 …
 func compose(other: Composite) -> Composite {
 return Composite(self, other)
 }
}
```
**Not this:**
```
struct Composite<T> {
 …
 func compose(other: Composite<T>) -> Composite<T> {
 return Composite<T>(self, other)
 }
}
```
**Rationale:** Omitting redundant type parameters clarifies the intent, and makes it obvious by contrast when the returned type takes different type parameters.
### Operator Definitions
Use whitespace around operators when defining them.
**Like this:**
```
func <| (lhs: Int, rhs: Int) -> Int
func <|< <A>(lhs: A, rhs: A) -> A
```
**Not this:**
```
func <|(lhs: Int, rhs: Int) -> Int
func <|<<A>(lhs: A, rhs: A) -> A
```
**Rationale:** Operators consist of punctuation characters, which can make them difficult to read when immediately followed by the punctuation for a type or value parameter list. Adding whitespace separates the two more clearly.
### Dictionaries
When specifying the type of a dictionary, a colon should only have a space on the right side.
**Like this:**
```
let capitals: [Country: City] = [Sweden: Stockholm]
```
**Not this:**
```
let capitals: [Country: City] = [ Sweden : Stockholm ]
```
For literal dictionaries that exceed a single line, newline syntax is preferable:
**Like this:**
```
let capitals: [Country: City] = [
 Sweden: Stockholm,
 USA: WashingtonDC
]
```
**Not this:**
```
let capitals: [Country: City] = [Sweden: Stockholm, USA: WashingtonDC]
```
### Type Inference
Unless it impairs readability or understanding, it preferable to rely on Swift's type inference where appropriate.
**Like this:**
```
let hello = "Hello"
```
**Not this:**
```
let hello: String = "Hello"
```
This does not mean one should avoid those situations where an explicit type is required.
**Like this:**
```
let padding: CGFloat = 20
var hello: String? = "Hello"
```
**Rationale:** The type specifier is saying something about the *identifier* so it should be positioned with it.
### Enums
Enum cases should be defined in `camelCase` with leading lowercase letters. This is counter to Swift 2.x where uppercase was preferred.
**Like This**
```
enum Directions {
 case north
 case south
 case east
 case west
}
```
**Not This**
```
enum Directions {
 case North
 case South
 case East
 case West
}
```
**Rationale:** Uppercase syntax should be reserved for typed declarations only.
If enum is of a specific type, use rawValue property directly to access its assigned value. Create a computed variable only if some computation needs to be perfomed before returning rawValue.
**Like This**
```
enum Direction: String {
 case north = "North"
 case south = "South"
 case east = "East"
 case west = "West"
 var displayString: String {
 return self.rawValue + "Direction"
 }
}
usage: label.text = Direction.north.displayString
```
**Not This**
```
enum Direction {
 case north = "North"
 case south = "South"
 case east = "East"
 case west = "West"
 var value: String {
 return self.rawValue
 }
}
```
## Optionals
### Force-Unwrapping of Optionals
If you have an identifier `foo` of type `FooType?` or `FooType!`, don't force-unwrap it to get to the underlying value (`foo!`) if possible.
Instead, prefer this:
```
if let foo = foo {
 // Use unwrapped `foo` value in here
} else {
 // If appropriate, handle the case where the optional is nil
}
```
**Rationale:** Explicit `if let`-binding of optionals results in safer code. Force unwrapping is more prone to lead to runtime crashes.
### Optional Chaining
Optional chaining in Swift is similar to messaging `nil` in Objective-C, but in a way that works for any type, and that can be checked for success or failure.
Use optional chaining if you don’t plan on taking any alternative action if the optional is `nil`.
```
let cell: YourCell = tableView.dequeueCell(indexPath)
cell.label?.text = “Hello World”
return cell
```
**Rationale:** The use of optional binding here is overkill.
### Implicitly Unwrapped Optionals
Implicitly unwrapped optionals have the potential to cause runtime crashes and should be used carefully. If a variable has the possibility of being `nil`, you should always declare it as an optional `?`.
Implicitly unwrapped optionals may be used in situations where limitations prevent the use of a non-optional type, but will never be accessed without a value.
If a variable is dependent on `self` and thus not settable during initialization, consider using a `lazy` variable.
```
lazy var customObject: CustomObject = CustomObject(dataSource: self)
```
**Rationale:** Explicit optionals result in safer code. Implicitly unwrapped optionals have the potential of crashing at runtime.
### Getters
When possible, omit the `get` keyword on read-only computed properties and read-only subscripts.
**Like this:**
```
var myGreatProperty: Int {
 return 4
}
subscript(index: Int) -> T {
 return objects[index]
}
```
**Not this:**
```
var myGreatProperty: Int {
 get {
 return 4
 }
}
subscript(index: Int) -> T {
 get {
 return objects[index]
 }
}
```
**Rationale:** The intent and meaning of the first version is clear, and results in less code.
### Referring to `self`
When accessing properties or methods on `self`, leave the reference to `self` implicit by default:
```
private class History {
 var events: [Event]
 func rewrite() {
 events = []
 }
}
```
Only include the explicit keyword when required by the language—for example, in a closure, or when parameter names conflict:
```
extension History {
 init(events: [Event]) {
 self.events = events
 }
 var whenVictorious: () -> () {
 return {
 self.rewrite()
 }
 }
}
```
**Rationale:** This makes the capturing semantics of `self` stand out more in closures, and avoids verbosity elsewhere.
### Make classes `final` by default
Classes should start as `final`, and only be changed to allow subclassing if a valid need for inheritance has been identified. Even in that case, as many definitions as possible *within* the class should be `final` as well, following the same rules.
**Rationale:** Composition is usually preferable to inheritance, and opting *in* to inheritance hopefully means that more thought will be put into the decision.
## Functions
Specifications around the preferable syntax to use when declaring, and using functions.
### Declarations
With Swift 3, the way that parameter names are treated has changed. Now the first parameter will always be shown unless explicitly requested not to. This means that functions declarations should take that into account and no longer need to use long, descriptive names.
**Like this:**
```
func move(view: UIView, toFrame: CGRect)
func preferredFont(forTextStyle: String) -> UIFont
```
**Not this:**
```
func moveView(view: UIView, toFrame frame: CGRect)
func preferredFontForTextStyle(style: String) -> UIFont
```
If you absolutely need to hide the first parameter name it is still possible by using an `_` for its external name, but this is not preferred.
```
func moveView(_ view: UIView, toFrame: CGRect)
```
**Rationale:** Function declarations should flow as a sentence in order to make them easier to understand and reason about.
### Naming
Avoid needless repetition when naming functions. This is following the style of the core API changes in Swift 3.
**Like this:**
```
let blue = UIColor.blue
let newText = oldText.append(attributedString)
```
**Not this:**
```
let blue = UIColor.blueColor()
let newText = oldText.appendAttributedString(attributedString)
```
### Calling
... some specifications on calling functions
- Avoid declaring large arguments inline
- For functions with many arguments, specify each arg on a new line and the `)` on the final line
- Use trailing closure syntax for simple functions
- Avoid trailing closures at the end of functions with many arguments. (3+)
## Closures
### Closure Specifications
It is preferable to associate a closure's type from the left hand side when possible.
**Like this:**
```
let layout: (UIView, UIView) -> Void = { (view1, view2) in
 view1.center = view2.center
 // ...
}
```
**Not this:**
```
let layout = { (view1: UIView, view2: UIView) in
 view1.center = view2.center
 // ...
}
```
### Void arguments/return types
It is preferable to not use `Void` in closure arguments or return types, omitting parentheses where possible.
**Like this:**
```
let noArgNoReturnClosure = { doSomething() } // no arguments or return types, omit both
let noArgClosure = { () -> Int in return getValue() } // void argument, use '()'
let noReturnClosure = { (arg) in doSomething(with: arg) } // void return type, omit return type
```
**Not this:**
```
let noArgNoReturnClosure = { (Void) -> Void in doSomething() } // don't do this
let noArgClosure = { (Void) -> Int in return getValue() } // don't do this
let noReturnClosure = { (arg) -> Void in doSomething(with: arg) } // don't do this
```
However, when defining closure type, `Void` is preferred to `()`
**Like this:**
```
typealias NoArgNoReturnClosure = Void -> Void
typealias NoArgClosure = Void -> Int
typealias NoReturnClosure = Int -> Void
```
**Not this:**
```
typealias NoArgNoReturnClosure = () -> ()
typealias NoArgClosure = () -> Int
typealias NoReturnClosure = Int -> ()
```
### Shorthand
Shorthand argument syntax should only be used in closures that can be understood in a few lines. In other situations, declaring a variable that helps identify the underlying value is preferred.
**Like this:**
```
doSomethingWithCompletion() { result in
 // do things with result
 switch result {
 // ...
 }
}
```
**Not this:**
```
doSomethingWithCompletion() {
 // do things with result
 switch $0 {
 // ...
 }
}
```
Using shorthand syntax is preferable in situations where the arguments are well understood and can be expressed in a few lines.
```
let sortedNames = names.sort { $0 < $1 }
```
### Trailing Closures
Use trailing closure syntax only if there's a single closure expression parameter at the end of the argument list.
**Like this:**
```
UIView.animateWithDuration(1.0) {
 self.myView.alpha = 0
}
```
**Not this:**
```
UIView.animateWithDuration(1.0, animations: {
 self.myView.alpha = 0
})
```
### Multiple Closures
When a function takes multiple closures as arguments it can be difficult to read. To keep it clean, use a new line for each argument and avoid trailing closures. If you're not going to use the variable from the closure input, name it with an underscore `_`.
**Like this:**
```
UIView.animateWithDuration(
 SomeTimeValue,
 animations: {
 // Do stuff
 },
 completion: { _ in
 // Do stuff
 }
)
```
**Not this:**
```
UIView.animateWithDuration(SomeTimeValue, animations: {
 // Do stuff
 }) { complete in
 // Do stuff
}
```
(Even though the default spacing and syntax from Xcode might do it this way)
### Referring to `self`
When referring to `self` within a closure you must be careful to avoid creating a [strong reference cycle](https://developer.apple.com/library/content/documentation/Swift/Conceptual/Swift_Programming_Language/AutomaticReferenceCounting.html#//apple_ref/doc/uid/TP40014097-CH20-ID56). Always use a capture list, such as `[weak self]` when it's necessary to use functions or properties outside of the closure.
**Like this:**
```
lazy var someClosure: () -> String? = { [weak self] in
 guard let self = self else { return nil }
 return self.someFunction()
}
```
**Not this:**
```
viewModel.someClosure { self in
 self.outsideFunction()
}
```
**Rationale** If a closure holds onto a strong reference to a property being used within it there will be a strong reference cycle causing a memory leak.
### Should I use `unowned` or `weak`?
- `weak` is always preferable as it creates an optional so that crashes are prevented. `unowned` is only useful when you are guaranteed that the value will never be `nil` within the closure. Since this creates the possibility for unsafe access it should be avoided.
### Delegates
When creating custom delegate methods, an unnamed first parameter should be the delegate source. (UIKit contains numerous examples of this.)
**Preferred:**
```
func namePickerView(_ namePickerView: NamePickerView, didSelectName name: String)
func namePickerViewShouldReload(_ namePickerView: NamePickerView) -> Bool
```
**Not Preferred:**
```
func didSelectName(namePicker: NamePickerViewController, name: String)
func namePickerShouldReload() -> Bool
```
### Code Formatting
There should be a space before and after a binary operator such as `+`, `==`, or `->`. There should also not be a space after a `(` and before a `)`.
```
let myValue = 20 + (30 / 2) * 3
if 1 + 1 == 3 {
 fatalError("The universe is broken.")
}
func pancake(with syrup: Syrup) -> Pancake {
 /* ... */
}
```
- We follow Xcode's recommended indentation style (i.e. your code should not change if CTRL-I is pressed). When declaring a function that spans multiple lines, prefer using that syntax to which Xcode, as of version 7.3, defaults.
```
// Xcode indentation for a function declaration that spans multiple lines
func myFunctionWithManyParameters(parameterOne: String,
 parameterTwo: String,
 parameterThree: String) {
 // Xcode indents to here for this kind of statement
 print("\(parameterOne) \(parameterTwo) \(parameterThree)")
}
// Xcode indentation for a multi-line `if` statement
if myFirstValue > (mySecondValue + myThirdValue)
 && myFourthValue == .someEnumValue {
 // Xcode indents to here for this kind of statement
 print("Hello, World!")
}
```
- When calling a function that has many parameters, put each argument on a separate line with a single extra indentation.
```
someFunctionWithManyArguments(
 firstArgument: "Hello, I am a string",
 secondArgument: resultFromSomeFunction(),
 thirdArgument: someOtherLocalProperty)
```
- When dealing with an implicit array or dictionary large enough to warrant splitting it into multiple lines, treat the `[`and `]` as if they were braces in a method, `if` statement, etc. Closures in a method should be treated similarly.
```
someFunctionWithABunchOfArguments(
 someStringArgument: "hello I am a string",
 someArrayArgument: [
 "dadada daaaa daaaa dadada daaaa daaaa dadada daaaa daaaa",
 "string one is crazy - what is it thinking?"
 ],
 someDictionaryArgument: [
 "dictionary key 1": "some value 1, but also some more text here",
 "dictionary key 2": "some value 2"
 ],
 someClosure: { parameter1 in
 print(parameter1)
 })
```
- Prefer using local constants or other mitigation techniques to avoid multi-line predicates where possible.
```
// PREFERRED
let firstCondition = x == firstReallyReallyLongPredicateFunction()
let secondCondition = y == secondReallyReallyLongPredicateFunction()
let thirdCondition = z == thirdReallyReallyLongPredicateFunction()
if firstCondition && secondCondition && thirdCondition {
 // do something
}
// NOT PREFERRED
if x == firstReallyReallyLongPredicateFunction()
 && y == secondReallyReallyLongPredicateFunction()
 && z == thirdReallyReallyLongPredicateFunction() {
 // do something
}
```
- All constants other than singletons that are instance-independent should be `static`. All such `static` constants should be placed in a container `enum` type. The naming of this container should be singular (e.g. `Constant` and not `Constants`) and it should be named such that it is relatively obvious that it is a constant container. If this is not obvious, you can add a `Constant` suffix to the name. You should use these containers to group constants that have similar or the same prefixes, suffixes and/or use cases.
```
class MyClassName {
 // PREFERRED
 enum AccessibilityIdentifier {
 static let pirateButton = "pirate_button"
 }
 enum SillyMathConstant {
 static let indianaPi = 3
 }
 static let shared = MyClassName()
 // NOT PREFERRED
 static let kPirateButtonAccessibilityIdentifier = "pirate_button"
 enum SillyMath {
 static let indianaPi = 3
 }
 enum Singleton {
 static let shared = MyClassName()
 }
}
```
- Do not abbreviate, use shortened names, or single letter names.
```
// PREFERRED
class RoundAnimatingButton: UIButton {
 let animationDuration: NSTimeInterval
 func startAnimating() {
 let firstSubview = subviews.first
 }
}
// NOT PREFERRED
class RoundAnimating: UIButton {
 let aniDur: NSTimeInterval
 func srtAnmating() {
 let v = subviews.first
 }
}
```
### Shadows
To implement shadows from Sketch file (bottom outer shadow, bottom inner shadow, modal shadow, banner shadow) use existing function:
```
func applySketchShadow(color: UIColor = .black,
 alpha: Float = 0.5,
 x: CGFloat = 0,
 y: CGFloat = 2,
 blur: CGFloat = 4,
 spread: CGFloat = 0) {
 shadowColor = color.cgColor
 shadowOpacity = alpha
 shadowOffset = CGSize(width: x, height: y)
 shadowRadius = blur / 2.0
 if spread == 0 {
 shadowPath = nil
 } else {
 let dx = -spread
 let rect = bounds.insetBy(dx: dx, dy: dx)
 shadowPath = UIBezierPath(rect: rect).cgPath
 }
}
// PREFERRED
selectTopicView.layer.applySketchShadow(color: Color.red)
// NOT PREFERRED
selectTopicView.layer.shadowColor = Color.red
selectTopicView.layer.shadowRadius = shadowRadiusValue
selectTopicView.layer.shadowOffset = CGSize(width: 0, height: 2)
selectTopicView.layer.shadowOpacity = shadowOpacityValue
```
# Clean Architecture
## Communication between layers
Clean Architecture has 3 layers: Data, Domain and Presentation. Domain is the central layer and it contains business logic. Data and Presentation layers are implementations details for data persistance and user interface. The communication between layers is described by Dependency Rule and this rule states that the inner layers cant talk to the outer layers. Presentation layer is using Use Cases to communicate with Domain and Data layer.
### Data
Data layer contains 2 folders named: local and remote
- local - This folder is responsible for local data storage and it contains
 - Core Data entities
 - Core Data entities should conform `DataModel` protocol that enables conversion to the data model.
 - Local repositories, subclasses of `Repository` class contain methods that work with the Core Data database. `Repository` (base repository) class already has methods for removing, saving, updating and deleting object(s) and local repositories should just override them if it is necessary.
- remote - This folder contains everything related to API and other remote dependencies.
 - Data model - a model that is in use for mapping an API response
 - Data model should conform the `Decodable` or `Encodable` or both protocols in order to map json response in model and vice versa.
 - Data model should conform `DomainModel` in order to convert object to the domain model.
Note: Folders should be presented only in case that a feature needs local and remote repositories. If a feature has only local cache logic, everything should go to the Data folder.
### Domain
Domain layer contains two subfolders: Model and UseCases
- Model folder contains all struct models that are in use for the presentation layer
- UseCases folder contains use case protocols and implementations
### Presentation
Structure of the presentation layer depends on the feature complexity. There are no strict rules around that.
- Each view in the presentation layer should have its own view model.
- View model, besides other properties, should be initialized with the use case from the domain layer
 - Use case property in the init method should protocol
 - View model should be unit tested by providing mocked use case
