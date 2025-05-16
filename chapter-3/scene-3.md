# Scene 3

## Nothing New, Everything Better

The `DefaultScene` is the one (and only) scene that will be used throughout the game. It builds the scene, node by node, and returns the built scene graph.

{% code title="game/default_scene.h" %}
```cpp
#pragma once

#include <memory>

#include "engine/app.h"
#include "engine/scene.h"

namespace game {

std::unique_ptr<ng::Scene> MakeDefaultScene(ng::App* app);

}  // namespace game
```
{% endcode %}

{% code title="game/default_scene.cc" %}
```cpp
#include "default_scene.h"

#include <cstdint>
#include <memory>
#include <utility>

#include "background.h"
#include "engine/app.h"
#include "engine/node.h"
#include "engine/scene.h"
#include "engine/tile.h"
#include "engine/tilemap.h"
#include "engine/tileset.h"
#include "mushroom.h"
#include "player.h"
#include "tile_id.h"

namespace game {

std::unique_ptr<ng::Scene> MakeDefaultScene(ng::App* app) {
  auto scene = std::make_unique<ng::Scene>(app);
  scene->SetName("Scene");

  ng::Tileset tileset(
      {32, 32}, &app->GetResourceManager().LoadTexture("Terrain (16x16).png"));
  tileset.AddTile(ng::Tile(TileID::kVoid));
  tileset.AddTile(
      ng::Tile(TileID::kDirt, sf::IntRect({7 * 16, 0 * 16}, {16, 16})));

  auto tmp_tilemap =
      std::make_unique<ng::Tilemap>(app, sf::Vector2u(64, 32), tileset);
  scene->MakeChild<Background>(
      tmp_tilemap->GetSize().componentWiseMul(tmp_tilemap->GetTileSize()));

  auto& tilemap = *tmp_tilemap;
  scene->AddChild(std::move(tmp_tilemap));

  for (uint32_t i = 0; i < 8; ++i) {
    tilemap.SetTile({0, i}, TileID::kDirt);
  }
  for (uint32_t i = 0; i < 20; ++i) {
    tilemap.SetTile({i, 8}, TileID::kDirt);
  }
  for (uint32_t i = 0; i < 8; ++i) {
    tilemap.SetTile({20 - 1, i}, TileID::kDirt);
  }

  for (int i = 0; i < 5; ++i) {
    auto& mushroom = scene->MakeChild<Mushroom>(&tilemap);
    mushroom.SetLocalPosition({64.F * (static_cast<float>(i) + 1), 64.F});
  }

  auto& player = scene->MakeChild<Player>(&tilemap);
  player.SetLocalPosition({64, 64});

  return scene;
}

}  // namespace game
```
{% endcode %}
