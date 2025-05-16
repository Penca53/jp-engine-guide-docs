# Platformer Player

## Player Movement

The Player character will be controlled via standard platformer keybindings: the 'A' key moves the player left, the 'D' key moves them right, and the Spacebar initiates a jump, but only when the player is currently grounded. Additionally, gravity will constantly affect the player's movement.

{% code title="game/player.h" %}
```cpp
#pragma once

#include <SFML/Audio/Sound.hpp> // Add: Include Sound.
#include <SFML/Graphics/RenderTarget.hpp>
#include <SFML/Graphics/Sprite.hpp>

#include "engine/app.h"
#include "engine/tilemap.h" // Add: Include Tilemap.

namespace game {

class Player {
 public:
  Player(ng::App* app, const ng::Tilemap* tilemap); // Change: Add tilemap parameter and remove explicit keyword.

  void Update();
  void Draw(sf::RenderTarget& target);

 private:
  ng::App* app_ = nullptr;
  const ng::Tilemap* tilemap_ = nullptr; // Add: tilemap pointer.
  sf::Sprite sprite_;
  sf::Vector2f player_velocity_; // Add: current player velocity.
  bool is_on_ground_ = false; // Add: is the player on the ground?
  sf::Sound jump_sound_; // Add: sound played when jumping.
};

}  // namespace game
```
{% endcode %}

{% code title="game/player.cc" %}
```cpp
// --- Add: Includes for this and the following change. ---
#include <SFML/System/Vector2.hpp>
#include <cmath>

#include "engine/tilemap.h"
#include "tile_id.h"
// --- End ---

namespace game {

Player::Player(ng::App* app, const ng::Tilemap* tilemap) // Change: Add tilemap parameter and jump_sound.
    : app_(app),
      tilemap_(tilemap), // Add: tilemap pointer.
      sprite_(
          app_->GetResourceManager().LoadTexture("Player/Idle (32x32).png")),
      // --- Add: Load the jump sound. ---
      jump_sound_(
          app_->GetResourceManager().LoadSoundBuffer("Player/Jump_2.wav")) 
      // --- End ---
{

  assert(app);

  sprite_.setTextureRect(sf::IntRect({0, 0}, {32, 32}));
  sprite_.setOrigin({16, 16});
  sprite_.setScale({2, 2});
  sprite_.setPosition({64, 64}); // Add: Set the initial position. This is temporary.
}

void Player::Update() {  // NOLINT
  // --- Add: Replace old Update code with the following. ---
  sf::Vector2f direction;
  if (app_->GetInput().GetKey(sf::Keyboard::Scancode::A)) {
    direction.x -= 1;
    // Flip the sprite horizontally.
    sprite_.setScale(sf::Vector2f{-2.F, 2.F});
  }
  if (app_->GetInput().GetKey(sf::Keyboard::Scancode::D)) {
    direction.x += 1;
    // Reset the sprite's horizontal flip.
    sprite_.setScale(sf::Vector2f{2.F, 2.F});
  }

  static constexpr float kMovementSpeed = 4.F;
  // Calculate horizontal velocity.
  player_velocity_.x = direction.x * kMovementSpeed;
  // Apply gravity.
  player_velocity_.y += 1;

  // Jump if on the ground and space is pressed.
  if (is_on_ground_ &&
      app_->GetInput().GetKeyDown(sf::Keyboard::Scancode::Space)) {
    // Apply jump force.
    player_velocity_.y -= 15;
    jump_sound_.play();
  }
  
  sf::Vector2f collision_offset(0, 8);
  // Get the current position.
  sf::Vector2f old_pos = sprite_.getPosition() + collision_offset;
  // Calculate the new position.
  // This only works under the assumption that the rotation is always 0.
  sf::Vector2f new_pos = old_pos + player_velocity_;

  sprite_.setPosition(new_pos - collision_offset); // Set the new position of the sprite.
  // --- End ---
}

// -----

}  // namespace game
```
{% endcode %}

The `ResourceManager` still doesn't implement sound loading. The next page in the guide addresses that aspect.

## Player Collision with Tilemap

The `Tilemap` represents the game world, requiring the `Player` to interact with it and be stopped by solid tiles like the ground and walls. The player's movement system computes the next position and validates it by checking if all four corners of the player's collision box are clear of solid tiles. Collision checks are performed per side, and if a solid tile is hit, the player's new position is adjusted (clamped) to prevent passing through. Attempting to move outside the tilemap boundaries results in the player's movement being frozen, indicating a fall off the map (player death and losing will be added later). Once any necessary collision resolution is complete, the player's sprite is moved to the validated position.

Note that before we said that we only check the 4 corners of the player's box. That would lead to a curious **bug**, because the player's collision square (32x48) is larger than the tile size (32x32). If we only check the 4 corners of the player, the player could technically walk through a 1-tile-tall wall because the collision steps are done every 24 pixels of height, and the checks wouldn't catch it. The solution is to add 2 new checks in the middle of the player, one on the left and one on the right, making them **6 checks** in total.

```cpp
#include "player.h"

namespace game {

// --- Add: Collision detection with tilemap. ---
namespace {

// Helper function to check for collisions with tiles.
bool DoesCollide(sf::Vector2f position, const ng::Tilemap& tilemap) {
  TileID id = tilemap.GetWorldTile(position).GetID();
  return id == TileID::kDirt;
}

}  // namespace
// --- End ---

void Player::Update() {  // NOLINT
  // -----
  
  sf::Vector2f collision_offset(0, 8);
  // Get the current position.
  sf::Vector2f old_pos = sprite_.getPosition() + collision_offset;
  // Calculate the new position.
  // This only works under the assumption that the rotation is always 0.
  sf::Vector2f new_pos = old_pos + player_velocity_;

  // --- Add: Collision detection and resolution with tilemap. ---
  // Shrink the collision size a bit compared to the actual sprite.
  // This is done because the sprite image has some "free pixels" around 
  // that make the player look like it's not colliding with the map.
  sf::Vector2f col_half_size = {16, 24};
  sf::Vector2f tilemap_size = sf::Vector2f(tilemap_->GetTileSize());

  static constexpr float kEps = 0.001F;
  // Calculate collision points for the left side of the player.
  sf::Vector2f top_left = {new_pos.x - (col_half_size.x - kEps),
                           old_pos.y - (col_half_size.y - kEps)};
  sf::Vector2f middle_left = {new_pos.x + (col_half_size.x - kEps),
                              old_pos.y + (0 - kEps)};
  sf::Vector2f bottom_left = {new_pos.x - (col_half_size.x - kEps),
                              old_pos.y + (col_half_size.y - kEps)};

  // Check if the collision points are within the tilemap bounds.
  if (!tilemap_->IsWithinWorldBounds(top_left) ||
      !tilemap_->IsWithinWorldBounds(middle_left) ||
      !tilemap_->IsWithinWorldBounds(bottom_left)) {
    return;
  }

  // Check for collisions on the left side.
  if (player_velocity_.x < 0 && (DoesCollide(top_left, *tilemap_) ||
                                 DoesCollide(middle_left, *tilemap_) ||
                                 DoesCollide(bottom_left, *tilemap_))) {
    // If collided, adjust the new position and reset the horizontal velocity.
    new_pos.x = std::ceil(top_left.x / tilemap_size.x) * tilemap_size.x +
                col_half_size.x;
    player_velocity_.x = 0;
  }

  // Calculate collision points for the right side of the player.
  sf::Vector2f top_right = {new_pos.x + (col_half_size.x - kEps),
                            old_pos.y - (col_half_size.y - kEps)};
  sf::Vector2f middle_right = {new_pos.x + (col_half_size.x - kEps),
                               old_pos.y + (0 - kEps)};
  sf::Vector2f bottom_right = {new_pos.x + (col_half_size.x - kEps),
                               old_pos.y + (col_half_size.y - kEps)};

  // Check if the collision points are within the tilemap bounds.
  if (!tilemap_->IsWithinWorldBounds(top_right) ||
      !tilemap_->IsWithinWorldBounds(middle_right) ||
      !tilemap_->IsWithinWorldBounds(bottom_right)) {
    return;
  }

  // Check for collisions on the right side.
  if (player_velocity_.x > 0 && (DoesCollide(top_right, *tilemap_) ||
                                 DoesCollide(middle_right, *tilemap_) ||
                                 DoesCollide(bottom_right, *tilemap_))) {
    // If collided, adjust the new position and reset the horizontal velocity.
    new_pos.x = std::floor(top_right.x / tilemap_size.x) * tilemap_size.x -
                col_half_size.x;
    player_velocity_.x = 0;
  }

  // Calculate collision points for the top of the player.
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
  if (player_velocity_.y < 0 &&
      (DoesCollide(top_left, *tilemap_) || DoesCollide(top_right, *tilemap_))) {
    // If collided, adjust the new position and reset the vertical velocity.
    new_pos.y = std::ceil(top_left.y / tilemap_size.y) * tilemap_size.y +
                col_half_size.y;
    player_velocity_.y = 0;
  }

  // Calculate collision points for the bottom of the player.
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
  if (player_velocity_.y > 0 && (DoesCollide(bottom_left, *tilemap_) ||
                                 DoesCollide(bottom_right, *tilemap_))) {
    // If collided, adjust the new position, reset the vertical velocity, and set the on-ground flag.
    new_pos.y = std::floor(bottom_left.y / tilemap_size.y) * tilemap_size.y -
                col_half_size.y;
    player_velocity_.y = 0;
    is_on_ground_ = true;
  } else {
    is_on_ground_ = false;
  }
  // --- End ---

  sprite_.setPosition(new_pos - collision_offset); // Set the new position of the sprite.
}

// -----

}  // namespace game
```
