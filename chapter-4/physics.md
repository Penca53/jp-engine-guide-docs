# Physics

## Physics World

The `Physics` class acts as a central manager that keeps track of all the objects in your game that can interact physically. These objects, represented by the Collider class, are registered and unregistered.

When you want to know if a specific object (collider) is colliding with anything else in the game, the `Overlap` function within the physics world does the heavy lifting. It iterates through all the _other_ registered `Collider` objects and checks if they collide with the given collider. The physics world then returns a list of all the `Collider` objects it found overlapping.

{% code title="engine/physics.h" %}
```cpp
#pragma once

#include <unordered_set>
#include <vector>

#include "collider.h"

namespace ng {

/// @brief Manages the physics simulation within a scene, primarily handling collision detection.
class Physics {
  // Collider needs to be able to call AddCollider, and RemoveCollider.
  friend class Collider;

 public:
  /// @brief Checks if a given collider overlaps with any other collider currently in the physics world.
  /// @param collider The Collider to check for overlaps.
  /// @return A vector of pointers to the Colliders that overlaps with the given collider, empty if no overlap is found.
  [[nodiscard]] std::vector<const Collider*> Overlap(
      const Collider& collider) const;

 private:
  /// @brief Adds a collider to the physics world for collision detection. Called by Collider during its addition to a scene.
  /// @param collider A pointer to the Collider to add. This pointer must not be null and the Collider's lifetime should be managed externally to this class.
  void AddCollider(const Collider* collider);

  /// @brief Removes a collider from the physics world. Called by Collider during its removal from a scene.
  /// @param collider A pointer to the Collider to remove. This pointer must not be null and the Collider's lifetime should be managed externally to this class.
  void RemoveCollider(const Collider* collider);

  // A set containing pointers to all colliders in the physics world. The Physics class does not own these pointers.
  std::unordered_set<const Collider*> colliders_;
};

}  // namespace ng
```
{% endcode %}

{% code title="engine/physics.cc" %}
```cpp
#include "physics.h"

#include <cassert>
#include <unordered_set>
#include <vector>

#include "collider.h"

namespace ng {

std::vector<const Collider*> Physics::Overlap(const Collider& collider) const {
  std::vector<const Collider*> collisions;
  // Find all the collisions against all the registered colliders.
  for (const auto* other : colliders_) {
    if (other == &collider) {
      continue;
    }

    if (collider.Collides(*other)) {
      collisions.push_back(other);
    }
  }

  return collisions;
}

void Physics::AddCollider(const Collider* collider) {
  assert(collider);
  colliders_.insert(collider);
}

void Physics::RemoveCollider(const Collider* collider) {
  assert(collider);
  colliders_.erase(collider);
}

}  // namespace ng
```
{% endcode %}

{% code title="engine/scene.h" %}
```cpp
// -----
#include "physics.h" // Add: Include physics.

class Scene {
 public:
  // -----
  // --- Add: Getters for physics object. ---
  /// @brief Returns a constant reference to the Physics engine for the scene.
  /// @return A constant reference to the Physics engine.
  [[nodiscard]] const Physics& GetPhysics();

  /// @brief Returns a mutable reference to the Physics engine for the scene.
  /// @return A mutable reference to the Physics engine.
  [[nodiscard]] Physics& GetMutablePhysics();
  // --- End ---
  // -----
 private:
  // -----
  // --- Add: Physics object. ---
  // Handles the physics simulation for the scene.
  Physics physics_;
  // --- End ---
};
```
{% endcode %}

{% code title="engine/scene.cc" %}
```cpp
#include "physics.h" // Add: Include.

namespace ng {

// -----
// --- Add: Physics getters. ---
const Physics& Scene::GetPhysics() {
  return physics_;
}

Physics& Scene::GetMutablePhysics() {
  return physics_;
}
// --- End ---

}  // namespace ng
```
{% endcode %}
