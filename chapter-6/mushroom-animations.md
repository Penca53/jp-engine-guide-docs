# Mushroom Animations

## Animating the Mushroom

The `Mushroom` has the following states:

* **Run:** The mushroom is actively moving either to the left or to the right.
* **Hit:** The mushroom has been struck by the player or has fallen outside the boundaries of the map.

The current state of the player is determined by the context, which includes information about their life status (`is_dead`).

{% code title="game/mushroom.h" %}
```cpp
// -----
// --- Add: Includes. ---
#include <SFML/Audio/Sound.hpp>

#include "engine/fsm.h"
#include "engine/sprite_sheet_animation.h"
#include "engine/state.h"
// --- End ---
// -----

class Mushroom : public ng::Node {
 // -----
 private:
  // --- Add: Context and Mushroom states.  ---
  struct Context {
    bool is_dead = false;
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

  class HitState : public ng::State<Context> {
   public:
    HitState(ng::State<Context>::ID id, ng::SpriteSheetAnimation animation,
             const sf::SoundBuffer* sound_buffer, ng::Node* node);

   protected:
    void OnEnter() override;

    void Update() override;

   private:
    void Die();

    ng::SpriteSheetAnimation animation_;
    sf::Sound sound_;
    ng::Node* node_ = nullptr;
  };
  // --- End ---
  
  // -----
  bool is_dead_ = false; // Remove.
  // --- Add: Context and FSM. ---
  Context context_;
  ng::FSM<Context> animator_;
  // --- End ---
};
```
{% endcode %}

{% code title="game/mushroom.cc" %}
```cpp
// -----
// --- Add: Includes. ---
#include <SFML/Audio/Sound.hpp>
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

Mushroom::RunState::RunState(ng::State<Context>::ID id,
                             ng::SpriteSheetAnimation animation)
    : ng::State<Context>(std::move(id)), animation_(std::move(animation)) {}

void Mushroom::RunState::OnEnter() {
  animation_.Start();
}

void Mushroom::RunState::Update() {
  animation_.Update();
}

Mushroom::HitState::HitState(ng::State<Context>::ID id,
                             ng::SpriteSheetAnimation animation,
                             const sf::SoundBuffer* sound_buffer,
                             ng::Node* node)
    : ng::State<Context>(std::move(id)),
      animation_(std::move(animation)),
      sound_(*sound_buffer),
      node_(node) {
  animation_.RegisterOnEndCallback([this]() -> void { Die(); });
}

void Mushroom::HitState::OnEnter() {
  animation_.Start();
  sound_.play();
}

void Mushroom::HitState::Update() {
  animation_.Update();
}

void Mushroom::HitState::Die() {
  node_->Destroy();
}
// --- End ---

Mushroom::Mushroom(ng::App* app, const ng::Tilemap* tilemap)
    : ng::Node(app),
      // -----
      sprite_(GetApp()->GetResourceManager().LoadTexture(
          "Mushroom/Run (32x32).png")),
      // --- Add: Default animation state. ---
      animator_(&context_, std::make_unique<RunState>(
                               "run", ng::SpriteSheetAnimation(
                                          &sprite_, &sprite_.getTexture(),
                                          kAnimationTPF))) {
      // --- End ---
  // -----
  // --- Add: All the States and Transitions for the Mushroom. ---
  animator_.AddState(std::make_unique<HitState>(
      "hit",
      ng::SpriteSheetAnimation(
          &sprite_,
          &GetApp()->GetResourceManager().LoadTexture("Mushroom/Hit.png"),
          kAnimationTPF),
      &GetApp()->GetResourceManager().LoadSoundBuffer("Mushroom/Hit_2.wav"),
      this));

  animator_.AddTransition({"run", "hit", [](Context& context) -> bool {
                             return context.is_dead;
                           }});
   // --- End ---
}

bool Mushroom::GetIsDead() const {
  return context_.is_dead; // Change: use the context.
}

// --- Change: Simply set the variable to true and let the FSM do the work. ---
void Mushroom::TakeDamage() {
  context_.is_dead = true; 
}
// --- End ---

void Mushroom::Update() {
  animator_.Update(); // Add: update the FSM.

  if (context_.is_dead) { // Change: use the context.
    return;
  }
  // -----
}

} //  namespace game
```
{% endcode %}
