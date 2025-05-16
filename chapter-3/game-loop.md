# Game Loop

The **Game Loop** is the core, continuously running cycle that drives the entire game forward. It's the heartbeat of the application. While implementations vary, a typical game loop performs these fundamental steps repeatedly, many times per second (aiming for 30, 60, or even higher frames per second):

1. **Process Input:** The engine checks for any input from the player (keyboard presses, mouse movements, controller buttons, touch events, etc.) or other sources (like network messages).
2. **Update Game State (Logic/Simulation):** This is where the game world changes based on input, time elapsed, AI, physics, and game rules.
3. **Render Frame (Drawing):** Based on the _updated_ game state, the engine draws the visual representation of the scene onto the screen for the player to see.

Different [game loop](https://gameprogrammingpatterns.com/game-loop.html) designs offer various trade-offs. This guide will use the "[Play catch up](https://gameprogrammingpatterns.com/game-loop.html#play-catch-up)" method, which divides the loop into "ticks" for updates and "frames" for rendering. TPS (ticks per second) establishes a fixed update rate, while FPS (frames per second) sets the target rendering rate. This decoupling means that the game's logic updates (TPS) and its visual presentation (FPS) are independent. A prime example is Minecraft, where a lagging TPS (often around 20) will only impact the simulation's speed, not the smoothness of the rendering.

{% code title="engine/app.h" %}
```cpp
#pragma once

#include <SFML/Graphics/RenderWindow.hpp>
#include <SFML/System/String.hpp>
#include <cstdint>

#include "input.h"
#include "resource_manager.h"

namespace ng {

/// @brief The core application class, managing the game loop, window, resources, input, and scenes.
class App {
 public:
  /// @brief Constructs an App instance with the specified window size and title, ticks and frames per second.
  /// @param window_size The initial size of the game window.
  /// @param window_title The title of the game window.
  /// @param tps The target ticks per second (game logic updates).
  /// @param fps The target frames per second (rendering updates).
  App(sf::Vector2u window_size, const sf::String& window_title, uint32_t tps,
      uint32_t fps);
  ~App() = default;

  App(const App& other) = delete;
  App& operator=(const App& other) = delete;
  App(App&& other) = delete;
  App& operator=(App&& other) = delete;

  /// @brief Runs the main game loop.
  void Run();

  /// @brief Returns the duration of a single game tick in seconds.
  /// @return The time elapsed per tick.
  [[nodiscard]] std::chrono::duration<float> SecondsPerTick() const;

  /// @brief Returns the duration of a single game tick in nanoseconds.
  /// @return The time elapsed per tick.
  [[nodiscard]] std::chrono::nanoseconds NanosecondsPerTick() const;

  /// @brief Returns a reference to the SFML RenderWindow.
  /// @return A reference to the game window object.
  [[nodiscard]] sf::RenderWindow& GetWindow();

  /// @brief Returns a reference to the ResourceManager for managing game assets.
  /// @return A reference to the ResourceManager.
  [[nodiscard]] ResourceManager& GetResourceManager();

  /// @brief Returns a constant reference to the Input manager.
  /// @return A constant reference to the Input manager.
  [[nodiscard]] const Input& GetInput() const;

 private:
  /// @brief Polls for SFML window events and updates the input state.
  void PollInput();

  // The main SFML render window.
  sf::RenderWindow window_;

  // Target ticks per second for game logic updates.
  uint32_t tps_ = 0;
  // Target frames per second for rendering.
  uint32_t fps_ = 0;

  // Manages game resources like textures and sounds.
  ResourceManager resource_manager_;
  // Handles user input events.
  Input input_;
};

}  // namespace ng
```
{% endcode %}

{% code title="engine/app.cc" %}
```cpp
#include "app.h"

#include <SFML/System/String.hpp>
#include <SFML/System/Vector2.hpp>
#include <SFML/Window/Event.hpp>
#include <SFML/Window/VideoMode.hpp>
#include <chrono>
#include <cstdint>
#include <optional>

#include "input.h"
#include "resource_manager.h"

namespace ng {

App::App(sf::Vector2u window_size, const sf::String& window_title, uint32_t tps,
         uint32_t fps)
    : window_(sf::RenderWindow(sf::VideoMode(window_size), window_title)),
      tps_(tps),
      fps_(fps) {
  window_.setFramerateLimit(fps_);
}

void App::Run() {
  auto previous = std::chrono::steady_clock::now();
  // Accumulator for unprocessed time.
  std::chrono::nanoseconds lag(0);
  while (window_.isOpen()) {
    auto current = std::chrono::steady_clock::now();

    std::chrono::duration elapsed = (current - previous);

    previous = current;
    lag += elapsed;
    
    PollInput();

    // Process game logic updates based on the target TPS.
    while (lag >= NanosecondsPerTick()) {
      // Update...
      lag -= NanosecondsPerTick();
    }

    window_.clear();

    // Draw...

    window_.display();
  }
}

std::chrono::duration<float> App::SecondsPerTick() const {
  using namespace std::chrono_literals;
  return std::chrono::duration<float>(1s) / tps_;  // NOLINT
}

std::chrono::nanoseconds App::NanosecondsPerTick() const {
  using namespace std::chrono_literals;
  return std::chrono::nanoseconds(1s) / tps_;  // NOLINT
}

sf::RenderWindow& App::GetWindow() {
  return window_;
}

ResourceManager& App::GetResourceManager() {
  return resource_manager_;
}

const Input& App::GetInput() const {
  return input_;
}

void App::PollInput() {
  // Prepare the input handler for new events.
  input_.Advance();

  while (std::optional event = window_.pollEvent()) {
    if (event->is<sf::Event::Closed>()) {
      window_.close();
    }

    // Pass the event to the input handler for processing.
    input_.Handle(*event);
  }
}

}  // namespace ng
```
{% endcode %}
