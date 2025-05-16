# Scene 4

## Physics and GameManager

```cpp
// -----
// Add: Includes. ---
#include "banana.h"
#include "game_manager.h"
// --- End ---

namespace game {

std::unique_ptr<ng::Scene> MakeDefaultScene(ng::App* app) {
  // -----
  // --- Change: Spawn fewer mushrooms. ---
  for (int i = 0; i < 3; ++i) {
    auto& mushroom = scene->MakeChild<Mushroom>(&tilemap);
    mushroom.SetLocalPosition({64.F * (static_cast<float>(i) + 6), 64.F});
  }
  // --- End ---
  // --- Add: Instantiate Banana and GameManager. ---
  auto& banana = scene->MakeChild<Banana>();
  banana.SetLocalPosition({256, 128});

  auto& game_manager = scene->MakeChild<GameManager>();
  // --- End ---

  auto& player = scene->MakeChild<Player>(&tilemap, &game_manager); // Change: Add GameManager argument.
  player.SetLocalPosition({64, 64});

  return scene;
}

}  // namespace game
```
