# Main 1

## A New Main

{% code title="game/main.cc" %}
```cpp
#include <SFML/Graphics/CircleShape.hpp>
#include <SFML/Graphics/RenderWindow.hpp>
#include <SFML/System/Vector2.hpp>
#include <cstdlib>
#include <optional>

#include "engine/app.h"
#include "engine/input.h"
#include "player.h"

int main() {
  static constexpr sf::Vector2u kWindowSize = {800U, 600U};
  ng::App app(kWindowSize, "Platformer", 60);
  sf::RenderWindow& window = app.GetWindow();
  ng::Input& input = app.GetInput();

  static constexpr float kCircleRadius = 32.F;
  sf::CircleShape circle(kCircleRadius);
  circle.setOrigin({kCircleRadius, kCircleRadius});
  circle.setFillColor(sf::Color::Green);

  game::Player player(&app);

  while (window.isOpen()) {
    // Advance the input one step forward.
    input.Advance();
    while (const std::optional event = window.pollEvent()) {
      if (event->is<sf::Event::Closed>()) {
        window.close();
      }

      // Handle a possible input event to update the current input state.
      input.Handle(*event);
    }

    player.Update();

    window.clear();

    // What is drawn first goes "behind" what is drawn later.
    // In this case, the circle goes behind the player.
    window.draw(circle);
    player.Draw(window);

    window.display();
  }

  return EXIT_SUCCESS;
}
```
{% endcode %}

Executing this setup should produce an 800x600px window. Keyboard input will allow you to move the player sprite. The viewport will track the player's position, ensuring the player remains visible, which may result in the <mark style="color:green;">green</mark> circle moving outside the viewport's bounds as the player moves around.

Observe that neither the circle nor the player had their positions explicitly set, yet they appear centered in the window. Even though their default position is (0,0), the view-following system keeps the player locked at the center of the view, effectively placing both initial elements in the "middle of the window".
