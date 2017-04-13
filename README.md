# Yet Another Animation Library

Designed for gesture-driven animations. Extremely fast, yet simple and extensible. 

It is written in pure swift 3.1 with protocol oriented design and extensive use of generics.

Compare to other animation libraries, Animate has the following advantages:

**Extremely fast**: 
  * Uses SIMD types and instructions for calculation
  * Better compiler optimization through swift generics
  * Static store for animations, no key lookup when changing animation target

**Simple**:
  * Builtin Curve(Basic), Spring, & Decay animations
  * Super simple API wrapper for UIView
  * Swift generics provide type safe way to assign animation values
  * Observable, including value, velocity, and target value
  * Builtin chaining operator to easily react to changes in value
  * Provide velocity interpolation with gestures

**Extensible**:
  * Supports custom property
  * Supports custom animatable type
  * Supports custom animation

## Installation

```ruby
pod "YetAnotherAnimationLibrary"
```

## Usage

### Animation

```swift
// Spring animation
view.yaal.center.animateTo(CGPoint(x:50, y:100))
view.yaal.alpha.animateTo(0.5, stiffness: 300, damping: 20)

// Curve(Basic) animation
view.yaal.frame.animateTo(CGRect(x:0, y:0, width:50, height:50), duration:0.5, curve: .linear)

// Decay Animation
view.yaal.center.decay(initialVelocity:CGPoint(x:100, y:0))
```

### Observe Changes

```swift
// observe value changes
view.yaal.center.value.changes.addListener { oldVelocity, newVelocity in
  print(oldVelocity, newVelocity)
}
// observe velocity changes
view.yaal.center.velocity.changes.addListener { oldVelocity, newVelocity in
  print(oldVelocity, newVelocity)
}
```

### Chaining Reactions
```swift
// when scale changes, also change its alpha
// for example if view's scale animates from 1 to 0.5. its alpha will animate to 0.5 as well
view.yaal.scale.value => view.yaal.alpha
// equvalent to the following
// view.yaal.scale.value.changes.addListener { _, newScale in
//   view.yaal.alpha.animateTo(newScale)
// }

// optionally you can provide a mapping function in between.
// For example, the following code makes the view more transparent the faster it is moving
view.yaal.center.velocity => { 1 - $0.magnitude / 1000 } => view.yaal.alpha
// equvalent to the following
// view.yaal.center.velocity.changes.addListener { _, newVelocity in
//   view.yaal.alpha.animateTo(1 - newVelocity.magnitude / 1000)
// }
```

### Set Value (Notify listeners)
```swift
// this sets the value directly (not animate to). Change listeners are called.
// Velocity listeners will receive a series of smoothed velocity values.
view.yaal.center.setTo(gestureRecognizer.location(in:nil))
```

## Advance Usages

### React to changes
Animate is very efficient at observing animated value and react accordingly. Some awesome effects can be achieved through observed values.

For example, here is a simple 2d rotation animation thats made possible through observing the center value's velocity.
```swift
   override func viewDidLoad() {
     // ...
     view.addGestureRecognizer(UITapGestureRecognizer(target: self, action: #selector(tap(gr:))))
     squareView.yaal.center.velocity => { $0.x / 1000 } => squareView.yaal.rotation
   }
   func tap(gr: UITapGestureRecognizer) {
       squareView.yaal.center.animateTo(gr.location(in: view))
   }
```
<img src="https://cloud.githubusercontent.com/assets/3359850/24976406/51c0e0ae-1f97-11e7-8e7d-7684a625195f.gif" width="300"/>

----------------------

Animate also provide smooth velocity interpolation when calling `setTo(_:)`. This is especially useful when dealing with user gesture.

For example. the following does a 3d rotate animation when dragged
```swift
override func viewDidLoad() {
    // ...
    squareView.addGestureRecognizer(UIPanGestureRecognizer(target: self, action: #selector(pan(gr:))))
    squareView.yaal.perspective.setTo(-1.0 / 500.0)
    squareView.yaal.center.velocity => { $0.x / 1000 } => squareView.yaal.rotationY
    squareView.yaal.center.velocity => { -$0.y / 1000 } => squareView.yaal.rotationX
}

func pan(gr: UIPanGestureRecognizer) {
    squareView.yaal.center.setTo(gr.location(in: view))
}
```
<img src="https://cloud.githubusercontent.com/assets/3359850/24976408/52d1afe6-1f97-11e7-84ee-356b92076333.gif" width="300"/>

### Custom property

To animate custom property, just create an animation object by calling `SpringAnimation(getter:setter:)`. Use the animation object to animate and set the values. There are 4 types of animations provided by Animate:

* SpringAnimation
* CurveAnimation
* DecayAnimation
* MixAnimation (does all three types of animation)


```swift
class Foo {
    var volumn: Float = 0.0
    lazy var volumnAnimation: SpringAnimation<Float>
        = SpringAnimation(getter: { [weak self] in self?.volumn },
                          setter: { [weak self] newValue in self?.volumn = newValue })
}

volumnAnimation.animateTo(0.5)
```

### Custom Animatable Type

Custom animatable types are also supported. Just make the type conform to `VectorConvertable`.

```swift
// the following makes IndexPath animatable
extension IndexPath: VectorConvertible {
    public typealias Vector = Vector2
    public init(vector: Vector) {
        self.init(item: Int(vector.x), section: Int(vector.y))
    }
    public var vector: Vector {
        return [Double(item), Double(section)]
    }
}
// Can now be used like this
let indexAnimation = SpringAnimation(getter: { self.indexPath },
                                     setter: { self.indexPath = $0 })
indexAnimation.animateTo(IndexPath(item:0, section:0))

// Note that everything is type safe. incorrect type won't be allowed to compile
```

### Custom Animation

Just subclass `Animation` and override `update(dt:TimeInterval)` method.
If your animation need getter & setter support, subclass `ValueAnimation` instead.
Checkout the builtin animations for example.

