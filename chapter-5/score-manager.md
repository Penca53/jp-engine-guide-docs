# Score Manager

## First User Interface

The score manager component is responsible for maintaining the player's score, which is represented as an integer. Additionally, it handles the rendering of this score as centered text at the top of the game window. As the initial UI element in the game, its implementation will require some attention.

{% code title="game/score_manager.h" %}
```cpp
#pragma once

#include <SFML/Graphics/RenderTarget.hpp>
#include <SFML/Graphics/Text.hpp>
#include <cstdint>

#include "engine/node.h"

namespace game {

class ScoreManager : public ng::Node {
 public:
  explicit ScoreManager(ng::App* app);

  void AddScore(int32_t score);

 protected:
  void Update() override;
  void Draw(sf::RenderTarget& target) override;

 private:
  void UpdateUI();

  sf::Text score_text_;
  int32_t score_ = 0;
};

}  // namespace game
```
{% endcode %}

{% code title="game/score_manager.cc" %}
```cpp
#include "score_manager.h"

#include <SFML/Graphics/RenderTarget.hpp>
#include <cstdint>
#include <string>

#include "engine/app.h"
#include "engine/node.h"

namespace game {

ScoreManager::ScoreManager(ng::App* app)
    : ng::Node(app),
      score_text_(
          GetApp()->GetResourceManager().LoadFont("Roboto-Regular.ttf")) {
  UpdateUI();
}

void ScoreManager::AddScore(int32_t score) {
  score_ += score;
  UpdateUI();
}

void ScoreManager::Update() {
  // Empty for now...
}

void ScoreManager::Draw(sf::RenderTarget& target) {
  target.draw(score_text_, GetGlobalTransform().getTransform());
}

void ScoreManager::UpdateUI() {
  score_text_.setString(std::to_string(score_));
  score_text_.setOrigin(score_text_.getGlobalBounds().size / 2.F);
}

}  // namespace game
```
{% endcode %}

{% code title="game/player.h" %}
```cpp
// -----
#include "score_manager.h" // Add: Include.

class Player : public ng::Node {
 public:
  Player(ng::App* app, ng::Tilemap* tilemap, GameManager* game_manager,
         ScoreManager* score_manager); // Change: Add ScoreManager parameter.
 // -----
 private:
  // -----
  GameManager* game_manager_ = nullptr;
  ScoreManager* score_manager_ = nullptr; // Add: ScoreManager pointer.
  // -----
};
```
{% endcode %}

{% code title="game/player.cc" %}
```cpp
// -----
#include "score_manager.h" // Add: Include.

Player::Player(ng::App* app, ng::Tilemap* tilemap, GameManager* game_manager,
               ScoreManager* score_manager) // Change: Add ScoreManager parameter.
    : ng::Node(app),
      tilemap_(tilemap),
      game_manager_(game_manager),
      score_manager_(score_manager), // Change: Add ScoreManager.
      // -----
      
// -----

void Player::Update() {
  // -----
    if (other->GetParent()->GetName() == "Mushroom") {
      if (velocity_.y > 0) {
        auto* mushroom = dynamic_cast<Mushroom*>(other->GetParent());
        if (!mushroom->GetIsDead()) {
          mushroom->TakeDamage();
          score_manager_->AddScore(100); // Add: Add score when a mushroom is hit.
          velocity_.y = -10;
        }
      }
    } else if (other->GetParent()->GetName() == "Banana") {
      auto* banana = dynamic_cast<Banana*>(other->GetParent());
      if (!banana->GetIsCollected()) {
        banana->Collect();
        score_manager_->AddScore(500); // Add: Add score when a banana is collected.
        banana_sound_.play();
      }
    }
  }
 // -----
}
```
{% endcode %}

The `Player` is responsible for increasing the score managed by the `ScoreManager` whenever it collides with specific designated objects in the game world.
