# Tilemap Node

## Tilemap Migration

As an in-world entity, the `Tilemap` must become a `Node` to inherit its Draw method. The tilemap's transform will now handle world translation, and the map will be drawn at the tilemap's transform.&#x20;

<pre class="language-cpp" data-title="engine/tilemap.h"><code class="lang-cpp">// --- Add: Includes. ---
#include "app.h" 
<strong>#include "node.h"
</strong><strong>// --- End ---
</strong>
namespace ng {

/// @brief Represents a grid-based map composed of tiles from a Tileset.
class Tilemap : public Node { // Add: Inherit from Node.
 Tilemap(App* app, sf::Vector2u size, Tileset tileset); // Change: Add App parameter.
 
 // -----
 protected:
  /// @brief Renders the tilemap.
  /// @param target The SFML RenderTarget to draw to.
  void Draw(sf::RenderTarget&#x26; target) override; // Change: Instead of having a public Draw, override the node's method.
 // -----
};

}  // namespace ng
</code></pre>

{% code title="engine/tilemap.cc" %}
```cpp
// --- Add: Includes. ---
#include "app.h"
#include "node.h"
// --- End ---

namespace ng {

Tilemap::Tilemap(App* app, sf::Vector2u size, Tileset tileset) // Change: Add App parameter.
    : Node(app),  // Add: Every node needs to call the base constructor with the app pointer.
      size_(size),
      // -----

// -----

bool Tilemap::IsWithinWorldBounds(sf::Vector2f world_position) const {
  sf::Vector2f tilemap_relative_position =
      (world_position - GetGlobalTransform().getPosition()); // Add: factor in the node's transform position.

  if (tilemap_relative_position.x < 0 || tilemap_relative_position.y < 0) {
    return false;
  }

  sf::Vector2u tile_position(tilemap_relative_position.componentWiseDiv(
      sf::Vector2f(tileset_.GetTileSize())));

  return IsWithinBounds(tile_position);
}

void Tilemap::Draw(sf::RenderTarget& target) {
  sf::RenderStates state;
  state.transform = GetGlobalTransform().getTransform(); // Add: use the node's transform.
  state.texture = tileset_.GetTexture();
  target.draw(vertices_, state);
}

}  // namespace ng
```
{% endcode %}
