# Mushroom Enemy

## Mushroom 🍄

The `Mushroom` movement logic closely mirrors the `Player` in some aspects. It automatically travels in a single direction at a constant speed. Upon colliding with a wall, the mushroom reverses its direction and continues moving indefinitely. If the mushroom attempts to access a tile outside the map's boundaries, it signifies that it has fallen off, and its movement will stop. Later on, the destruction of mushroom objects will be implemented whenever it goes out of bounds.

{% code title="game/mushroom.h" %}
```cpp
#pragma once

#include <SFML/Graphics/RenderTarget.hpp>
#include <SFML/Graphics/Sprite.hpp>
#include <SFML/System/Vector2.hpp>

#include "engine/app.h"
#include "engine/tilemap.h"

namespace game {

class Mushroom {
 public:
  Mushroom(ng::App* app, const ng::Tilemap* tilemap);

  void Update();
  void Draw(sf::RenderTarget& target);

 private:
  ng::App* app_ = nullptr;
  sf::Vector2f direction_{-1, 0};
  sf::Vector2f velocity_;
  const ng::Tilemap* tilemap_ = nullptr;
  sf::Sprite sprite_;
  bool is_on_ground_ = false;
};

}  // namespace game
```
{% endcode %}

{% code title="game/mushroom.cc" %}
```cpp
#include "mushroom.h"

#include <SFML/Graphics/RenderTarget.hpp>
#include <SFML/Graphics/Sprite.hpp>
#include <SFML/System/Vector2.hpp>
#include <cmath>
#include <cassert>

#include "engine/app.h"
#include "engine/tilemap.h"
#include "tile_id.h"

namespace game {

namespace {

bool DoesCollide(sf::Vector2f position, const ng::Tilemap& tilemap) {
  TileID id = tilemap.GetWorldTile(position).GetID();
  return id == TileID::kDirt;
}

}  // namespace

Mushroom::Mushroom(ng::App* app, const ng::Tilemap* tilemap)
    : app_(app),
      tilemap_(tilemap),
      sprite_(
          app_->GetResourceManager().LoadTexture("Mushroom/Run (32x32).png")) {
  assert(app);
          
  sprite_.setScale({2, 2});
  sprite_.setOrigin({16, 16});
  sprite_.setTextureRect(sf::IntRect({0, 0}, {32, 32}));

  // Set the initial position. This is temporary.
  sprite_.setPosition({64, 64});
}

void Mushroom::Update() {  // NOLINT
  static constexpr float kMovementSpeed = 2.F;

  // Calculate horizontal velocity.
  velocity_.x = direction_.x * kMovementSpeed;
  // Apply gravity.
  velocity_.y += 1;

  // Get the current position.
  sf::Vector2f old_pos = sprite_.getPosition();
  // Calculate the new position.
  // This only works under the assumption that the rotation is always 0.
  sf::Vector2f new_pos = old_pos + velocity_;

  // The sprite is 32x32. The scale is 2, so the collision size is 64x64.
  sf::Vector2f col_half_size = {32, 32};
  sf::Vector2f tilemap_size = sf::Vector2f(tilemap_->GetTileSize());

  static constexpr float kEps = 0.001F;
  // Calculate collision points for the left side of the mushroom.
  sf::Vector2f top_left = {new_pos.x - (col_half_size.x - kEps),
                           old_pos.y - (col_half_size.y - kEps)};
  sf::Vector2f bottom_left = {new_pos.x - (col_half_size.x - kEps),
                              old_pos.y + (col_half_size.y - kEps)};

  // Check if the collision points are within the tilemap bounds.
  if (!tilemap_->IsWithinWorldBounds(top_left) ||
      !tilemap_->IsWithinWorldBounds(bottom_left)) {
    return;
  }

  // Check for collisions on the left side.
  if (velocity_.x < 0 && (DoesCollide(top_left, *tilemap_) ||
                          DoesCollide(bottom_left, *tilemap_))) {
    // If collided, adjust the new position and reverse the direction.
    new_pos.x = std::ceil(top_left.x / tilemap_size.x) * tilemap_size.x +
                col_half_size.x;
    velocity_.x = 0;
    direction_.x = 1;
  }

  // Calculate collision points for the right side of the mushroom.
  sf::Vector2f top_right = {new_pos.x + (col_half_size.x - kEps),
                            old_pos.y - (col_half_size.y - kEps)};
  sf::Vector2f bottom_right = {new_pos.x + (col_half_size.x - kEps),
                               old_pos.y + (col_half_size.y - kEps)};

  // Check if the collision points are within the tilemap bounds.
  if (!tilemap_->IsWithinWorldBounds(top_right) ||
      !tilemap_->IsWithinWorldBounds(bottom_right)) {
    return;
  }

  // Check for collisions on the right side.
  if (velocity_.x > 0 && (DoesCollide(top_right, *tilemap_) ||
                          DoesCollide(bottom_right, *tilemap_))) {
    // If collided, adjust the new position and reverse the direction.
    new_pos.x = std::floor(top_right.x / tilemap_size.x) * tilemap_size.x -
                col_half_size.x;
    velocity_.x = 0;
    direction_.x = -1;
  }

  // Calculate collision points for the top of the mushroom.
  top_left = {new_pos.x - (col_half_size.x - kEps),
              new_pos.y - (col_half_size.y - kEps)};
  top_right = {new_pos.x + (col_half_size.x - kEps),
               new_pos.y - (col_half_size.y - kEps)};

  // Check if the collision points are within the tilemap bounds.
  if (!tilemap_->IsWithinWorldBounds(top_left) ||
      !tilemap_->IsWithinWorldBounds(top_right)) {
    return;
  }

  // Check for collisions on the top.
  if (velocity_.y < 0 &&
      (DoesCollide(top_left, *tilemap_) || DoesCollide(top_right, *tilemap_))) {
    // If collided, adjust the new position and reset the vertical velocity.
    new_pos.y = std::ceil(top_left.y / tilemap_size.y) * tilemap_size.y +
                col_half_size.y;
    velocity_.y = 0;
  }

  // Calculate collision points for the bottom of the mushroom.
  bottom_left = {new_pos.x - (col_half_size.x - kEps),
                 new_pos.y + (col_half_size.y - kEps)};
  bottom_right = {new_pos.x + (col_half_size.x - kEps),
                  new_pos.y + (col_half_size.y - kEps)};

  // Check if the collision points are within the tilemap bounds.
  if (!tilemap_->IsWithinWorldBounds(bottom_left) ||
      !tilemap_->IsWithinWorldBounds(bottom_right)) {
    return;
  }

  // Check for collisions on the bottom.
  if (velocity_.y > 0 && (DoesCollide(bottom_left, *tilemap_) ||
                          DoesCollide(bottom_right, *tilemap_))) {
    // If collided, adjust the new position, reset the vertical velocity, and set the on-ground flag.
    new_pos.y = std::floor(bottom_left.y / tilemap_size.y) * tilemap_size.y -
                col_half_size.y;
    velocity_.y = 0;
    is_on_ground_ = true;
  } else {
    is_on_ground_ = false;
  }

  sprite_.setPosition(new_pos);  // Set the new position of the sprite.
}

void Mushroom::Draw(sf::RenderTarget& target) {
  // Flip the sprite horizontally based on the movement direction.
  sprite_.setScale(sf::Vector2f{-direction_.x * 2, 2.F});
  target.draw(sprite_);
}

}  // namespace game
```
{% endcode %}
