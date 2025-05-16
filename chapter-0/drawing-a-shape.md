---
layout:
  title:
    visible: true
  description:
    visible: false
  tableOfContents:
    visible: true
  outline:
    visible: true
  pagination:
    visible: true
---

# Drawing a Shape

## Green Circle Shape

To test that the whole setup phase is going smoothly, let's try to draw a <mark style="color:green;">green</mark> `sf::CircleShape` at the center of the window.

{% code title="game/main.cc" %}
```cpp
#include <SFML/Graphics/CircleShape.hpp> // Add: Include for CircleShape.
#include <SFML/Graphics/RenderWindow.hpp>
#include <SFML/System/Vector2.hpp>
#include <SFML/Window/Event.hpp>
#include <SFML/Window/VideoMode.hpp>
#include <cstdlib>
#include <optional>

int main() {
  static constexpr sf::Vector2u kWindowSize = {800U, 600U};
  auto window = sf::RenderWindow(sf::VideoMode(kWindowSize), "Platformer");
  window.setFramerateLimit(60);

  // --- Add: Create and configure a green circle. ---
  
  // Define the radius of the circle.
  static constexpr float kCircleRadius = 32.F;
  // Create a circle shape with the defined radius.
  sf::CircleShape circle(kCircleRadius);
  // Set the origin of the circle to its center.
  circle.setOrigin({kCircleRadius, kCircleRadius}); 
  // Set the position of the circle to the center of the window.
  circle.setPosition(sf::Vector2f(kWindowSize) / 2.F);
  // Set the fill color of the circle to green.
  circle.setFillColor(sf::Color::Green);
  
  // --- End ---

  while (window.isOpen()) {
    while (const std::optional event = window.pollEvent()) {
      if (event->is<sf::Event::Closed>()) {
        window.close();
      }
    }

    window.clear();
    window.draw(circle); // Add: Draw the circle to the window.
    window.display();
  }

  return EXIT_SUCCESS;
}
```
{% endcode %}

A <mark style="color:green;">green</mark> circle with a radius of 32px will appear in the center of the window. Notice that the circle's draw call is issued between the `window.clear()` and `window.display()` . While the window is open, at the beginning of each frame, the window contents are cleared, the circle is drawn, and finally, it is rendered on the screen.
