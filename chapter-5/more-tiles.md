# More Tiles

## World Tiles

To build a detailed and complete game world represented by the tilemap, we need to incorporate more tile options, specifically multiple variations for the ground and outer walls. The invisible barrier tile will also be crucial for functionally blocking off certain areas to the player while remaining visually hidden.

{% code title="game/tile_id.h" %}
```cpp
#pragma once

#include <cstdint>

enum class TileID : uint64_t {  // NOLINT
  kVoid = 0,
  kInvisibleBarrier,
  kDirtTopLeft,
  kDirtTopCenter,
  kDirtTopRight,
  kDirtMiddleLeft,
  kDirtMiddleCenter,
  kDirtMiddleRight,
  kDirtBottomLeft,
  kDirtBottomCenter,
  kDirtBottomRight,
  kStoneHorizontalLeft,
  kStoneHorizontalCenter,
  kStoneHorizontalRight,
  kStoneVerticalTop,
  kStoneVerticalMiddle,
  kStoneVerticalBottom,
  kPlasticBlock,
};
```
{% endcode %}

## Breakable Tiles

To ensure proper interaction with the updated tiles, we first need to adapt the collision handling functions for the `Player` and the `Mushroom`.&#x20;

Then, we will introduce a new gameplay feature: **breakable tiles**. The orange "plastic" tiles will be destroyed when the Player jumps and hits them from below.

{% code title="game/player.h" %}
```cpp
// -----
class Player : public ng::Node {
 public:
  Player(ng::App* app, ng::Tilemap* tilemap, GameManager* game_manager,
         ScoreManager* score_manager); // Change: Take Tilemap as mutable pointer.
         
  // -----
  
 private:
  // -----
  ng::Tilemap* tilemap_ = nullptr; // Change: Tilemap as mutable pointer.
  // -----
  sf::Sound plastic_block_sound_; // Add: Plastic block breaking sound.
};
```
{% endcode %}

{% code title="game/player.cc" %}
```cpp
// -----
namespace {

// --- Change: Adapt collision mask. ---
bool DoesCollide(sf::Vector2f position, const ng::Tilemap& tilemap) {
  TileID id = tilemap.GetWorldTile(position).GetID();
  return id == TileID::kInvisibleBarrier ||
         (id >= TileID::kDirtTopLeft && id <= TileID::kDirtBottomRight) ||
         (id >= TileID::kStoneHorizontalLeft &&
          id <= TileID::kStoneVerticalBottom) ||
         id == TileID::kPlasticBlock;
}
// --- End ---

}  // namespace


Player::Player(ng::App* app, ng::Tilemap* tilemap, GameManager* game_manager,
               ScoreManager* score_manager) // Change: Take Tilemap as mutable pointer.
    : ng::Node(app),
      // -----
      // --- Add: Plastic block breaking sound. ---
      plastic_block_sound_(
          GetApp()->GetResourceManager().LoadSoundBuffer("Hit_1.wav"))
      // --- End ---

void Player::Update() {
  // -----
  
  if (velocity_.y < 0 &&
      (DoesCollide(top_left, *tilemap_) || DoesCollide(top_right, *tilemap_))) {
    // --- Add: Break blocks above player. ---
    {
      if (tilemap_->GetWorldTile(top_left).GetID() == TileID::kPlasticBlock) {
        tilemap_->SetWorldTile(top_left, TileID::kVoid);
        plastic_block_sound_.play();
      }

      if (tilemap_->GetWorldTile(top_right).GetID() == TileID::kPlasticBlock) {
        tilemap_->SetWorldTile(top_right, TileID::kVoid);
        plastic_block_sound_.play();
      }
    }
    // --- End ---

    new_pos.y = std::ceil(top_left.y / tilemap_size.y) * tilemap_size.y +
                col_half_size.y;
    velocity_.y = 0;
  }
  
  // -----
}
// -----
```
{% endcode %}

{% code title="engine/mushroom.cc" %}
```cpp
// -----
namespace {

// --- Change: Adapt collision mask. ---
bool DoesCollide(sf::Vector2f position, const ng::Tilemap& tilemap) {
  TileID id = tilemap.GetWorldTile(position).GetID();
  return id == TileID::kInvisibleBarrier ||
         (id >= TileID::kDirtTopLeft && id <= TileID::kDirtBottomRight) ||
         (id >= TileID::kStoneHorizontalLeft &&
          id <= TileID::kStoneVerticalBottom) ||
         id == TileID::kPlasticBlock;
}
// --- End ---

}  // namespace
// -----
```
{% endcode %}
