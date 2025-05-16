# Player Animations

## Animating the Player

The `Player` has the following states:

* **Idle:** The player is stationary and not moving.
* **Run:** The player is actively moving either to the left or to the right.
* **Jump:** The player is currently moving upwards after initiating a jump.
* **Fall:** The player is moving downwards, typically due to gravity, after a jump or falling from a platform.
* **Hit:** The player has been struck by an enemy or has fallen outside the boundaries of the map.

The current state of the player is determined by the context, which includes information such as the player's `velocity`, whether they are currently on the ground (`is_on_ground`), and their life status (`is_dead`).

{% code title="game/player.h" %}
```cpp
// -----
// --- Add: Includes. ---
#include "engine/fsm.h"
#include "engine/sprite_sheet_animation.h"
#include "engine/state.h"
// --- End ---
// -----

class Player : public ng::Node {
// -----
private:
  // --- Add: Context and Player states. ---
  struct Context {
    sf::Vector2f velocity;
    bool is_on_ground = false;
    bool is_dead = false;
  };

  class IdleState : public ng::State<Context> {
   public:
    IdleState(ng::State<Context>::ID id, ng::SpriteSheetAnimation animation);

   protected:
    void OnEnter() override;

    void Update() override;

   private:
    ng::SpriteSheetAnimation animation_;
  };

  class RunState : public ng::State<Context> {
   public:
    RunState(ng::State<Context>::ID id, ng::SpriteSheetAnimation animation);

   protected:
    void OnEnter() override;

    void Update() override;

   private:
    ng::SpriteSheetAnimation animation_;
  };

  class JumpState : public ng::State<Context> {
   public:
    JumpState(ng::State<Context>::ID id, ng::SpriteSheetAnimation animation,
              const sf::SoundBuffer* sound_buffer);

   protected:
    void OnEnter() override;

    void Update() override;

   private:
    ng::SpriteSheetAnimation animation_;
    sf::Sound sound_;
  };

  class FallState : public ng::State<Context> {
   public:
    FallState(ng::State<Context>::ID id, ng::SpriteSheetAnimation animation);

   protected:
    void OnEnter() override;

    void Update() override;

   private:
    ng::SpriteSheetAnimation animation_;
  };

  class HitState : public ng::State<Context> {
   public:
    HitState(ng::State<Context>::ID id, ng::SpriteSheetAnimation animation,
             ng::Node* node, GameManager* game_manager);

   protected:
    void OnEnter() override;

    void Update() override;

   private:
    void Die();

    ng::SpriteSheetAnimation animation_;
    ng::Node* node_ = nullptr;
    GameManager* game_manager_ = nullptr;
  };
  // --- End ---

  // -----
  // --- Remove: The context has these already. ---
  sf::Vector2f velocity_;
  bool is_on_ground_ = false;
  bool is_dead_ = false;
  // --- End ---
  // --- Add: Context and FSM. ---
  Context context_;
  ng::FSM<Context> animator_;
  // --- End ---
};
```
{% endcode %}

{% code title="game/player.cc" %}
```cpp
// -----
// --- Add: Includes. ---
#include <SFML/Audio/SoundBuffer.hpp>
#include <cstdint>
#include <memory>
#include <utility>

#include "engine/fsm.h"
#include "engine/sprite_sheet_animation.h"
#include "engine/state.h"
#include "engine/transition.h"
// --- End ---
// -----

namespace game {

// --- Add: Implement all the FSM states. ---
static constexpr int32_t kAnimationTPF = 4;

Player::IdleState::IdleState(ng::State<Context>::ID id,
                             ng::SpriteSheetAnimation animation)
    : ng::State<Context>(std::move(id)), animation_(std::move(animation)) {}

void Player::IdleState::OnEnter() {
  animation_.Start();
}

void Player::IdleState::Update() {
  animation_.Update();
}

Player::RunState::RunState(ng::State<Context>::ID id,
                           ng::SpriteSheetAnimation animation)
    : ng::State<Context>(std::move(id)), animation_(std::move(animation)) {}

void Player::RunState::OnEnter() {
  animation_.Start();
}

void Player::RunState::Update() {
  animation_.Update();
}

Player::JumpState::JumpState(ng::State<Context>::ID id,
                             ng::SpriteSheetAnimation animation,
                             const sf::SoundBuffer* sound_buffer)
    : ng::State<Context>(std::move(id)),
      animation_(std::move(animation)),
      sound_(*sound_buffer) {}

void Player::JumpState::OnEnter() {
  animation_.Start();
  sound_.play();
}

void Player::JumpState::Update() {
  animation_.Update();
}

Player::FallState::FallState(ng::State<Context>::ID id,
                             ng::SpriteSheetAnimation animation)
    : ng::State<Context>(std::move(id)), animation_(std::move(animation)) {}

void Player::FallState::OnEnter() {
  animation_.Start();
}

void Player::FallState::Update() {
  animation_.Update();
}

Player::HitState::HitState(ng::State<Context>::ID id,
                           ng::SpriteSheetAnimation animation, ng::Node* node,
                           GameManager* game_manager)
    : ng::State<Context>(std::move(id)),
      animation_(std::move(animation)),
      node_(node),
      game_manager_(game_manager) {
  animation_.RegisterOnEndCallback([this]() { Die(); });
}

void Player::HitState::OnEnter() {
  animation_.Start();
}

void Player::HitState::Update() {
  animation_.Update();
}

void Player::HitState::Die() {
  game_manager_->Lose();
  node_->Destroy();
}
// --- End ---

Player::Player(ng::App* app, ng::Tilemap* tilemap, GameManager* game_manager,
               ScoreManager* score_manager)
    : ng::Node(app),
      // -----
      sprite_(GetApp()->GetResourceManager().LoadTexture(
          "Player/Idle (32x32).png")),
      // --- Add: Default animation state. ---
      animator_(&context_, std::make_unique<IdleState>(
                               "idle", ng::SpriteSheetAnimation(
                                           &sprite_, &sprite_.getTexture(),
                                           kAnimationTPF))) {
      // --- End ---
      
  // -----
  // --- Add: All the States and Transitions for the Player. --- 
  animator_.AddState(std::make_unique<RunState>(
      "run",
      ng::SpriteSheetAnimation(
          &sprite_,
          &GetApp()->GetResourceManager().LoadTexture("Player/Run (32x32).png"),
          kAnimationTPF)));
  animator_.AddState(std::make_unique<JumpState>(
      "jump",
      ng::SpriteSheetAnimation(&sprite_,
                               &GetApp()->GetResourceManager().LoadTexture(
                                   "Player/Jump (32x32).png"),
                               kAnimationTPF),
      &GetApp()->GetResourceManager().LoadSoundBuffer("Player/Jump_2.wav")));
  animator_.AddState(std::make_unique<FallState>(
      "fall",
      ng::SpriteSheetAnimation(&sprite_,
                               &GetApp()->GetResourceManager().LoadTexture(
                                   "Player/Fall (32x32).png"),
                               kAnimationTPF)));
  animator_.AddState(std::make_unique<HitState>(
      "hit",
      ng::SpriteSheetAnimation(
          &sprite_,
          &GetApp()->GetResourceManager().LoadTexture("Player/Hit (32x32).png"),
          kAnimationTPF),
      this, game_manager_));

  animator_.AddTransition({"idle", "run", [](Context& context) -> bool {
                             return context.velocity.x != 0;
                           }});
  animator_.AddTransition({"run", "idle", [](Context& context) -> bool {
                             return context.velocity.x == 0;
                           }});

  animator_.AddTransition({"idle", "jump", [](Context& context) -> bool {
                             return context.velocity.y < 0;
                           }});
  animator_.AddTransition({"run", "jump", [](Context& context) -> bool {
                             return context.velocity.y < 0;
                           }});

  animator_.AddTransition({"jump", "fall", [](Context& context) -> bool {
                             return context.velocity.y > 0 &&
                                    !context.is_on_ground;
                           }});

  animator_.AddTransition({"jump", "idle", [](Context& context) -> bool {
                             return context.is_on_ground;
                           }});
  animator_.AddTransition({"fall", "idle", [](Context& context) -> bool {
                             return context.is_on_ground;
                           }});

  animator_.AddTransition({"idle", "hit", [](Context& context) -> bool {
                             return context.is_dead;
                           }});
  animator_.AddTransition({"run", "hit", [](Context& context) -> bool {
                             return context.is_dead;
                           }});
  animator_.AddTransition({"jump", "hit", [](Context& context) -> bool {
                             return context.is_dead;
                           }});
  animator_.AddTransition({"fall", "hit", [](Context& context) -> bool {
                             return context.is_dead;
                           }});
  // --- End ---
}

sf::Vector2f Player::GetVelocity() const {
  return context_.velocity; // Change: use the context.
}

// --- Change: Simply set the variable to true and let the FSM do the work. ---
void Player::TakeDamage() {
  context_.is_dead = true;
}
// --- End ---

void Player::Update() {
  animator_.Update(); // Add: Update the FSM.

  if (context_.is_dead) {
    return;
  }
  // -----
  
  // Change: Substitute all player_velocity_ and is_on_ground_ with the context variables.
  // player_velocity_ -> context.velocity
}

}  // namespace game
```
{% endcode %}
