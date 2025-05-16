# Mushroom Collisions

## Damaging and Bouncing

The `Mushroom` node now incorporates a child `RectangleCollider` node, intentionally sized slightly smaller than its sprite. Collision detection with the tilemap now relies on this collider's dimensions and position, resulting in more precise hitboxes compared to the sprite's visual bounds. This collider is also used to detect interactions with other mushrooms and bananas. Upon colliding with a falling mushroom, the mushroom is destroyed, and the player receives a small vertical bounce. When a banana is hit, it's considered collected, and a collection sound is triggered. This sound is managed by the player Node to ensure its complete playback, as the banana Node is immediately destroyed upon collection, which would prematurely terminate the sound if played from the banana itself. The player's longer lifespan guarantees the sound finishes.

{% code title="game/mushroom.h" %}
```cpp
// -----
#include "engine/rectangle_collider.h" // Add: Include RectangleCollider.

class Mushroom {
 public:
 // -----
  bool GetIsDead() const;
  void TakeDamage();
 // -----
 private:
  // -----
  // --- Add: Store the collider so that it's possible to check for collisions and death state. ---
  const ng::RectangleCollider* collider_ = nullptr;
  bool is_dead_ = false;
  // --- End ---
};
```
{% endcode %}

{% code title="game/mushroom.cc" %}
```cpp
// -----
// --- Add: Includes. ---
#include <vector>
#include "engine/collider.h"
#include "engine/rectangle_collider.h"
#include "player.h"
// --- End ---

Mushroom::Mushroom(ng::App* app, const ng::Tilemap* tilemap) {
  // -----
  // --- Add: Add collider as child of the mushroom.  ---
  auto& collider = MakeChild<ng::RectangleCollider>(sf::Vector2f(32, 32));
  collider.SetLocalPosition({0, 16});
  collider_ = &collider;
  // --- End ---
}

// -----
// --- Add: Death getter and TakeDamage. ---
bool Mushroom::GetIsDead() const {
  return is_dead_;
}

void Mushroom::TakeDamage() {
  if (is_dead_) {
    return;
  }

  is_dead_ = true;
  Destroy();
}
// --- End ---
// -----

void Mushroom::Update() {
  // --- Add: Stop if mushroom is dead. ---
  if (is_dead_) {
    return;
  }
  // --- End ---

  // -----
  sf::Vector2f old_pos = collider_->GetGlobalTransform().getPosition(); // Change: Use collider position.
  sf::Vector2f new_pos = old_pos + velocity_;

  sf::Vector2f col_half_size = collider_->GetSize() / 2.F; // Change: Use collider size instead of hard-coded size.
  sf::Vector2f tilemap_size = sf::Vector2f(tilemap_->GetTileSize());
  // -----
  SetLocalPosition(new_pos - collider_->GetLocalTransform().getPosition()); // Change: Remove collider's local position offset.
  // -----
  if (!tilemap_->IsWithinWorldBounds(top_left) ||
      !tilemap_->IsWithinWorldBounds(bottom_left)) {
    TakeDamage(); // Add: TakeDamage when outside of bounds.
    return;
  }
  // -----
  if (!tilemap_->IsWithinWorldBounds(top_right) ||
      !tilemap_->IsWithinWorldBounds(bottom_right)) {
    TakeDamage(); // Add: TakeDamage when outside of bounds.
    return;
  }
  // -----
  if (!tilemap_->IsWithinWorldBounds(top_left) ||
      !tilemap_->IsWithinWorldBounds(top_right)) {
    TakeDamage(); // Add: TakeDamage when outside of bounds.
    return;
  }
  // -----
  if (!tilemap_->IsWithinWorldBounds(bottom_left) ||
      !tilemap_->IsWithinWorldBounds(bottom_right)) {
    TakeDamage(); // Add: TakeDamage when outside of bounds.
    return;
  }
  // -----
  // --- Add: Check collisions with the Player and other Mushrooms. ---
  std::vector<const ng::Collider*> others =
      GetScene()->GetPhysics().Overlap(*collider_);
  for (const auto* other : others) {
    if (other->GetParent()->GetName() == "Player") {
      auto* player = dynamic_cast<Player*>(other->GetParent());
      if (player->GetVelocity().y <= 0) {
        player->TakeDamage();
      }
    } else if (other->GetParent()->GetName() == "Mushroom") {
      auto* mushroom = dynamic_cast<Mushroom*>(other->GetParent());
      if (!mushroom->GetIsDead()) {
        mushroom->direction_.x = -mushroom->direction_.x;
        direction_.x = -direction_.x;
      }
    }
  }
  // --- End ---
}
```
{% endcode %}
