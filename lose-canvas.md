# Lose Canvas

## You Lost

The `LoseCanvas` becomes visible when the Player dies. To ensure it always fits the screen and remains centered, its size is updated every frame to match the window dimensions, and its position is recalculated to be in the center of the window on each frame. This allows it to respond dynamically to any window resizing.

{% code title="game/lose_canvas.h" %}
```cpp
#pragma once

#include <SFML/Graphics/RectangleShape.hpp>
#include <SFML/Graphics/RenderTarget.hpp>
#include <SFML/Graphics/Text.hpp>

#include "engine/node.h"

namespace game {

class LoseCanvas : public ng::Node {
 public:
  explicit LoseCanvas(ng::App* app);

  void Enable();
  void Disable();

 protected:
  void Draw(sf::RenderTarget& target) override;

 private:
  bool is_enabled_ = false;
  sf::RectangleShape background_;
  sf::Text title_text_;
  sf::Text restart_text_;
};

}  // namespace game
```
{% endcode %}

{% code title="game/lose_canvas.cc" %}
```cpp
#include "lose_canvas.h"

#include <SFML/Graphics/RenderTarget.hpp>
#include <SFML/System/Vector2.hpp>

#include "engine/app.h"
#include "engine/layer.h"
#include "engine/node.h"

namespace game {

LoseCanvas::LoseCanvas(ng::App* app)
    : ng::Node(app),
      title_text_(
          GetApp()->GetResourceManager().LoadFont("Roboto-Regular.ttf")),
      restart_text_(
          GetApp()->GetResourceManager().LoadFont("Roboto-Regular.ttf")) {
  SetName("LoseCanvas");
  SetLayer(ng::Layer::kUI);

  background_.setFillColor(sf::Color(50, 50, 50, 200));

  title_text_.setString("YOU LOST");
  title_text_.setCharacterSize(42);
  title_text_.setStyle(sf::Text::Bold);
  title_text_.setOrigin(title_text_.getGlobalBounds().size / 2.F);

  restart_text_.setString("(press ENTER to restart)");
  restart_text_.setCharacterSize(24);
  restart_text_.setStyle(sf::Text::Italic);
  restart_text_.setOrigin(restart_text_.getGlobalBounds().size / 2.F);
  restart_text_.setPosition(
      sf::Vector2f(0, title_text_.getGlobalBounds().size.y * 2));
}

void LoseCanvas::Enable() {
  is_enabled_ = true;
}

void LoseCanvas::Disable() {
  is_enabled_ = false;
}

void LoseCanvas::Draw(sf::RenderTarget& target) {
  if (!is_enabled_) {
    return;
  }

  background_.setSize(sf::Vector2f(GetApp()->GetWindow().getSize()));
  background_.setOrigin(sf::Vector2f(GetApp()->GetWindow().getSize()) / 2.F);

  target.draw(background_, GetGlobalTransform().getTransform());
  target.draw(title_text_, GetGlobalTransform().getTransform());
  target.draw(restart_text_, GetGlobalTransform().getTransform());
}

}  // namespace game
```
{% endcode %}

{% code title="game/game_manager.h" %}
```cpp
// -----
#include "lose_canvas.h" // Add: Include.

class GameManager : public ng::Node {
 // -----

 private:
  // -----
  LoseCanvas* lose_canvas_ = nullptr; // Add: LoseCanvas pointer.
};
```
{% endcode %}

{% code title="game/game_manager.cc" %}
```cpp
#include "lose_canvas.h" // Add: Include.

namespace game {

// -----

void GameManager::OnAdd() {
  state_ = State::PLAY;
  
  // --- Add: Instantiate the LoseCanvas. ---
  auto& lose_canvas = GetParent()->MakeChild<LoseCanvas>();
  lose_canvas_ = &lose_canvas;
  // --- End ---
}

// -----

void GameManager::Lose() {
  if (state_ != State::PLAY) {
    return;
  }

  state_ = State::LOST;
  lose_sound_.play();
  lose_canvas_->Enable(); // Add: Enable the canvas.
}

// -----

}  // namespace game
```
{% endcode %}

The `GameManager` handles the instantiation of the `LoseCanvas`. The LoseCanvas is instantiated as a direct child of the scene, rather than a child of the GameManager itself. This is because the LoseCanvas belongs to a different rendering layer (UI) than the GameManager (default). Due to the layer filtering mechanism in the Node system, which halts rendering of a subtree when layers don't match at the root, this separate instantiation is necessary to ensure the UI is rendered correctly.
