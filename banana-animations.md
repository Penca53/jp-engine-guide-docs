# Banana Animations

## Animating the Banana

The `Banana` has only the `Idle` state.

{% code title="game/banana.h" %}
```cpp
// -----
// --- Add: Includes. ---
#include "engine/fsm.h"
#include "engine/sprite_sheet_animation.h"
#include "engine/state.h"
// --- End ---

class Banana : public ng::Node {

// -----
protected:
  // -----
  void Update() override; // Add: Update method.

private:
  // --- Add: Context and Banana states.  ---
  struct Context {};

  class IdleState : public ng::State<Context> {
   public:
    IdleState(ng::State<Context>::ID id, ng::SpriteSheetAnimation animation);

   protected:
    void OnEnter() override;

    void Update() override;

   private:
    ng::SpriteSheetAnimation animation_;
  };
  // --- End ---
  
  // -----
  // --- Add: Context and FSM. ---
  Context context_;
  ng::FSM<Context> animator_;
  // --- End ---
};
```
{% endcode %}

{% code title="game/banana.cc" %}
```cpp
// --- Add: Includes. ---
#include <cstdint>
#include <memory>
#include <utility>

#include "engine/sprite_sheet_animation.h"
#include "engine/state.h"
// --- End ---

namespace game {

// --- Add: Implement all the FSM states. --- 
static constexpr int32_t kAnimationTPF = 4;

Banana::IdleState::IdleState(ng::State<Context>::ID id,
                             ng::SpriteSheetAnimation animation)
    : ng::State<Context>(std::move(id)), animation_(std::move(animation)) {}

void Banana::IdleState::OnEnter() {
  animation_.Start();
}

void Banana::IdleState::Update() {
  animation_.Update();
}
// --- End ---

Banana::Banana(ng::App* app)
    : ng::Node(app),
      sprite_(GetApp()->GetResourceManager().LoadTexture("Banana/Bananas.png")),
      // --- Add: Default animation state. ---
      animator_(&context_, std::make_unique<IdleState>(
                               "run", ng::SpriteSheetAnimation(
                                          &sprite_, &sprite_.getTexture(),
                                          kAnimationTPF))) 
      // --- End ---
      // -----

// --- Add: Update the FSM. ---
void Banana::Update() {
  animator_.Update();
}
// --- End ---

} //  namespace game                                        
```
{% endcode %}
