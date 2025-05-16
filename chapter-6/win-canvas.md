# Win Canvas

## You Won

The `WinCanvas` becomes visible when the Player wins. To ensure it always fits the screen and remains centered, its size is updated every frame to match the window dimensions, and its position is recalculated to be in the center of the window on each frame. This allows it to respond dynamically to any window resizing.

{% code title="game/win_canvas.h" %}
```cpp
#pragma once

#include <SFML/Graphics/RectangleShape.hpp>
#include <SFML/Graphics/RenderTarget.hpp>
#include <SFML/Graphics/Text.hpp>

#include "engine/node.h"

namespace game {

class WinCanvas : public ng::Node {
 public:
  explicit WinCanvas(ng::App* app);

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

{% code title="game/win_canvas.cc" %}
```cpp
#include "win_canvas.h"

#include <SFML/Graphics/RenderTarget.hpp>
#include <SFML/Graphics/Text.hpp>
#include <SFML/System/Vector2.hpp>

#include "engine/app.h"
#include "engine/layer.h"
#include "engine/node.h"

namespace game {

WinCanvas::WinCanvas(ng::App* app)
    : ng::Node(app),
      title_text_(
          GetApp()->GetResourceManager().LoadFont("Roboto-Regular.ttf")),
      restart_text_(
          GetApp()->GetResourceManager().LoadFont("Roboto-Regular.ttf")) {
  SetName("WinCanvas");
  SetLayer(ng::Layer::kUI);

  background_.setFillColor(sf::Color(50, 50, 50, 200));

  title_text_.setString("YOU WON");
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

void WinCanvas::Enable() {
  is_enabled_ = true;
}

void WinCanvas::Disable() {
  is_enabled_ = false;
}

void WinCanvas::Draw(sf::RenderTarget& target) {
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
#include "win_canvas.h" // Add: Include.

class GameManager : public ng::Node {
 // -----

 private:
  // -----
  sf::Sound lose_sound_; 
  sf::Sound win_sound_; // Add: Win sound.
  
  LoseCanvas* lose_canvas_ = nullptr;
  WinCanvas* win_canvas_ = nullptr; // Add: WinCanvas pointer.
};
```
{% endcode %}

{% code title="game/game_manager.cc" %}
```cpp
#include "win_canvas.h" // Add: Include.

namespace game {

GameManager::GameManager(ng::App* app)
    : ng::Node(app),
      lose_sound_(
          GetApp()->GetResourceManager().LoadSoundBuffer("Loose_2.wav")),
      win_sound_(GetApp()->GetResourceManager().LoadSoundBuffer("Win_2.wav")) // Add: Load win sound.
      {}

// -----

void GameManager::OnAdd() {
  // -----
  
  // --- Add: Instantiate the WinCanvas. ---
  auto& win_canvas = GetParent()->MakeChild<WinCanvas>();
  win_canvas_ = &win_canvas;
  // --- End ---
}

// -----

void GameManager::Win() {
  if (state_ != State::PLAY) {
    return;
  }

  state_ = State::WON;
  // --- Add: Play sound and show win canvas. ---
  win_sound_.play();
  win_canvas_->Enable();
  // --- End ---
}

// -----

}  // namespace game
```
{% endcode %}
