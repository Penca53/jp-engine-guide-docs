# Circle Collider

{% code title="engine/collider.h" %}
```cpp
namespace ng {

class CircleCollider; // Add: forward declare CircleCollider.

class Collider : public Node {
 public:
  // -----
 
  // --- Add: Specialization for CircleCollider. ---
  /// @brief Checks for collision with a CircleCollider.
  /// @param other A constant reference to the other CircleCollider.
  /// @return True if a collision occurs, false otherwise.
  [[nodiscard]] virtual bool Collides(const CircleCollider& other) const = 0;
  // --- End ---

  // -----
};

}  // namespace ng
```
{% endcode %}

## Circle - Circle

Two circles collide if the distance between their centers is less than the sum of their radii: Collision occurs if `distance < r1​+r2​`.

The distance between two points requires a square root operation (relatively computationally heavy), but we can avoid it by comparing the squared distance to the squared sum of the radii: Collision occurs if `distance^2 < (r1​+r2​)^2`.

{% code title="engine/circle_collider.h" %}
```cpp
#pragma once

#include "collider.h"

namespace ng {

/// @brief Represents a circular collider for physics interactions.
class CircleCollider : public Collider {
 public:
  /// @brief Constructs a CircleCollider with the specified radius.
  /// @param app A pointer to the App instance this collider belongs to. This pointer must not be null.
  /// @param radius The radius of the circle collider.
  CircleCollider(App* app, float radius);

  /// @brief Returns the radius of the circle collider.
  /// @return The radius of the circle.
  [[nodiscard]] float GetRadius() const;

  /// @brief Checks for collision with another Collider. Uses double-dispatch.
  /// @param other A constant reference to the other Collider.
  /// @return True if a collision occurs, false otherwise.
  [[nodiscard]] bool Collides(const Collider& other) const override;

  /// @brief Checks for collision with another CircleCollider.
  /// @param other A constant reference to the other CircleCollider.
  /// @return True if a collision occurs, false otherwise.
  [[nodiscard]] bool Collides(const CircleCollider& other) const override;

 protected:
#ifndef NDEBUG
  /// @brief Draw the collider's bounds for debugging purposes.
  /// @param target The SFML RenderTarget to draw to.
  void Draw(sf::RenderTarget& target) override;
#endif

 private:
  // The radius of the circle collider.
  float radius_ = 0;
};

}  // namespace ng
```
{% endcode %}

{% code title="engine/circle_collider.cc" %}
```cpp
#include "circle_collider.h"

#ifndef NDEBUG
#include <SFML/Graphics/CircleShape.hpp>
#include <SFML/Graphics/RenderTarget.hpp>
#endif
#include <SFML/System/Vector2.hpp>
#include <algorithm>

#include "app.h"
#include "collider.h"

namespace ng {

CircleCollider::CircleCollider(App* app, float radius)
    : Collider(app), radius_(radius) {
  SetName("CircleCollider");
}

float CircleCollider::GetRadius() const {
  return radius_;
}

bool CircleCollider::Collides(const Collider& other) const {
  return other.Collides(*this);
}

bool CircleCollider::Collides(const CircleCollider& other) const {
  float distanceSquared = (GetGlobalTransform().getPosition() -
                           other.GetGlobalTransform().getPosition())
                              .lengthSquared();
  float combinedRadius =
      (radius_ * std::max(GetGlobalTransform().getScale().x,
                          GetGlobalTransform().getScale().y)) +
      (other.radius_ * std::max(other.GetGlobalTransform().getScale().x,
                                other.GetGlobalTransform().getScale().y));
  return distanceSquared <= combinedRadius * combinedRadius;
}

#ifndef NDEBUG
void CircleCollider::Draw(sf::RenderTarget& target) {
  sf::CircleShape shape(radius_);
  shape.setOutlineColor(sf::Color(0, 255, 0, 150));
  shape.setOutlineThickness(2);
  shape.setFillColor(sf::Color::Transparent);
  shape.setOrigin(sf::Vector2f(radius_, radius_));
  target.draw(shape, GetGlobalTransform().getTransform());
}
#endif

}  // namespace ng
```
{% endcode %}
