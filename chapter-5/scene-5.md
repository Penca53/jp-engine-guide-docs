# Scene 5

## Complete Tilemap and UI

The complete contents of `default_scene.cc` are provided here due to the extensive modifications made. This ensures that no changes are overlooked.

{% code title="game/default_scene.cc" %}
```cpp
#include "default_scene.h"

#include <cstdint>
#include <memory>
#include <utility>

#include "background.h"
#include "banana.h"
#include "engine/app.h"
#include "engine/camera.h"
#include "engine/layer.h"
#include "engine/node.h"
#include "engine/scene.h"
#include "engine/tile.h"
#include "engine/tilemap.h"
#include "engine/tileset.h"
#include "follow_player.h"
#include "game_manager.h"
#include "mushroom.h"
#include "player.h"
#include "score_manager.h"
#include "tile_id.h"

namespace game {

std::unique_ptr<ng::Scene> MakeDefaultScene(ng::App* app) {
  auto scene = std::make_unique<ng::Scene>(app);
  scene->SetName("Scene");

  ng::Tileset tileset(
      {32, 32}, &app->GetResourceManager().LoadTexture("Terrain (16x16).png"));

  {
    tileset.AddTile(ng::Tile(TileID::kVoid));
    tileset.AddTile(ng::Tile(TileID::kInvisibleBarrier));

    tileset.AddTile(ng::Tile(TileID::kDirtTopLeft,
                             sf::IntRect({6 * 16, 0 * 16}, {16, 16})));
    tileset.AddTile(ng::Tile(TileID::kDirtTopCenter,
                             sf::IntRect({7 * 16, 0 * 16}, {16, 16})));
    tileset.AddTile(ng::Tile(TileID::kDirtTopRight,
                             sf::IntRect({8 * 16, 0 * 16}, {16, 16})));
    tileset.AddTile(ng::Tile(TileID::kDirtMiddleLeft,
                             sf::IntRect({6 * 16, 1 * 16}, {16, 16})));
    tileset.AddTile(ng::Tile(TileID::kDirtMiddleCenter,
                             sf::IntRect({7 * 16, 1 * 16}, {16, 16})));
    tileset.AddTile(ng::Tile(TileID::kDirtMiddleRight,
                             sf::IntRect({8 * 16, 1 * 16}, {16, 16})));
    tileset.AddTile(ng::Tile(TileID::kDirtBottomLeft,
                             sf::IntRect({6 * 16, 2 * 16}, {16, 16})));
    tileset.AddTile(ng::Tile(TileID::kDirtBottomCenter,
                             sf::IntRect({7 * 16, 2 * 16}, {16, 16})));
    tileset.AddTile(ng::Tile(TileID::kDirtBottomRight,
                             sf::IntRect({8 * 16, 2 * 16}, {16, 16})));

    tileset.AddTile(ng::Tile(TileID::kStoneHorizontalLeft,
                             sf::IntRect({12 * 16, 4 * 16}, {16, 16})));
    tileset.AddTile(ng::Tile(TileID::kStoneHorizontalCenter,
                             sf::IntRect({13 * 16, 4 * 16}, {16, 16})));
    tileset.AddTile(ng::Tile(TileID::kStoneHorizontalRight,
                             sf::IntRect({14 * 16, 4 * 16}, {16, 16})));
    tileset.AddTile(ng::Tile(TileID::kStoneVerticalTop,
                             sf::IntRect({15 * 16, 4 * 16}, {16, 16})));
    tileset.AddTile(ng::Tile(TileID::kStoneVerticalMiddle,
                             sf::IntRect({15 * 16, 5 * 16}, {16, 16})));
    tileset.AddTile(ng::Tile(TileID::kStoneVerticalBottom,
                             sf::IntRect({15 * 16, 6 * 16}, {16, 16})));

    tileset.AddTile(ng::Tile(TileID::kPlasticBlock,
                             sf::IntRect({12 * 16, 9 * 16}, {16, 16})));
  }

  auto tmp_tilemap =
      std::make_unique<ng::Tilemap>(app, sf::Vector2u(64, 32), tileset);
  scene->MakeChild<Background>(
      tmp_tilemap->GetSize().componentWiseMul(tmp_tilemap->GetTileSize()));

  auto& tilemap = *tmp_tilemap;
  scene->AddChild(std::move(tmp_tilemap));

  for (uint32_t i = 0; i < tilemap.GetSize().x; ++i) {
    tilemap.SetTile({i, 0}, TileID::kStoneHorizontalCenter);
  }

  for (uint32_t i = 1; i < tilemap.GetSize().x; ++i) {
    tilemap.SetTile({i, tilemap.GetSize().y - 3}, TileID::kDirtTopCenter);
    tilemap.SetTile({i, tilemap.GetSize().y - 2}, TileID::kDirtMiddleCenter);
    tilemap.SetTile({i, tilemap.GetSize().y - 1}, TileID::kDirtMiddleCenter);
  }

  for (uint32_t i = 0; i < tilemap.GetSize().y; ++i) {
    tilemap.SetTile({0, i}, TileID::kStoneVerticalMiddle);
    tilemap.SetTile({tilemap.GetSize().x - 1, i}, TileID::kStoneVerticalMiddle);
  }

  tilemap.SetTile({13, 27}, TileID::kDirtTopLeft);
  tilemap.SetTile({14, 27}, TileID::kDirtTopRight);
  tilemap.SetTile({13, 28}, TileID::kDirtMiddleLeft);
  tilemap.SetTile({14, 28}, TileID::kDirtMiddleRight);

  tilemap.SetTile({23, 26}, TileID::kDirtTopLeft);
  tilemap.SetTile({24, 26}, TileID::kDirtTopRight);
  tilemap.SetTile({23, 27}, TileID::kDirtMiddleLeft);
  tilemap.SetTile({24, 27}, TileID::kDirtMiddleRight);
  tilemap.SetTile({23, 28}, TileID::kDirtMiddleLeft);
  tilemap.SetTile({24, 28}, TileID::kDirtMiddleRight);

  tilemap.SetTile({26, 29}, TileID::kVoid);
  tilemap.SetTile({26, 30}, TileID::kVoid);
  tilemap.SetTile({26, 31}, TileID::kVoid);
  tilemap.SetTile({27, 29}, TileID::kVoid);
  tilemap.SetTile({27, 30}, TileID::kVoid);
  tilemap.SetTile({27, 31}, TileID::kVoid);

  tilemap.SetTile({25, 29}, TileID::kDirtTopRight);
  tilemap.SetTile({25, 30}, TileID::kDirtMiddleRight);
  tilemap.SetTile({25, 31}, TileID::kDirtMiddleRight);
  tilemap.SetTile({28, 29}, TileID::kDirtTopLeft);
  tilemap.SetTile({28, 30}, TileID::kDirtMiddleLeft);
  tilemap.SetTile({28, 31}, TileID::kDirtMiddleLeft);

  tilemap.SetTile({32, 28}, TileID::kDirtMiddleLeft);
  tilemap.SetTile({33, 28}, TileID::kDirtMiddleRight);
  tilemap.SetTile({32, 27}, TileID::kDirtTopLeft);
  tilemap.SetTile({33, 27}, TileID::kDirtTopRight);

  tilemap.SetTile({41, 26}, TileID::kDirtTopLeft);
  tilemap.SetTile({42, 26}, TileID::kDirtTopRight);
  tilemap.SetTile({41, 27}, TileID::kDirtMiddleLeft);
  tilemap.SetTile({42, 27}, TileID::kDirtMiddleRight);
  tilemap.SetTile({41, 28}, TileID::kDirtMiddleLeft);
  tilemap.SetTile({42, 28}, TileID::kDirtMiddleRight);

  for (uint32_t i = 41 - 4; i < 41; ++i) {
    tilemap.SetTile({i, 25}, TileID::kPlasticBlock);
  }

  tilemap.SetTile({46, 29}, TileID::kVoid);
  tilemap.SetTile({46, 30}, TileID::kVoid);
  tilemap.SetTile({46, 31}, TileID::kVoid);
  tilemap.SetTile({47, 29}, TileID::kVoid);
  tilemap.SetTile({47, 30}, TileID::kVoid);
  tilemap.SetTile({47, 31}, TileID::kVoid);
  tilemap.SetTile({48, 29}, TileID::kVoid);
  tilemap.SetTile({48, 30}, TileID::kVoid);
  tilemap.SetTile({48, 31}, TileID::kVoid);
  tilemap.SetTile({49, 29}, TileID::kVoid);
  tilemap.SetTile({49, 30}, TileID::kVoid);
  tilemap.SetTile({49, 31}, TileID::kVoid);

  tilemap.SetTile({45, 29}, TileID::kDirtTopRight);
  tilemap.SetTile({45, 30}, TileID::kDirtMiddleRight);
  tilemap.SetTile({45, 31}, TileID::kDirtMiddleRight);
  tilemap.SetTile({50, 29}, TileID::kDirtTopLeft);
  tilemap.SetTile({50, 30}, TileID::kDirtMiddleLeft);
  tilemap.SetTile({50, 31}, TileID::kDirtMiddleLeft);

  tilemap.SetTile({53, 28}, TileID::kDirtTopLeft);
  tilemap.SetTile({54, 28}, TileID::kDirtTopCenter);
  tilemap.SetTile({55, 28}, TileID::kDirtTopCenter);
  tilemap.SetTile({56, 28}, TileID::kDirtTopRight);
  tilemap.SetTile({57, 28}, TileID::kDirtMiddleCenter);
  tilemap.SetTile({58, 28}, TileID::kDirtMiddleCenter);
  tilemap.SetTile({59, 28}, TileID::kDirtMiddleCenter);
  tilemap.SetTile({60, 28}, TileID::kDirtMiddleCenter);
  tilemap.SetTile({61, 28}, TileID::kDirtMiddleCenter);
  tilemap.SetTile({62, 28}, TileID::kDirtMiddleCenter);

  tilemap.SetTile({56, 27}, TileID::kDirtTopLeft);
  tilemap.SetTile({57, 27}, TileID::kDirtTopCenter);
  tilemap.SetTile({58, 27}, TileID::kDirtTopCenter);
  tilemap.SetTile({59, 27}, TileID::kDirtTopRight);
  tilemap.SetTile({60, 27}, TileID::kDirtMiddleCenter);
  tilemap.SetTile({61, 27}, TileID::kDirtMiddleCenter);
  tilemap.SetTile({62, 27}, TileID::kDirtMiddleCenter);

  tilemap.SetTile({59, 26}, TileID::kDirtTopLeft);
  tilemap.SetTile({60, 26}, TileID::kDirtTopCenter);
  tilemap.SetTile({61, 26}, TileID::kDirtTopCenter);
  tilemap.SetTile({62, 26}, TileID::kDirtTopCenter);

  auto& score_manager = scene->MakeChild<ScoreManager>();
  auto& game_manager = scene->MakeChild<GameManager>();

  auto& player =
      scene->MakeChild<Player>(&tilemap, &game_manager, &score_manager);
  
  player.SetLocalPosition({8.F * static_cast<float>(tilemap.GetTileSize().x),
                           25.F * static_cast<float>(tilemap.GetTileSize().y)});

  scene->MakeChild<ng::Camera>(1, ng::Layer::kUI);

  auto& camera = scene->MakeChild<ng::Camera>();
  camera.MakeChild<FollowPlayer>(&player, &tilemap);

  auto& mushroom_0 = scene->MakeChild<Mushroom>(&tilemap);
  mushroom_0.SetLocalPosition({38 * 32, 27 * 32});

  auto& mushroom_1 = scene->MakeChild<Mushroom>(&tilemap);
  mushroom_1.SetLocalPosition({40 * 32, 27 * 32});

  auto& banana_0 = scene->MakeChild<Banana>();
  banana_0.SetLocalPosition({14 * 32, 26 * 32});

  auto& banana_1 = scene->MakeChild<Banana>();
  banana_1.SetLocalPosition({40 * 32, 28 * 32});

  return scene;
}

}  // namespace game
```
{% endcode %}

First, additional tiles are incorporated into the Tileset. Following this, the complete Tilemap is constructed using this expanded set of tiles.

You might observe two cameras present in the scene. The first is a standard camera responsible for rendering gameplay elements within the game world's coordinate system (world-space). The second is a dedicated UI camera, specifically used to render user interface elements in screen-space, ensuring they remain fixed relative to the display.
