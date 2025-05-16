# Game Manager

## Central Game State

The `GameManager` is a common design pattern used in game development. It's essentially a script or object that acts as a central controller or orchestrator for the overall game. It oversees and coordinates many different parts of the game, such as keeping track of the player's progress, managing the player, and coordinating the whole scene.

In this case, its job is to keep track of the play `State` of the game and manage the Win / Loss states by playing sounds and enabling the user to restart the game.

{% code title="game/game_manager.h" %}
```cpp
#pragma once

#include <SFML/Audio/Sound.hpp>
#include <cstdint>

#include "engine/node.h"

namespace game {

class GameManager : public ng::Node {
 public:
  enum class State : uint8_t {
    BEGIN = 0,
    PLAY,
    WON,
    LOST,
  };

  explicit GameManager(ng::App* app);

  void Win();
  void Lose();

  State GetState() const;

 protected:
  void OnAdd() override;
  void Update() override;

 private:
  State state_{};
  sf::Sound lose_sound_;
};

}  // namespace game
```
{% endcode %}

{% code title="game/game_manager.cc" %}
```cpp
#include "game_manager.h"

#include <SFML/Audio/Sound.hpp>
#include <SFML/Window/Keyboard.hpp>

#include "default_scene.h"
#include "engine/app.h"
#include "engine/node.h"

namespace game {

GameManager::GameManager(ng::App* app)
    : ng::Node(app),
      lose_sound_(
          GetApp()->GetResourceManager().LoadSoundBuffer("Loose_2.wav")) {}

void GameManager::OnAdd() {
  state_ = State::PLAY;
}

void GameManager::Update() {
  if (state_ == State::WON || state_ == State::LOST) {
    if (GetApp()->GetInput().GetKeyDown(sf::Keyboard::Scancode::Enter)) {
      GetApp()->LoadScene(MakeDefaultScene(GetApp()));
      return;
    }
  }
}

void GameManager::Win() {
  if (state_ != State::PLAY) {
    return;
  }

  state_ = State::WON;
}

void GameManager::Lose() {
  if (state_ != State::PLAY) {
    return;
  }

  state_ = State::LOST;
  lose_sound_.play();
}

GameManager::State GameManager::GetState() const {
  return state_;
}

}  // namespace game
```
{% endcode %}

{% code title="game/player.h" %}
```cpp
// -----
#include "game_manager.h" // Add: Include.

class Player : public ng::Node {
 public:
  Player(ng::App* app, const ng::Tilemap* tilemap, GameManager* game_manager); // Change: Add GameManager parameter.
  
  // -----
 private:
  const ng::Tilemap* tilemap_ = nullptr;
  GameManager* game_manager_ = nullptr; // Add: GameManager pointer.
  // -----
};
```
{% endcode %}

{% code title="game/player.cc" %}
```cpp
// -----
#include "game_manager.h" // Add: Include.

namespace game {

Player::Player(ng::App* app, const ng::Tilemap* tilemap,
               GameManager* game_manager) // Change: Add GameManager parameter.
    : ng::Node(app),
      tilemap_(tilemap),
      game_manager_(game_manager), // Change: Add GameManager parameter.
      // -----
      
void Player::TakeDamage() {
  if (is_dead_) {
    return;
  }

  is_dead_ = true;
  Destroy();
  game_manager_->Lose(); // Add: Call the Lose game.
}

// -----
      
}  //  namespace game
```
{% endcode %}
