# Collisions

## Collision Detection

Collision detection is the process within a game engine that determines if two or more separate objects in the game world are touching or overlapping in space. Its main job is to identify when a collision event occurs. It answers the question: "Are these objects currently intersecting?"

Game engines often represent objects with simpler invisible shapes (like boxes, spheres, capsules, or complex meshes) called **Colliders**.

## Collision Resolution

Collision resolution is what happens **after** a collision has been detected. It defines the **consequences** or the **response** to that collision. Its job is to handle the interaction realistically or according to the game's rules. It answers the question: "Okay, two objects collided. Now what happens?"

For physically accurate responses, the resolution algorithms are pretty advanced, even for simple shapes like rectangles and circles, let alone irregular shape colliders. For this reason, physically based collision resolution will NOT be covered in this guide.

## Collider

The `Collider` class is the abstract base for all colliders, managing its own registration with the scene's physics system. Its main method, `Collides`, currently takes a generic Collider. Future extensions will include specialized `Collides` overloads for concrete types like Rectangle and Circle. Employing double-dispatch, much like the [Visitor](https://refactoring.guru/design-patterns/visitor) pattern, will allow us to define specific collision logic for every pair of collider types, ensuring at compile time that all collider implementations support all necessary collision checks.

{% code title="engine/collider.h" %}
```cpp
#pragma once

#include "node.h"

namespace ng {

/// @brief An abstract base class for all types of colliders used for physics interactions.
class Collider : public Node {
 public:
  /// @brief Constructs a Collider associated with a specific App instance.
  /// @param app A pointer to the App instance this collider belongs to. This pointer must not be null.
  explicit Collider(App* app);

  /// @brief Checks for collision with another Collider.
  /// This method uses double-dispatch to call the correct overload based on the runtime type of 'other'.
  /// @param other A constant reference to the other Collider.
  /// @return True if a collision occurs, false otherwise.
  [[nodiscard]] virtual bool Collides(const Collider& other) const = 0;

 protected:
  void OnAdd() override;
  void OnDestroy() override;
};

}  // namespace ng
```
{% endcode %}

{% code title="engine/collider.cc" %}
```cpp
#include "collider.h"

#include "node.h"
#include "scene.h"

namespace ng {

Collider::Collider(App* app) : Node(app) {}

void Collider::OnAdd() {
  GetScene()->GetMutablePhysics().AddCollider(this);
}

void Collider::OnDestroy() {
  GetScene()->GetMutablePhysics().RemoveCollider(this);
}

}  // namespace ng
```
{% endcode %}
