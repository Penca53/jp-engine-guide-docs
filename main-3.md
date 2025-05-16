# Main 3

## The Last Main

{% code title="game/main.cc" %}
```cpp
#include "default_scene.h"
#include "engine/app.h"

#include <SFML/System/Vector2.hpp>
#include <cstdlib>

int main() {
  static constexpr sf::Vector2u kWindowSize = {832U, 640U};
  ng::App app(kWindowSize, "Platformer", 60, 60);
  app.LoadScene(game::MakeDefaultScene(&app)).Run();
  return EXIT_SUCCESS;
}
```
{% endcode %}

This is, and will be, the main file for the rest of the project. It creates an `ng::App` object, configuring the window's size and title, as well as the desired TPS and FPS, before starting the main loop with the `Run` call.&#x20;

Clear and concise.
