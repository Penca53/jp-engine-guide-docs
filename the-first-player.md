# The First Player

## Drawing the Player

### Update and Draw

The `Update` method is responsible for modifying an object's state based on its current condition and external factors, such as user input in this context.

The `Draw` method handles the rendering of an object's visual representation, if one exists, based on its internal state.

An object possessing only an `Update` method operates invisibly in the background, managing logic without visual output. Conversely, an object with only a `Draw` method is visible but remains static unless its state is altered by other entities (e.g., translation).

{% code title="game/player.h" %}
```cpp
#pragma once

#include <SFML/Graphics/RenderTarget.hpp>
#include <SFML/Graphics/Sprite.hpp>

#include "engine/app.h"

namespace game {

class Player {
 public:
  explicit Player(ng::App* app);

  void Update();
  void Draw(sf::RenderTarget& target);

 private:
  ng::App* app_ = nullptr;
  sf::Sprite sprite_;
};

}  // namespace game
```
{% endcode %}

The `Player` class holds a member pointer to the `App` object to access user input. It also contains a `sf::Sprite` member for rendering the player's 32x32 pixel texture.

{% code title="game/player.cc" %}
```cpp
#include "player.h"

#include <SFML/Graphics/RenderTarget.hpp>
#include <SFML/Window/Keyboard.hpp>
#include <cassert>

#include "engine/app.h"
#include "engine/input.h"

namespace game {

Player::Player(ng::App* app)
    : app_(app),
      // Load the player's texture.
      sprite_(
          app_->GetResourceManager().LoadTexture("Player/Idle (32x32).png")) {

  // Ensure the App pointer is not null.
  assert(app);

  // Set the texture rectangle. The texture is a texture sheet composed of multiple
  // frames, so we restrict to the first one.
  sprite_.setTextureRect(sf::IntRect({0, 0}, {32, 32}));
  // Set the sprite's origin to the center so that it's easier to handle the movement.
  sprite_.setOrigin({16, 16});
  // Scale the sprite up because it's a little bit too small.
  sprite_.setScale({2, 2});
}

void Player::Update() {
  // Empty for now...
}

void Player::Draw(sf::RenderTarget& target) {
  target.draw(sprite_);
}

}  // namespace game
```
{% endcode %}

### Pixels Per Unit (PPU)

An important concept for this game is the PPU, which defines the pixel density per engine unit. A lower PPU value results in a larger on-screen representation. While the default PPU in this case would be 32 pixels per unit, we will use a PPU of 16 pixels per unit (or, equivalently, scale sprites by 2, resulting in 32 pixels over 2 units) to achieve a more suitable size.

### Sprite Origin

SFML sprites default to an origin at the top-left corner. For ease of placement and collision detection, especially with circular shapes, we will set the origin of most, if not all, sprites to the center of their image. This is a common practice in game engine design.

## Moving the Player

The player's movement is controlled using the WASD keys, allowing for 2D top-down navigation. To visually indicate the direction of travel, the player's sprite is horizontally flipped when moving left or right.

Throughout this guide, please note that we will consistently use the following coordinate system: left corresponds to negative <mark style="color:red;">x</mark>, right to positive <mark style="color:red;">x</mark>, up to negative <mark style="color:green;">y</mark>, and down to positive <mark style="color:green;">y</mark>.

```cpp
void Player::Update() {
  // Movement direction vector.
  sf::Vector2f direction;
  const ng::Input& input = app_->GetInput();
  if (input.GetKey(sf::Keyboard::Scancode::A)) {
    direction.x -= 1;
    // Flip the sprite by inverting the scale.
    sprite_.setScale(sf::Vector2f{-2.F, 2.F});
  }
  if (input.GetKey(sf::Keyboard::Scancode::D)) {
    direction.x += 1;
    // Reset the sprite's horizontal flip.
    sprite_.setScale(sf::Vector2f{2.F, 2.F});
  }
  if (input.GetKey(sf::Keyboard::Scancode::W)) {
    direction.y -= 1;
  }
  if (input.GetKey(sf::Keyboard::Scancode::S)) {
    direction.y += 1;
  }

  // If the player is moving...
  if (direction.lengthSquared() > 0) { 
    // Get the current position.
    sf::Vector2f new_pos = sprite_.getPosition(); 
    static constexpr float kMovementSpeed = 4.F; 
    // Calculate the new position by normalizing the direction vector first.
    // This is required to avoid moving faster when the direction is diagonal.
    // A square with a side whose length is 1 unit, the diagonal's length is sqrt(2).
    new_pos += direction.normalized() * kMovementSpeed; 
    // Apply the new position.
    sprite_.setPosition(new_pos); 
  }
}
```

## Following the Player

The view of the game window is configured to follow the player's position, preventing the player from ever moving outside the boundaries of what is displayed.

```cpp
#include <SFML/Graphics/View.hpp> // Add: Include for managing the window view.

void Player::Draw(sf::RenderTarget& target) {
  // --- Add: Center the window's view onto the player. ---
  // Get the current window view.
  sf::View v = target.getView(); 
  // Center the view on the player.
  v.setCenter(sprite_.getPosition()); 
  // Set the new window view.
  target.setView(v); 
  // --- End ---
  target.draw(sprite_);
}
```
