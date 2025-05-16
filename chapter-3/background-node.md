# Background Node

## Background Migration

As an in-world entity, the `Background` must become a `Node` to inherit its Update and Draw methods. The background's transform will now handle world translation, and the texture will be drawn at the background's transform.

{% code title="game/background.h" %}
```cpp
// -----
#include "engine/node.h" // Add: Include.

namespace game {

class Background : public ng::Node { // Add: Inherit from Node.
 public:
  Background(ng::App* app, sf::Vector2u size);

 // --- Change: Instead of having a public Update and Draw, override the node's methods. ---
 protected:
  void Update() override;
  void Draw(sf::RenderTarget& target) override;
 // --- End ---
 
 // -----
 private:
  ng::App* app_ = nullptr; // Remove.
};

} }  // namespace game
```
{% endcode %}

{% code title="game/background.cc" %}
```cpp
// -----
#include "engine/node.h" // Add: Include.
#include <cassert> // Remove.

namespace game {

Background::Background(ng::App* app, sf::Vector2u size)
    : ng::Node(app),  // Add: Every node needs to call the base constructor with the app pointer.
      app_(app), // Remove.
      size_(size),
      texture_(&GetApp()->GetResourceManager().LoadTexture("Gray.png")), // Change: Use GetApp().
      // -----
{
    assert(app); // Remove.
    // -----
}

// -----

void Background::Draw(sf::RenderTarget& target) {
  sf::RenderStates state;
  state.texture = texture_;
  state.transform = GetGlobalTransform().getTransform(); // Add: use the node's transform.
  target.draw(image_vertices_, state);
}

}  // namespace game
```
{% endcode %}
