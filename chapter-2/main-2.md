# Main 2

## Main Strikes Back

The complete contents of `main.cc` are provided here due to the extensive modifications made. This ensures that no changes are overlooked.

{% code title="game/main.cc" %}
```cpp
#include <SFML/Graphics/RenderWindow.hpp>
#include <SFML/System/Vector2.hpp>
#include <SFML/Window/Event.hpp>
#include <cstdint>
#include <cstdlib>
#include <optional>

#include "background.h"
#include "engine/app.h"
#include "engine/input.h"
#include "engine/resource_manager.h"
#include "engine/tile.h"
#include "engine/tilemap.h"
#include "engine/tileset.h"
#include "mushroom.h"
#include "player.h"
#include "tile_id.h"

int main() {
  static constexpr sf::Vector2u kWindowSize = {832U, 640U};
  ng::App app(kWindowSize, "Platformer", 60);
  sf::RenderWindow& window = app.GetWindow();
  ng::Input& input = app.GetInput();
  ng::ResourceManager& resource_manager = app.GetResourceManager();

  ng::Tileset tileset({32, 32},
                      &resource_manager.LoadTexture("Terrain (16x16).png"));
  tileset.AddTile(ng::Tile(TileID::kVoid));
  tileset.AddTile(
      ng::Tile(TileID::kDirt, sf::IntRect({7 * 16, 0 * 16}, {16, 16})));

  ng::Tilemap tilemap(sf::Vector2u(64, 32), tileset);
  for (uint32_t i = 0; i < 8; ++i) {
    tilemap.SetTile({0, i}, TileID::kDirt);
  }
  for (uint32_t i = 0; i < 20; ++i) {
    tilemap.SetTile({i, 8}, TileID::kDirt);
  }
  for (uint32_t i = 0; i < 8; ++i) {
    tilemap.SetTile({20 - 1, i}, TileID::kDirt);
  }

  game::Background background(
      &app, sf::Vector2u(32, 32).componentWiseMul(tilemap.GetSize()));
  game::Player player(&app, &tilemap);
  game::Mushroom mushroom(&app, &tilemap);

  while (window.isOpen()) {
    input.Advance();
    while (const std::optional event = window.pollEvent()) {
      if (event->is<sf::Event::Closed>()) {
        window.close();
      }

      input.Handle(*event);
    }

    // Update everything.
    background.Update();
    mushroom.Update();
    player.Update();

    window.clear();

    // Draw everything.
    background.Draw(window);
    tilemap.Draw(window);
    mushroom.Draw(window);
    player.Draw(window);

    window.display();
  }

  return EXIT_SUCCESS;
}

```
{% endcode %}
