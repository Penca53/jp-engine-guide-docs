# Banana

## Banana 🍌&#x20;

Bananas are designed as collectible items. Currently, their behavior upon collection is to be destroyed. In the next chapter, this will be updated to provide the player with a bonus score.

{% code title="game/banana.h" %}
```cpp
#pragma once

#include <SFML/Graphics/RenderTarget.hpp>
#include <SFML/Graphics/Sprite.hpp>

#include "engine/circle_collider.h"
#include "engine/node.h"

namespace game {

class Banana : public ng::Node {
 public:
  explicit Banana(ng::App* app);

  bool GetIsCollected() const;
  void Collect();

 protected:
  void Draw(sf::RenderTarget& target) override;

 private:
  sf::Sprite sprite_;
  bool is_collected_ = false;
};

}  // namespace game
```
{% endcode %}

{% code title="game/banana.cc" %}
```cpp
#include "banana.h"

#include <SFML/Graphics/RenderTarget.hpp>
#include <SFML/Graphics/Sprite.hpp>

#include "engine/app.h"
#include "engine/circle_collider.h"
#include "engine/node.h"

namespace game {

Banana::Banana(ng::App* app)
    : ng::Node(app),
      sprite_(
          GetApp()->GetResourceManager().LoadTexture("Banana/Bananas.png")) {
  SetName("Banana");
  sprite_.setScale({2, 2});
  sprite_.setOrigin({16, 16});
  sprite_.setTextureRect(sf::IntRect({0, 0}, {32, 32}));

  MakeChild<ng::CircleCollider>(16.F);
}

bool Banana::GetIsCollected() const {
  return is_collected_;
}

void Banana::Collect() {
  if (is_collected_) {
    return;
  }

  is_collected_ = true;
  Destroy();
}

void Banana::Draw(sf::RenderTarget& target) {
  target.draw(sprite_, GetGlobalTransform().getTransform());
}

}  // namespace game
```
{% endcode %}
