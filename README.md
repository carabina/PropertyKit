<img width="447" alt="PropertyKit" src="https://github.com/metasmile/PropertyKit/raw/master/title.png?raw=true">

[![cocoapods compatible](https://img.shields.io/badge/cocoapods-compatible-brightgreen.svg)](https://cocoapods.org/pods/PropertyKit)
[![carthage compatible](https://img.shields.io/badge/carthage-compatible-brightgreen.svg)](https://github.com/Carthage/Carthage)
[![language](https://img.shields.io/badge/spm-compatible-brightgreen.svg)](https://swift.org)
[![platform](https://img.shields.io/badge/platform-iOS%20%7C%20macOS%20%7C%20tvOS-lightgrey.svg)](https://developer.apple.com/develop/)
[![swift](https://img.shields.io/badge/swift-4.~2-orange.svg)](https://github.com/metasmile/PropertyKit/releases)

Light-weight, strict protocol-first styled PropertyKit helps you to easily and safely handle guaranteed values, keys or types on various situations of the large-scale Swift project on iOS, macOS and tvOS.

## Installation

CocoaPods
```ruby
pod 'PropertyKit'
```

Carthage
```ogdl
github "metasmile/PropertyKit"
```

Swift Package Manager
```swift
.Package(url: "https://github.com/metasmile/PropertyKit.git")
```

[Detail Guide](https://github.com/metasmile/PropertyKit/blob/master/INSTALL.md)

## Modules

- PropertyDefaults - for UserDefaults
- PropertyWatchable - for NSKeyValueObservation
- ~

## PropertyDefaults

The simplest, but reliable way to manage UserDefaults, PropertyDefaults automatically binds value and type from Swift property to UserDefaults keys and values.  And it supports only protocol extension pattern that is focusing on syntax-driven value handling, and it helps to avoid unsafe String key use. Therefore it can be perfectly safe through a Swift coding pattern.

- [x] Swift 4 Codable Support
- [x] Key-Value-Type-Safety, no String literal use.
- [x] Structural Protocol-Driven Pattern
- [x] Permission control
- [ ] Scope Control - File/Class/Struct/Function-Private

### Usage

An example to use with basic Codable types:
```swift
extension Defaults: PropertyDefaults {
    public var autoStringProperty: String? {
        set{ set(newValue) } get{ return get() }
    }
    public var autoDateProperty: Date? {
        set{ set(newValue) } get{ return get() }
    }
}

var sharedDefaults = Defaults()
sharedDefaults.autoStringProperty = "the new value will persist in shared scope"
// sharedDefaults.autoStringProperty == Defaults.shared.autoStringProperty

Defaults.shared.autoStringProperty = "another new value will persist in shared scope"
// Defaults.shared.autoStringProperty == sharedDefaults.autoStringProperty

var localDefaults = Defaults(suiteName:"local")
localDefaults.autoStringProperty = "the new value will persist in local scope"
// localDefaults.autoStringProperty != Defaults.shared.autoStringProperty
```

Directly save/load as Codable type
```swift
public struct CustomValueType: Codable{
    var key:String = "value"
    var date:Date?
    var data:Data?
}
extension Defaults: PropertyDefaults {
    // non-optional - must define the default value with the keyword 'or'
    public var autoCustomNonOptionalProperty: CustomValueType {
        set{ set(newValue) } get{ return get(or: CustomValueType()) }
    }
    // optional with/without setter default value
    public var autoCustomOptionalProperty: CustomValueType? {
        set{ set(newValue) } get{ return get() }
    }
    public var autoCustomOptionalPropertySetterDefaultValue: CustomValueType? {
        set{ set(newValue, or: CustomValueType()) } get{ return get() }
    }
}
```

With this pattern, as you know, you also can control access permission with the protocol. It means you can use 'private' or 'file-private' defaults access.

```swift
// MyFile.swift
fileprivate protocol PrivateDefaultKeysInThisSwiftFile: PropertyDefaults{
    var filePrivateValue: String? {set get}
}

extension Defaults: PrivateDefaultKeysInThisSwiftFile {
    public var filePrivateValue: String? {
        set{ set(newValue) } get{ return get() }
    }
}

// Can access - 👌
Defaults.shared.filePrivateValue
```

```swift
// MyOtherFile.swift

// Not able to access - ❌
Defaults.shared.filePrivateValue
```


## PropertyWatchable

A protocol extension based on NSKeyValueObservation. It simply enables to let a class object become a type-safe keypath observable object. And unique observer identifier will be assigned to all observers automatically. That prevents especially duplicated callback calls and so it can let you atomically manage a bunch of flows between key-value flows and queues.

- [x] Making an observable object with only protocol use.
- [x] Swift property literal based keypath observation.
- [x] Strictful type-guaranteed callback parameter support.
- [x] Automatic unique identifier support.
- [x] File-scoped observer removing support.
- [ ] Queue-private atomic operation support.

### Usage

The simplest example to use
```swift
class WatchableObject:NSObject, PropertyWatchable{
    @objc dynamic
    var testingProperty:String?
}

let object = WatchableObject()
object.watch(\.testingProperty) {
    //object.testingProperty == "some value"
    //Do Something.
}

object.testingProperty = "some value"
```

Typed callback object and NSKeyValueObservedChange<Value>.
```swift
object.watch(\.testingProperty) { (object, changes) in
    object.testingProperty
    //object.testingProperty == "some value"
    //Dd Something.
}
```

Automatic line by line identifier support.
```swift
object.watch(\.testingProperty) {
    // Listening as a unique observer A
}

object.watch(\.testingProperty) {
    // Listening as a unique observer A
}

object.watch(\.testingProperty, id:"myid") {
    // Listening as an observer which has identifier "myid"
}
```

Remove observations with various options.

```swift
// Remove only an observer which has "myid" only
object.unwatch(\.testingProperty, forIds:["myid"])

// Remove all observers that are watching ".testingProperty"
object.unwatch(\.testingProperty)

//Automatically remove all observers in current file.
object.unwatchAllFilePrivate()

//Automatically remove entire observers in application-wide.
object.unwatchAll()
```
