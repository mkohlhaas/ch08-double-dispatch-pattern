### Double Dispatch Pattern

Double dispatch is a programming technique that selects which function to run
based on the runtime types of two objects—the receiving object and the method
argument—rather than just one. It helps bypass single-dispatch limitations in
languages like Java or C++ to handle complex interactions cleanly.

### How It Works

* First dispatch: You call a method on the first object. The runtime uses its actual class type to pick the correct implementation.
* Second dispatch: Inside that method, the first object calls a method on the second object (often passing itself this), which triggers dynamic binding a second time based on the second object's type.

### UML Diagrams

#### Class Diagram

```
┌────────────────────────────────┐
│        «trait» Shape           │
├────────────────────────────────┤
│ + collide(&self,               │
│     other: &dyn Shape)         │  ① first dispatch  (receiver-driven)
│ + collide_with_asteroid(       │
│     &self, &Asteroid)          │
│ + collide_with_spaceship(      │
│     &self, &Spaceship)         │  ② second dispatch (callback handoff)
└────────────────────────────────┘
        ▲                    ▲
        │                    │
  implements            implements
        │                    │
┌────────────────┐    ┌────────────────┐
│   Asteroid     │    │   Spaceship    │
├────────────────┤    ├────────────────┤
│ collide(       │    │ collide(       │
│   &dyn Shape)  │    │   &dyn Shape)  │
│ collide_with_  │    │ collide_with_  │
│   asteroid     │    │   asteroid     │
│ collide_with_  │    │ collide_with_  │
│   spaceship    │    │   spaceship    │
└────────────────┘    └────────────────┘
```

#### Sequence Diagram — `shape1.collide(shape2)` (Asteroid × Spaceship)

```
 shape1:&dyn Shape         Asteroid self         Spaceship other
        │                         │                       │
        │      collide(other)     │                       │
        │───────────────────────▶│  ① Dynamic dispatch   │
        │                         │      on self          │
        │  collision_with_space-  │                       │
        │      ship(self)         │                       │
        │                         │─────────────────────▶│  ② Dynamic dispatch
        │                         │                       │      on other
        │                         │  "Spaceship hit an    │
        │                         │   Asteroid!" ◀───────│
        │                         │                       │
```

### Common Use Cases

* Collisions in games: Deciding what happens when an object like an Asteroid hits a Spaceship depends on the specific subtype of both objects.
* The Visitor Pattern: Separating algorithms from data structures by letting elements accept a visitor and call back a specific visit() method tailored to their concrete type.
* Binary operations: Math operations where behavior changes depending on whether you are adding an integer to a float, or a fraction to a complex number.

### Rust

In Rust, true double dispatch does not exist natively because Rust relies on single dispatch via trait objects (dyn Trait). To achieve double dispatch, you must explicitly implement the two-step handoff using traits and methods that swap the receiver and the argument. [1, 2] 

### The Problem: Single Dispatch Limitation

Rust only knows the concrete type of the object on which you call a method (the receiver). It does not know the dynamic type of a trait object passed as an argument.

```rust
// This will NOT compile or work polymorphically out of the box:fn collide(a: &dyn Shape, b: &dyn Shape) {
    // Rust cannot dynamically look up a function based on BOTH 'a' and 'b' types at runtime.
}
```
```
