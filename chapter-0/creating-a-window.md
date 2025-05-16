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

# Creating a Window

## Blank Window

To display something on the screen, we need a surface to draw on. A "window" is an object managed by the OS that lets us draw pixels onto it. SFML exposes it through `sf::RenderWindow` and that's exactly what we will use.

{% code title="game/main.cc" %}
```cpp
#include <SFML/Graphics/RenderWindow.hpp>
#include <SFML/System/Vector2.hpp>
#include <SFML/Window/Event.hpp>
#include <SFML/Window/VideoMode.hpp>
#include <cstdlib>
#include <optional>

int main() {
  // Define the window size.
  // A vector has multiple meanings in different fields, but in this case it is simply
  // used to represent the width (x component) and height (y component) of the window.
  static constexpr sf::Vector2u kWindowSize = {800U, 600U};

  // Create a render window.
  auto window = sf::RenderWindow(sf::VideoMode(kWindowSize), "Platformer");

  // Set the frame rate limit (60FPS).
  window.setFramerateLimit(60);

  // Main loop.
  while (window.isOpen()) {
    // Event polling loop.
    while (const std::optional event = window.pollEvent()) {
      // Check for close event.
      if (event->is<sf::Event::Closed>()) {
        window.close();
      }
    }

    // Clear the window.
    window.clear();
    
    // Drawing stuff will go here...

    // Display the contents of the window.
    window.display();
  }

  // Return success.
  return EXIT_SUCCESS;
}
```
{% endcode %}

You'll see a new window pop up on your screen. This window is titled "Platformer" and is empty (it will be a blank, solid color, likely black or white depending on your system's defaults), with a size of 800x600 pixels.

The code sets a frame rate limit of 60 frames per second. This means the program tries to update and redraw the window 60 times every second, making it appear smooth.

The program will keep running, and the window will stay open until you close it. Once you close it, the program will successfully end.
