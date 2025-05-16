# Input Handling

## Input Interface

Reading the input state is a common operation in any game. As such, let's define the interface that will be comfortable to use to read input key states.&#x20;

Technically, SFML already provides such functionalities through the `sf::Keyboard` namespace, but that API directly reads the keystates from the OS, even when the window is running in the background. As such, it requires input monitoring access, which on MacOS is a permission that has to be added to the application.&#x20;

In this case, such low-level capabilities are not needed, so instead we'll make our own input wrapper that works by handling the input events that originate from the managed window object. By polling the events at the beginning of the window loop, the freshness of the state of the keyboard will only depend on the latency of the events sent to the window, which is usually negligible, and the go-to option for most libraries and engines.

{% code title="engine/input.h" %}
```cpp
#pragma once

#include <SFML/Window/Keyboard.hpp>

namespace ng {

/// @brief Manages keyboard input, tracking key presses, holds, and releases.
class Input {
 public:
  /// @brief Checks if a specific key was pressed down (went from not pressed to pressed) in the current frame.
  /// @param key The SFML scancode of the key to check.
  /// @return True if the key was pressed down, false otherwise.
  [[nodiscard]] bool GetKeyDown(sf::Keyboard::Scancode key) const;

  /// @brief Checks if a specific key is currently being held down.
  /// @param key The SFML scancode of the key to check.
  /// @return True if the key is currently pressed, false otherwise.
  [[nodiscard]] bool GetKey(sf::Keyboard::Scancode key) const;

  /// @brief Checks if a specific key was released (went from pressed to not pressed) in the current frame.
  /// @param key The SFML scancode of the key to check.
  /// @return True if the key was released, false otherwise.
  [[nodiscard]] bool GetKeyUp(sf::Keyboard::Scancode key) const;
};

}  // namespace ng
```
{% endcode %}

{% code title="engine/input.cc" %}
```cpp
#include "input.h"

#include <SFML/Window/Keyboard.hpp>

namespace ng {

bool Input::GetKeyDown(sf::Keyboard::Scancode key) const {
  
}

bool Input::GetKey(sf::Keyboard::Scancode key) const {
  
}

bool Input::GetKeyUp(sf::Keyboard::Scancode key) const {
  
}

}  // namespace ng
```
{% endcode %}

* GetKeyDown: "just pressed", "jump" action
* GetKey: "pressing now", contiguous actions, such as "player movement"
* GetKeyUp: "just released", "release bow arrow" action

## Key State

Now that the interface is defined, we need to implement it.

{% code title="engine/input.h" %}
```cpp
#pragma once

#include <SFML/Window/Keyboard.hpp>
#include <bitset> // Add: Include for keeping the key states.

namespace ng {

/// @brief Manages keyboard input, tracking key presses, holds, and releases.
class Input {
 public:
  /// @brief Checks if a specific key was pressed down (went from not pressed to pressed) in the current frame.
  /// @param key The SFML scancode of the key to check.
  /// @return True if the key was pressed down, false otherwise.
  [[nodiscard]] bool GetKeyDown(sf::Keyboard::Scancode key) const;

  /// @brief Checks if a specific key is currently being held down.
  /// @param key The SFML scancode of the key to check.
  /// @return True if the key is currently pressed, false otherwise.
  [[nodiscard]] bool GetKey(sf::Keyboard::Scancode key) const;

  /// @brief Checks if a specific key was released (went from pressed to not pressed) in the current frame.
  /// @param key The SFML scancode of the key to check.
  /// @return True if the key was released, false otherwise.
  [[nodiscard]] bool GetKeyUp(sf::Keyboard::Scancode key) const;

 // --- Add: Store the current and previous key states. ---
 private:
  // Stores the current state (pressed or not pressed) of each keyboard key.
  std::bitset<sf::Keyboard::ScancodeCount> key_states_;
  // Stores the previous frame's state of each keyboard key, used for detecting key down and up events.
  std::bitset<sf::Keyboard::ScancodeCount> old_key_states_;
 // --- End ---
};

}  // namespace ng
```
{% endcode %}

{% code title="engine/input.cc" %}
```cpp
#include "input.h"

#include <SFML/Window/Keyboard.hpp>
#include <utility>  // Add: Include for std::underlying.

namespace ng {

bool Input::GetKeyDown(sf::Keyboard::Scancode key) const {
  // Checks if a key was pressed down.
  // Returns true if the key is currently pressed (key_states_ is true) and was not pressed in the previous frame (old_key_states_ is false).
  return (key_states_[std::to_underlying(key)] &&
          !old_key_states_[std::to_underlying(key)]);
}

bool Input::GetKey(sf::Keyboard::Scancode key) const {
  // Checks if a key is currently being held down.
  // Returns true if the key is currently pressed (key_states_ is true).
  return key_states_[std::to_underlying(key)];
}

bool Input::GetKeyUp(sf::Keyboard::Scancode key) const {
  // Checks if a key was released.
  // Returns true if the key is currently not pressed (key_states_ is false) and was pressed in the previous frame (old_key_states_ is true).
  return (!key_states_[std::to_underlying(key)] &&
          old_key_states_[std::to_underlying(key)]);
}

}  // namespace ng
```
{% endcode %}

## Input Event Detection

Finally, the input events need to be handled to update the current state of the keys.

{% code title="engine/input.h" %}
```cpp
#pragma once

#include <SFML/Window/Event.hpp>  // Add: Include for handling window events.
#include <SFML/Window/Keyboard.hpp>
#include <bitset>

namespace ng {

/// @brief Manages keyboard input, tracking key presses, holds, and releases.
class Input {
 public:
  // --- Add: Window event handling to retrieve and update the keyboard state. ---
  /// @brief Advances the input state, copying the current key states to the old key states for detecting key down and up events.
  void Advance();

  /// @brief Handles SFML window events, specifically KeyPressed and KeyReleased events, to update the current key states.
  /// @param event The SFML event to process.
  void Handle(const sf::Event& event);
  // --- End ---
  
  // -----
};

}  // namespace ng
```
{% endcode %}

{% code title="engine/input.cc" %}
```cpp
#include "input.h"

#include <SFML/Window/Event.hpp> // Add: Include for handling window events.
#include <SFML/Window/Keyboard.hpp>
#include <utility>

namespace ng {

// --- Add: Window event handling to retrieve and update the keyboard state. ---
void Input::Advance() {
  // Copies the current key states to the old key states. This is crucial for detecting key down and key up events.
  // By storing the previous frame's states, we can compare them to the current states.
  old_key_states_ = key_states_;
}

void Input::Handle(const sf::Event& event) {
  // Checks if the event is a KeyPressed event.
  if (const auto* pressed = event.getIf<sf::Event::KeyPressed>()) {
    // If it's a KeyPressed event, set the corresponding bit in key_states_ to true.
    // std::to_underlying converts the Scancode enum to its underlying integer type, which is used as an index for the bitset.
    key_states_[std::to_underlying(pressed->scancode)] = true;
  } else if (const auto* released = event.getIf<sf::Event::KeyReleased>()) {
    // Checks if the event is a KeyReleased event.
    // If it's a KeyReleased event, set the corresponding bit in key_states_ to false.
    key_states_[std::to_underlying(released->scancode)] = false;
  }
}
// --- End ---

// -----

}  // namespace ng
```
{% endcode %}
