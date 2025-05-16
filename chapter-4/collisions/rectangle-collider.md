# Rectangle Collider



<pre class="language-cpp" data-title="engine/collider.h"><code class="lang-cpp"><strong>namespace ng {
</strong><strong>
</strong><strong>// -----
</strong><strong>class RectangleCollider; // Add: forward declare RectangleCollider.
</strong>
class Collider : public Node {
 public:
  // -----
 
  // --- Add: Specialization for RectangleCollider. ---
  /// @brief Checks for collision with a RectangleCollider.
  /// @param other A constant reference to the other RectangleCollider.
  /// @return True if a collision occurs, false otherwise.
  [[nodiscard]] virtual bool Collides(const RectangleCollider&#x26; other) const = 0;
  // --- End ---

  // -----
};

}  // namespace ng
</code></pre>

{% code title="engine/rectangle_collider.h" %}
```cpp
#pragma once

#include <SFML/Graphics/RenderTarget.hpp>
#include <SFML/System/Vector2.hpp>

#include "collider.h"

namespace ng {

/// @brief Represents a rectangular collider for physics interactions.
class RectangleCollider : public Collider {
 public:
  /// @brief Constructs a RectangleCollider with the specified size.
  /// @param app A pointer to the App instance this collider belongs to. This pointer must not be null.
  /// @param size The size of the rectangle collider.
  RectangleCollider(App* app, sf::Vector2f size);

  /// @brief Returns the size of the rectangle collider.
  /// @return A constant reference to the size vector.
  [[nodiscard]] const sf::Vector2f& GetSize() const;

  /// @brief Checks for collision with another Collider. Uses double-dispatch.
  /// @param other A constant reference to the other Collider.
  /// @return True if a collision occurs, false otherwise.
  [[nodiscard]] bool Collides(const Collider& other) const override;

  /// @brief Checks for collision with a CircleCollider.
  /// @param other A constant reference to the other CircleCollider.
  /// @return True if a collision occurs, false otherwise.
  [[nodiscard]] bool Collides(const CircleCollider& other) const override;

  /// @brief Checks for collision with another RectangleCollider.
  /// @param other A constant reference to the other RectangleCollider.
  /// @return True if a collision occurs, false otherwise.
  [[nodiscard]] bool Collides(const RectangleCollider& other) const override;

 protected:
#ifndef NDEBUG
  /// @brief Draw the collider's bounds for debugging purposes.
  /// @param target The SFML RenderTarget to draw to.
  void Draw(sf::RenderTarget& target) override;
#endif

 private:
  // The size of the rectangle collider.
  sf::Vector2f size_;
};

}  // namespace ng
```
{% endcode %}

{% code title="engine/circle_collider.h" %}
```cpp
class CircleCollider {
public:
  // -----

  // --- Add: Specialization for RectangleCollider. ---
  /// @brief Checks for collision with another RectangleCollider.
  /// @param other A constant reference to the other RectangleCollider.
  /// @return True if a collision occurs, false otherwise.
  [[nodiscard]] bool Collides(const RectangleCollider& other) const override;
  // --- End ---

  // -----
};
```
{% endcode %}

## Rectangle - Rectangle

Two rectangles are "Axis-Aligned Bounding Boxes" (AABBs) when their sides are parallel to the game's X and Y axes. This assumption allows to have much simpler collision detection strategies compared to rotatable rectangles.

Two AABBs are not colliding if they are separated on either the X-axis OR the Y-axis.

* **Check for X-axis non-overlap:**
  * Is the right side of rectangle A to the left of the left side of rectangle B?
  * **OR** is the right side of rectangle B to the left of the left side of rectangle A?
* **Check for Y-axis non-overlap:**
  * Is the bottom side of rectangle A above the top side of rectangle B?
  * **OR** is the bottom side of rectangle B above the top side of rectangle A?

If either the X-axis non-overlap condition is true OR the Y-axis non-overlap condition is true, then the rectangles are NOT colliding.

{% code title="engine/rectangle_collider.cc" %}
```cpp
#include "rectangle_collider.h"

#ifndef NDEBUG
#include <SFML/Graphics/RectangleShape.hpp>
#include <SFML/Graphics/RenderTarget.hpp>
#endif
#include <SFML/System/Vector2.hpp>

#include "app.h"
#include "circle_collider.h"
#include "collider.h"

namespace ng {

RectangleCollider::RectangleCollider(App* app, sf::Vector2f size)
    : Collider(app), size_(size) {
  SetName("RectangleCollider");
}

const sf::Vector2f& RectangleCollider::GetSize() const {
  return size_;
}

bool RectangleCollider::Collides(const Collider& other) const {
  return other.Collides(*this);
}

bool RectangleCollider::Collides(const CircleCollider& other) const {
  // Delegate collision check to the CircleCollider's specific implementation.
  return other.Collides(*this);
}

bool RectangleCollider::Collides(const RectangleCollider& other) const {
  sf::Vector2f pos = GetGlobalTransform().getPosition();
  sf::Vector2f otherPos = other.GetGlobalTransform().getPosition();
  // Calculate the extents of both rectangles in world space.
  sf::Vector2f halfSize =
      size_.componentWiseMul(GetGlobalTransform().getScale()) / 2.F;
  sf::Vector2f otherHalfSize =
      other.size_.componentWiseMul(other.GetGlobalTransform().getScale()) / 2.F;

  // Check for non-overlapping conditions on both x and y axes.
  bool AisToTheRightOfB = pos.x - halfSize.x > otherPos.x + otherHalfSize.x;
  bool AisToTheLeftOfB = pos.x + halfSize.x < otherPos.x - otherHalfSize.x;
  bool AisBelowB = pos.y - halfSize.y > otherPos.y + otherHalfSize.y;
  bool AisAboveB = pos.y + halfSize.y < otherPos.y - otherHalfSize.y;

  // If any of the non-overlapping conditions are true, then they are not colliding.
  return !(AisToTheRightOfB || AisToTheLeftOfB || AisAboveB || AisBelowB);
}

#ifndef NDEBUG
void RectangleCollider::Draw(sf::RenderTarget& target) {
  sf::RectangleShape shape(size_);
  shape.setOutlineColor(sf::Color(0, 255, 0, 150));
  shape.setOutlineThickness(2);
  shape.setFillColor(sf::Color::Transparent);
  shape.setOrigin(size_ / 2.F);
  target.draw(shape, GetGlobalTransform().getTransform());
}
#endif

}  // namespace ng
```
{% endcode %}

## Circle - Rectangle

* **Find the Closest Point on the Rectangle to the Circle's Center:** The core idea is to find the point on the rectangle that is closest to the center of the circle. If this closest point is within the circle's radius, then they are colliding.
* **Calculate the Distance Between the Circle's Center and the Closest Point:** Then, calculate the distance (squared to avoid the square root) between the circle's center and the closest point on the rectangle: `d^2 = (pos.x - closest_x)^2 + (pos.y - closest_y)^2` .

A collision occurs if `d^2 < r^2` .

{% code title="engine/circle_collider.cc" %}
```cpp
#include "rectangle_collider.h" // Add: Include RectangleCollider.

namespace ng {

// -----

// --- Add: Circle-Rectangle collision detection. ---
bool CircleCollider::Collides(const RectangleCollider& other) const {
  sf::Vector2f pos = GetGlobalTransform().getPosition();
  sf::Vector2f other_pos = other.GetGlobalTransform().getPosition();
  // Calculate the half-extents of the rectangle in world space.
  float half_x =
      (other.GetSize().x * other.GetGlobalTransform().getScale().x) / 2;
  float half_y =
      (other.GetSize().y * other.GetGlobalTransform().getScale().y) / 2;

  // Find the closest point on the rectangle to the circle's center.
  float closest_x =
      std::max(other_pos.x - half_x, std::min(pos.x, other_pos.x + half_x));
  float closest_y =
      std::max(other_pos.y - half_y, std::min(pos.y, other_pos.y + half_y));

  float diff_x = pos.x - closest_x;
  float diff_y = pos.y - closest_y;
  float distance_squared = (diff_x * diff_x) + (diff_y * diff_y);

  float radius = radius_ * std::max(GetGlobalTransform().getScale().x,
                                    GetGlobalTransform().getScale().y);
  return distance_squared <= radius * radius;
}
// --- End ---

}  // namespace ng 
```
{% endcode %}
