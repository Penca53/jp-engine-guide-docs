# Scene 6

## Animations and Plants

{% code title="game/default_scene.cc" %}
```cpp
// -----
#include "end.h"
#include "plant.h"

namespace game {

std::unique_ptr<ng::Scene> MakeDefaultScene(ng::App* app) {
  // -----
  // --- Add: Invisible barriers behind the end cup so that the player can't go through it. ---
  tilemap.SetTile({60, 23}, TileID::kInvisibleBarrier);
  tilemap.SetTile({60, 24}, TileID::kInvisibleBarrier);
  tilemap.SetTile({60, 25}, TileID::kInvisibleBarrier);
  tilemap.SetTile({61, 23}, TileID::kInvisibleBarrier);
  tilemap.SetTile({61, 24}, TileID::kInvisibleBarrier);
  tilemap.SetTile({61, 25}, TileID::kInvisibleBarrier);
  // --- End ---

  auto& score_manager = scene->MakeChild<ScoreManager>();
  auto& game_manager = scene->MakeChild<GameManager>();

  auto& player =
      scene->MakeChild<Player>(&tilemap, &game_manager, &score_manager);
  player.SetLocalPosition({8.F * static_cast<float>(tilemap.GetTileSize().x),
                           25.F * static_cast<float>(tilemap.GetTileSize().y)});

  scene->MakeChild<ng::Camera>(1, ng::Layer::kUI);

  auto& camera = scene->MakeChild<ng::Camera>();
  camera.MakeChild<FollowPlayer>(&player, &tilemap);

  // --- Add: End cup. ---
  auto& end = scene->MakeChild<End>(&game_manager);
  end.SetLocalPosition({61 * 32, 24 * 32});
  // --- End ---

  auto& mushroom_0 = scene->MakeChild<Mushroom>(&tilemap);
  mushroom_0.SetLocalPosition({38 * 32, 27 * 32});

  auto& mushroom_1 = scene->MakeChild<Mushroom>(&tilemap);
  mushroom_1.SetLocalPosition({40 * 32, 27 * 32});

  // --- Add: Plant enemies. ---
  auto& plant_0 = scene->MakeChild<Plant>(&tilemap);
  plant_0.SetLocalPosition({52 * 32, (28 * 32) - 10});

  auto& plant_1 = scene->MakeChild<Plant>(&tilemap);
  plant_1.SetLocalPosition({58 * 32, (26 * 32) - 10});
  // --- End ---

  auto& banana_0 = scene->MakeChild<Banana>();
  banana_0.SetLocalPosition({14 * 32, 26 * 32});

  auto& banana_1 = scene->MakeChild<Banana>();
  banana_1.SetLocalPosition({40 * 32, 28 * 32});

  return scene;
}

}  // namespace game
```
{% endcode %}
