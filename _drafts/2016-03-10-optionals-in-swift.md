---
layout: post
title: 'Optionals in Swift'
category: 'Programming'
tags: ['Swift', 'iOS']
---

I've been working with Swift for the past 6 weeks as part of my [CS3217](https://nusmods.com/modules/CS3217). Perhaps a good way to document my learning experience is to blog about it.

As my past projects were mostly in Ruby, Swift has really been a fresh change. The 2 languages were designed based on different philosophies and with different end goals in mind. Ruby values the flexibility that keeps programmers happy and productive. With Ruby, you can do whatever you want, but you're in charge of your own safety. That's certainly not the case with Swift. Swift puts safety above all else. You must know exactly what you're doing.

From my exposure so far, I find Swift immensely structured and well-designed. Among the positives, _Optional_ is particularly useful. Apple has done a pretty good job in its [documentation](https://developer.apple.com/library/ios/documentation/Swift/Conceptual/Swift_Programming_Language/OptionalChaining.html) and you should totally check it out. This post is about my understanding of Optionals.

### So what are _Optionals_?

Let's give this a bit of context. If you've played the Grand Theft Auto (GTA) series, there's a thing called [Nitro](http://gta.wikia.com/wiki/Nitro). Nitros can be inserted into cars to provide sudden bursts of speed when activated. 

![Nitro in action](http://www.g-unleashed.com/upload/screenshots/16_hood_nitro_shot.jpg)

Anyway, I digress. Now let's create a class `Car`.

```swift
class Nitro {
    func activate() {}
    func recharge() {}
}

class Car {
    let model: String
    var nitro: Nitro

    init(model: String, nitro: String) {
        self.model = model
        self.nitro = nitro
    }
}
```

For the purpose of this post, our class `Car` only has 2 properties: `model` and `nitro`. One obvious problem with this design: not all cars have Nitro! But some cars do have it, so we still have to accommodate. How can we do this without creating a subclass? Should we give all the cars without `nitro` a generic default Nitro? It's probably a bad idea.

Sometimes, it's _necessary_ that the value of a property be `nil`.

Enter _Optionals_.

```swift
class Car {
    let model: String
    var nitro: Nitro?

    init(model: String, nitro: Nitro?) {
        self.model = model
        self.nitro = nitro
    }

    // Initializer for Cars without nitro
    convenience init(model: String) {
        self.init(model, nitro: nil)
    }
}
```

Notice that `nitro` is now declared with type `Nitro?`. `Nitro?` is actually a short form for `Optional<Nitro>`. This explicitly declares that `nitro` can hold a value of type `Nitro`, or `nil`, i.e. nothing at all. Adding a `?` after the data type tells the compiler: "Be careful, this value can be `nil`".

In order to get the value of type `X` from an Optional of type `X?`, we need to _unwrap_ it. Remember that `X?` (or `Optional<X>`) is a completely different type from `X`.

## Getting value from an Optional

### Forced Unwrapping

This method uses `!` to unwrap the optional. With `!`, you're telling the compiler: "This is an optional value, but I'm absolutely sure that it's not nil. Let's treat it as its original type". Unwrapping with `!` will return a value of the original type.

```swift
let someNitro = withNitro.nitro!
let thisThrowsRuntimeError = withoutNitro.nitro!
```

In the above piece of code, someNitro has the type `Nitro`. It's important to note the distinction between `Optional` type and original type.

Meanwhile, unwrapping a `nil` value would result in a runtime error being thrown. Your app will crash altogether.

You should use `!` only when you're absolutely sure the optional value is not `nil`. One intuitive way to check is:

```swift
if randomCar.nitro != nil {
    randomCar.nitro!.activate()
}
```
There are neater ways to write this, which we'll discuss later.

### Optional Chaining

This is somewhat the "milder" of the 2 approaches. Imagine you have to activate the nitro of a car:

```swift
let withNitro = Car(model: "With Nitro", nitro: Nitro())
let withoutNitro = Car(model: "Without Nitro")

// This calls activate() as per normal
withNitro.nitro?.activate()
// This whole expression equates to nil, since without.nitro is nil
withoutNitro.nitro?.activate()
```

If there's actually nitro in the car, i.e. `withNitro.nitro != nil`, the code runs normally and nitro in activated. If, however, the car has no nitro (`withoutNitro.nitro == nil`), the whole expression will just be considered as `nil`, instead of throwing runtime error like forced unwrapping.

### "if let" and "guard"

It's pretty inelegant to write `withNitro.nitro?` everytime you want to access a property/method of this optional value. An alternative is to use `if let` to unwrap it once and carry out your operation within the `if` block. Consider a `randomCar`:

```swift
if let n = randomCar.nitro {
    n.activate()
    n.recharge()
} else {
    print("LOL no nitro.")
}
```

Again, if `randomCar.nitro` is `nil`, the whole expression `let n = randomCar.nitro` will be considered as `nil`, and the `else` case will happen.

As mentioned above, it's important to note the distinction between `Optional` type and original type. For the `if let` code above, we used `randomCar.nitro` instead of `randomCar.nitro!` since _the value of conditional binding must have type `Optional`_. And using `randomCar.nitro?` alone without a chain is just a compilation error.

However, all your operations that's related to `randomCar.nitro` must then be contained within this `if` block. Eventually as your code grows more complex, you'll encounter something called the [Pyramid of Doom](https://en.wikipedia.org/wiki/Pyramid_of_doom_(programming)), where there's just too many layers of unwrapping. Fortunately, Swift offers a neat solution to this: the `guard` statement.

You can write:

```swift
guard let someNitro = randomCar.nitro else {
    return
}
someNitro.activate()
someNitro.recharge()
```

It's very clear that `guard` acts as a "checkpoint" that makes sure `randomCar.nitro` is not `nil`.

`guard` and `if let` serve a very similar purpose, but there are situations where one is better than the other.

If you find yourself wraping the whole function inside the `if let` clause, and the `else` case exits the function, `guard` would probably be a better fit.

If you only need to deal with `someCar.nitro` for a certain part of your code, and `someCar.nitro == nil` doesn't lead to the end of the function execution, use `if let`.


