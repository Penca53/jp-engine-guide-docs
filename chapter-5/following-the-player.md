# Following the Player

## Follow Player

The `FollowPlayer` node, which will be positioned as a child of the main `Camera` Node, manages the camera's position by tracking the Player's movement. It also enforces boundaries, ensuring the camera's view never extends beyond the Tilemap's edges, thus preventing the blank background from being visible to the user.&#x20;

Notice that the script includes a check to ensure the Player Node is still valid before attempting to access its position. This precaution is necessary because the Player might be destroyed during gameplay (e.g., upon being hit), and accessing an invalid pointer would cause an error.

{% code title="game/follow_player.h" %}
```cpp
#pragma once

#include "engine/node.h"
#include "engine/tilemap.h"
#include "player.h"

namespace game {

class FollowPlayer : public ng::Node {
 public:
  FollowPlayer(ng::App* app, const Player* player, const ng::Tilemap* tilemap);

 protected:
  void OnAdd() override;
  void Update() override;

 private:
  void Follow();

  const Player* player_ = nullptr;
  const ng::Tilemap* tilemap_ = nullptr;
};

}  // namespace game
```
{% endcode %}

{% code title="game/follow_player.cc" %}
```cpp
#include "follow_player.h"

#include <SFML/System/Vector2.hpp>
#include <algorithm>

#include "engine/node.h"
#include "engine/scene.h"
#include "engine/tilemap.h"
#include "player.h"

namespace game {

FollowPlayer::FollowPlayer(ng::App* app, const Player* player,
                           const ng::Tilemap* tilemap)
    : ng::Node(app), player_(player), tilemap_(tilemap) {}

void FollowPlayer::OnAdd() {
  Follow();
}

void FollowPlayer::Update() {
  Follow();
}

void FollowPlayer::Follow() {
  if (!GetScene()->IsValid(player_)) {
    return;
  }

  sf::Vector2f tilemap_size = sf::Vector2f(tilemap_->GetSize());
  sf::Vector2f tile_size = sf::Vector2f(tilemap_->GetTileSize());
  sf::Vector2f window_size = sf::Vector2f(GetApp()->GetWindow().getSize());
  sf::Vector2f player_pos = player_->GetGlobalTransform().getPosition();
  sf::Vector2f new_pos(
      std::min(
          std::max(player_pos.x, tile_size.x * window_size.x / tile_size.x / 2),
          tile_size.x * (tilemap_size.x - window_size.x / tile_size.x / 2)),
      std::min(
          std::max(player_pos.y, tile_size.y * window_size.y / tile_size.y / 2),
          tile_size.y * (tilemap_size.y - window_size.y / tile_size.y / 2)));
  GetParent()->SetLocalPosition(new_pos);
}

}  // namespace game
```
{% endcode %}

{% code title="game/player.cc" %}
```cpp
#include <SFML/Graphics/View.hpp> // Remove: Not needed anymore.

// -----

void Player::Draw(sf::RenderTarget& target) {
  // --- Remove. ---
  sf::View v = target.getView();
  v.setCenter(GetLocalTransform().getPosition());
  target.setView(v);
  // --- End ---

  target.draw(sprite_, GetGlobalTransform().getTransform());
}
```
{% endcode %}

The previous method of directly centering the view from within the player node is now obsolete. You can safely remove that centering logic.
