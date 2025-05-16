# Layer

## Grouping Nodes&#x20;

If you are curious, you can try to add the `ScoreManager` to the current scene and run the game, you might observe an unexpected behavior. The score text doesn't remain fixed on the screen; instead, it acts as an in-world element, staying static in a world position as the player moves around. This occurs because we haven't specified that this UI element should behave differently from another node. By default, rendering happens in world space. However, for UI elements like the score, we require screen-space rendering. This means the text's coordinates should be interpreted relative to the screen itself, not the game world.

<figure><img src="../.gitbook/assets/Screen Recording 2025-04-24 at 18.53.08.gif" alt=""><figcaption></figcaption></figure>

Introducing the `Layer` enum is a necessary step to distinguish between standard in-world nodes and UI nodes.

{% code title="engine/layer.h" %}
```cpp
#pragma once

#include <cstdint>

namespace ng {

/// @brief Defines bitmask flags for rendering layers, allowing selective rendering of objects.
enum class Layer : uint64_t {  // NOLINT
  /// @brief The default rendering layer.
  kDefault = 1ULL << 0ULL,
  /// @brief The user interface rendering layer.
  kUI = 1ULL << 1ULL,
};

}  // namespace ng
```
{% endcode %}
