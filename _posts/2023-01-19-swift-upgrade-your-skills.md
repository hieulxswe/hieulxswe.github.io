---
title: "Swift Tips & Tricks for 2023"
date: 2023-01-19T00:00:00+00:00
author: Hieu X. Leu
layout: post
permalink: /swift-upgrade-your-skills/
categories: Swift
tags: [swift, ios, dev, developer]
image: /assets/images/profile.jpeg
---
## 1. Shorter If…Else Statements with Ternary Operators
Ternary operators allow us to make if...else statements shorter.

The syntax is:

`condition ? true : false`

For example, let’s compress this if-else expression:

```swift
let money = 100
if money > 0 {
    print("Some money")
} else {
    print("No money")
}
```

This can be written as a one-liner:

`money > 0 ? print("Some money") : print("No money")`

## 2. Destructuring Tuples
Let’s implement a function that returns a tuple that contains a name and an email address:

```swift
func getInfo() -> (name: String, email: String) {
    return (name: "Matt", email: "matt@example.com")
}
```

When accessing the tuple, you can keep the info together by assigning the whole tuple to a single variable by:

```swift
let info = getInfo()
print(info.name)
print(info.email)
```

Output:
```swift
Matt
matt@example.com
```

But you can also extract the name and email by destructuring the tuple into two separate variables by:

```swift
let (name, email) = getInfo()
print(name)
print(email)
```

Output:

```swift
Matt
matt@example.com
```

With destructuring, we can also solve the beginner problem of how to swap two variables without a third helper variable:

```swift
var a = 1
var b = 2
(a, b) = (b, a)
```

## 3. Read a Property Without Being Able to Change It

Say you have a structure for a house that contains the address of the house:

```swift
struct House {
    var address: String
}
```

Let’s make the address property readable but not changeable.

To do this, Swift has a type called public private(set). With this type, you are able to make the address readable but unchangeable.

```swift
struct House {
    public private(set) var address: String
}
```

## 4. Identity Operator
`==` is not the same as `===!`

`==` is the equality operator. It is meant for checking if two Equatable types match, for example:

```swift
"Test" == "Test"
2.0 == 1.0 + 1.0
```

`===` is the identity operator. It can be used to check if two instances of classes are identical.

For example:

```swift
class Fruit {
    var name = "Banana"
}

let fruit1 = Fruit()
let fruit2 = fruit1
fruit1 === fruit2 // returns true
```
In Swift, an instance of a class is a reference to a memory address. This is why they are called reference types in Swift.

The identity operator checks if two classes are identical. In other words, it checks if they point to the same memory address.

If you heard reference type for the first time, make sure to read this.

## 5. Shorthand for Checking Nils in Optionals
Say you have an optional string name that you want to print if it’s not nil. You can use an if-else check to do this:


```swift
var name: String?
if name != nil {
    print(name)
} else {
    print("N/A")
}
```
But there is a shorter way:

`print(name ?? "N/A")`
This is called nil coalescing. It returns the value on the left-hand side if it’s not nil. If it’s nil then it returns the default value provided on the right.

## 6. Default Values for Parameters
In Swift, you can give default values to function arguments:


```swift
func pick(fruit: String = "banana") {
    print("I picked up \(fruit)s")
}
```
This way you can call the function pick() with and without a parameter

```swift
pick()
pick(fruit: "apple")
```
Output:
```swift
I picked up bananas
I picked up apples
```

## 7. Computed Properties
Computed properties are properties that get computed when you try to access a property.

For example, let’s first use a function to convert the kilos to pounds:

```swift
var kilos = 100.0
func kilosToPounds() -> Double {
    return kilos * 2.205
}
```

But you can also create a pounds variable that is a computed property. In other words, it implements a getter method that computes a value when accessing the property:

```swift
var kilos = 100.0
var pounds: Double {
    get {
        return kilos * 2.205
    }
}
```
When you call:
`print(pounds)`
The pounds variable’s getter method computes the pounds based on the number kilos. The result in this case is:

`220.5`
Read more about computed properties in Getters and Setters in Swift.

## 8. Check If All Items in a Collection Satisfy a Condition
Use the allSatisfy() method to check if all values meet a criterion:

```swift
let dailyTemperatures = [101, 105, 108, 110]
let reallyHot = dailyTemperatures.allSatisfy { $0 >= 100 }
print(reallyHot)
```

Output:

`true`
