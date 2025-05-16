# Player Collisions

## Damaging and Collecting

The `Player` node now incorporates a child `RectangleCollider` node, intentionally sized slightly smaller than its sprite. Collision detection with the tilemap now relies on this collider's dimensions and position, resulting in more precise hitboxes compared to the sprite's visual bounds. This collider is also used to detect interactions with other mushrooms and bananas. Upon colliding with a falling mushroom, the mushroom is destroyed, and the player receives a small vertical bounce. When a banana is hit, it's considered collected, and a collection sound is triggered. This sound is managed by the player Node to ensure its complete playback, as the banana Node is immediately destroyed upon collection, which would prematurely terminate the sound if played from the banana itself. The player's longer lifespan guarantees the sound finishes.

```cpp
// -----
#include "engine/rectangle_collider.h" // Add: Include RectangleCollider.

class Player : public ng::Node {
 public:
 // -----
 // --- Add: Velocity getter and TakeDamage. ---
  sf::Vector2f GetVelocity() const;
  void TakeDamage();
 // --- End ---
 // -----
 private:
  // -----
  // --- Add: Store the collider so that it's possible to check for collisions and death state and banana sound. ---
  const ng::RectangleCollider* collider_ = nullptr;
  bool is_dead_ = false;
  sf::Sound banana_sound_;
  // --- End ---
};
```

```cpp
// -----
// --- Add: Includes. ---
#include <vector>
#include "engine/collider.h"
#include "engine/rectangle_collider.h"
#include "mushroom.h"
#include "banana.h"
// --- End ---

Player::Player(ng::App* app, const ng::Tilemap* tilemap) 
 : ng::Node(app),
 // -----
 // --- Add: Banana pickup sound. ---
 banana_sound_(GetApp()->GetResourceManager().LoadSoundBuffer(
          "Banana/Collectibles_2.wav")) 
 // --- End ---
 {
  // -----
  // --- Add: Add collider as child of the player.  ---
  auto& collider = MakeChild<ng::RectangleCollider>(sf::Vector2f(32, 48));
  collider.SetLocalPosition({0, 8});
  collider_ = &collider;
  // --- End ---
}

// -----
// --- Add: Velocity getter and TakeDamage. ---
sf::Vector2f Player::GetVelocity() const {
  return player_velocity_;
}

void Player::TakeDamage() {
  if (is_dead_) {
    return;
  }

  is_dead_ = true;
  Destroy();
}
// --- End ---
// -----

void Player::Update() {
  // --- Add: Stop if player is dead. ---
  if (is_dead_) {
    return;
  }
  // --- End ---

  // -----
  sf::Vector2f collision_offset(0, 8);
  sf::Vector2f old_pos = collider_->GetGlobalTransform().getPosition(); // Change: Use collider position.
  sf::Vector2f new_pos = old_pos + player_velocity_;

  sf::Vector2f col_half_size = collider_->GetSize() / 2.F; // Change: Use collider size instead of hard-coded size.
  sf::Vector2f tilemap_size = sf::Vector2f(tilemap_->GetTileSize());
  // -----
  SetLocalPosition(new_pos - collider_->GetLocalTransform().getPosition()); // Change: Subtract collider's local position offset.
  // -----
  if (!tilemap_->IsWithinWorldBounds(top_left) ||
      !tilemap_->IsWithinWorldBounds(middle_left) ||
      !tilemap_->IsWithinWorldBounds(bottom_left)) {
    TakeDamage(); // Add: TakeDamage when outside of bounds.
    return;
  }
  // -----
  if (!tilemap_->IsWithinWorldBounds(top_right) ||
      !tilemap_->IsWithinWorldBounds(middle_right) ||
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
  // --- Add: Check collisions with the Mushroom and Banana. ---
  std::vector<const ng::Collider*> others =
      GetScene()->GetPhysics().Overlap(*collider_);
  for (const auto* other : others) {
    if (other->GetParent()->GetName() == "Mushroom") {
      if (player_velocity_.y > 0) {
        auto* mushroom = dynamic_cast<Mushroom*>(other->GetParent());
        if (!mushroom->GetIsDead()) {
          mushroom->TakeDamage();
          player_velocity_.y = -10;
        }
      }
    } else if (other->GetParent()->GetName() == "Banana") {
      auto* banana = dynamic_cast<Banana*>(other->GetParent());
      if (!banana->GetIsCollected()) {
        banana->Collect();
        banana_sound_.play();
      }
    }
  }
}
```
