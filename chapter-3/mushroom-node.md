# Mushroom Node

## Mushroom Migration

As an in-world entity, the `Mushroom` must become a `Node` to inherit its Update and Draw methods. Instead of directly manipulating the sprite's position, the mushroom's transform will now handle world translation, and the sprite will be drawn at the mushroom's transform. Since the mushroom resides at the scene tree's root, its local transform directly mirrors its global transform.

{% code title="game/mushroom.h" %}
```cpp
// -----
#include "engine/node.h" // Add: Include.

namespace game {

class Mushroom : public ng::Node { // Add: Inherit from Node.
 public:
  Mushroom(ng::App* app, const ng::Tilemap* tilemap);

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

{% code title="game/mushroom.cc" %}
```cpp
// -----
#include "engine/node.h" // Add: Include.
#include <cassert> // Remove.

namespace game {

Mushroom::Mushroom(ng::App* app, const ng::Tilemap* tilemap)
    : ng::Node(app),  // Add: Every node needs to call the base constructor with the app pointer.
      app_(app), // Remove.
      tilemap_(tilemap),
      sprite_(GetApp()->GetResourceManager().LoadTexture(
          "Mushroom/Run (32x32).png")) // Change: Use GetApp().
      // -----
{
  assert(app); // Remove.
  SetName("Mushroom"); // Add: Name the node.
  // -----
  sprite_.setPosition({64, 64}); // Remove.
}

void Mushroom::Update() {
  // -----
  sf::Vector2f old_pos = GetLocalTransform().getPosition(); // Change: use the node's transform position.
  // -----
  sprite_.setPosition(new_pos); // Remove.
  SetLocalPosition(new_pos); // Add: Set the node position.
}

void Mushroom::Draw(sf::RenderTarget& target) {
  sprite_.setScale(sf::Vector2f{-direction_.x * 2, 2.F});
  target.draw(sprite_, GetGlobalTransform().getTransform()); // Change: use the node's transform.
}

}  // namespace game
```
{% endcode %}
