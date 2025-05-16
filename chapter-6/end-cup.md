# End Cup

## The End

The `End` Cup is the winning objective; when the Player jumps on it, a squish animation plays, confirming the victory. The `WinCanvas` is then presented, allowing the user to press Enter to restart the scene.

{% code title="game/end.h" %}
```cpp
#pragma once

#include <SFML/Graphics/RenderTarget.hpp>
#include <SFML/Graphics/Sprite.hpp>
#include <SFML/System/Vector2.hpp>

#include "engine/fsm.h"
#include "engine/node.h"
#include "engine/sprite_sheet_animation.h"
#include "game_manager.h"

namespace game {

class End : public ng::Node {
 public:
  End(ng::App* app, GameManager* game_manager);

  void EndGame();

 protected:
  void Update() override;
  void Draw(sf::RenderTarget& target) override;

 private:
  struct Context {
    bool is_pressed_ = false;
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

  class PressedState : public ng::State<Context> {
   public:
    PressedState(ng::State<Context>::ID id, ng::SpriteSheetAnimation animation,
                 GameManager* game_manager);

   protected:
    void OnEnter() override;

    void Update() override;

   private:
    ng::SpriteSheetAnimation animation_;
    GameManager* game_manager_ = nullptr;
  };

  sf::Sprite sprite_;
  Context context_;
  ng::FSM<Context> animator_;
  GameManager* game_manager_ = nullptr;
};

}  // namespace game
```
{% endcode %}

{% code title="game/end.cc" %}
```cpp
#include "end.h"

#include <SFML/Graphics/RenderTarget.hpp>
#include <SFML/Graphics/Sprite.hpp>
#include <cstdint>
#include <memory>
#include <utility>

#include "engine/app.h"
#include "engine/node.h"
#include "engine/rectangle_collider.h"
#include "engine/sprite_sheet_animation.h"
#include "engine/state.h"
#include "engine/transition.h"
#include "game_manager.h"

namespace game {

static constexpr int32_t kAnimationTPF = 4;

End::IdleState::IdleState(ng::State<Context>::ID id,
                          ng::SpriteSheetAnimation animation)
    : ng::State<Context>(std::move(id)), animation_(std::move(animation)) {}

void End::IdleState::OnEnter() {
  animation_.Start();
}

void End::IdleState::Update() {
  animation_.Update();
}

End::PressedState::PressedState(ng::State<Context>::ID id,
                                ng::SpriteSheetAnimation animation,
                                GameManager* game_manager)
    : ng::State<Context>(std::move(id)),
      animation_(std::move(animation)),
      game_manager_(game_manager) {}

void End::PressedState::OnEnter() {
  animation_.Start();
  animation_.RegisterOnEndCallback([this]() {
    game_manager_->Win(); // Win the game!
    GetContext()->is_pressed_ = false;
  });
}

void End::PressedState::Update() {
  animation_.Update();
}

End::End(ng::App* app, GameManager* game_manager)
    : ng::Node(app),
      sprite_(GetApp()->GetResourceManager().LoadTexture("End/End (Idle).png")),
      animator_(&context_, std::make_unique<IdleState>(
                               "idle", ng::SpriteSheetAnimation(
                                           &sprite_, &sprite_.getTexture(),
                                           kAnimationTPF))),
      game_manager_(game_manager) {
  SetName("End");
  sprite_.setScale({2, 2});
  sprite_.setOrigin({32, 32});

  // The collider is smaller than the cup itself and is placed slightly above the cup.
  // The collider is solely used to detect the player's win, not movement collisions.
  auto& collider = MakeChild<ng::RectangleCollider>(sf::Vector2f(60, 32));
  collider.SetLocalPosition({0, -20});

  animator_.AddState(std::make_unique<PressedState>(
      "pressed",
      ng::SpriteSheetAnimation(&sprite_,
                               &GetApp()->GetResourceManager().LoadTexture(
                                   "End/End (Pressed) (64x64).png"),
                               kAnimationTPF),
      game_manager_));

  animator_.AddTransition({"idle", "pressed", [](Context& context) -> bool {
                             return context.is_pressed_;
                           }});
  animator_.AddTransition({"pressed", "idle", [](Context& context) -> bool {
                             return !context.is_pressed_;
                           }});
}

void End::EndGame() {
  context_.is_pressed_ = true;
}

void End::Update() {
  animator_.Update();
}

void End::Draw(sf::RenderTarget& target) {
  target.draw(sprite_, GetGlobalTransform().getTransform());
}

}  // namespace game
```
{% endcode %}

{% code title="game/player.h" %}
```cpp
// -----
class Player : public ng::Node {
  // -----
  private:
    // -----
    bool has_won_ = false; // Add: Has the player won?
};
```
{% endcode %}

{% code title="game/player.cc" %}
```cpp
// -----
#include "end.h" // Add: Include.

// -----
void Player::Update() {
  // -----
  if (context_.is_dead) {
    return;
  }

  // --- Change: Input detection only when the player hasn't won yet. ---
  sf::Vector2f direction;
  if (!has_won_) {
    if (GetApp()->GetInput().GetKey(sf::Keyboard::Scancode::A)) {
      direction.x += -1;
      sprite_.setScale(sf::Vector2f{-2.F, 2.F});
    }
    if (GetApp()->GetInput().GetKey(sf::Keyboard::Scancode::D)) {
      direction.x += 1;
      sprite_.setScale(sf::Vector2f{2.F, 2.F});
    }
  }
  
  static constexpr float kMovementSpeed = 4.F;
  context_.velocity.x = direction.x * kMovementSpeed;
  context_.velocity.y += 1;

  if (!has_won_ && context_.is_on_ground &&  // Change: Add has_won_ condition.
      GetApp()->GetInput().GetKeyDown(sf::Keyboard::Scancode::Space)) {
    context_.velocity.y -= 15;
  }
  // --- End ---
  
  // -----
 
  std::vector<const ng::Collider*> others =
       GetScene()->GetPhysics().Overlap(*collider_);
  for (const auto* other : others) {
    // -----
    // --- Add: Collision with End Cup. ---
    else if (other->GetParent()->GetName() == "End") {
       if (!has_won_) {
         context_.velocity.y = -15;
         dynamic_cast<End*>(other->GetParent())->EndGame();
         has_won_ = true;
       }
    }
    // --- End ---
  }
  // -----
}
```
{% endcode %}

Upon touching the `End` Cup, the `Player` causes it to bounce vertically. This contact also marks the match as won for the player, preventing any movement or jump.
