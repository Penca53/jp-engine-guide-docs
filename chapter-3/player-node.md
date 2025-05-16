# Player Node

## Player Migration

As an in-world entity, the `Player` must become a `Node` to inherit its Update and Draw methods. Instead of directly manipulating the sprite's position, the player's transform will now handle world translation, and the sprite will be drawn at the player's transform. Since the player resides at the scene tree's root, its local transform directly mirrors its global transform.

{% code title="game/player.h" %}
```cpp
// -----
#include "engine/node.h" // Add: Include.

namespace game {

class Player : public ng::Node { // Add: Inherit from Node.
 public:
  Player(ng::App* app, const ng::Tilemap* tilemap);

 // --- Change: Instead of having a public Update and Draw, override the node's methods. ---
 protected:
  void Update() override;
  void Draw(sf::RenderTarget& target) override;
 // --- End ---
  
 // -----
 private:
   ng::App* app_ = nullptr; // Remove.
};

}  // namespace game
```
{% endcode %}

{% code title="game/player.cc" %}
```cpp
#include "engine/node.h" // Add: Include.
#include <cassert> // Remove.

namespace game {

Player::Player(ng::App* app, const ng::Tilemap* tilemap)
    : ng::Node(app), // Add: Every node needs to call the base constructor with the app pointer.
      app_(app), // Remove.
      tilemap_(tilemap),
       sprite_(GetApp()->GetResourceManager().LoadTexture(
          "Player/Idle (32x32).png")), // Change: Use GetApp().
      jump_sound_(
          GetApp()->GetResourceManager().LoadSoundBuffer("Player/Jump_2.wav")) // Change: Use GetApp().
      // -----
{
  assert(app); // Remove.
  SetName("Player"); // Add: Name the node.
  // -----
  sprite_.setPosition({64, 64}); // Remove.
}

void Player::Update() {
  // -----
  if (app_->GetInput().GetKey(sf::Keyboard::Scancode::A)) { // Change: Use GetApp().
    direction.x -= 1;
    // Flip the sprite horizontally.
    sprite_.setScale(sf::Vector2f{-2.F, 2.F});
  }
  if (app_->GetInput().GetKey(sf::Keyboard::Scancode::D)) { // Change: Use GetApp().
    direction.x += 1;
    // Reset the sprite's horizontal flip.
    sprite_.setScale(sf::Vector2f{2.F, 2.F});
  }
  // -----
  
  if (is_on_ground_ &&
      GetApp()->GetInput().GetKeyDown(sf::Keyboard::Scancode::Space)) { // Change: Use GetApp().
    // Apply jump force.
    player_velocity_.y -= 15;
    jump_sound_.play();
  }

  // -----
  sf::Vector2f old_pos = GetLocalTransform().getPosition() + collision_offset; // Change: use the node's transform position.
  // -----
  sprite_.setPosition(new_pos - collision_offset); // Remove.
  SetLocalPosition(new_pos - collision_offset); // Add: Set the node position.
}

void Player::Draw(sf::RenderTarget& target) {
  sf::View v = target.getView();
  v.setCenter(GetLocalTransform().getPosition()); // Change: use the node's transform position.
  target.setView(v);

  target.draw(sprite_, GetGlobalTransform().getTransform()); // Change: use the node's transform.
}

}  // namespace game
```
{% endcode %}
