# Resource Management

## Texture Loading

Primitive shapes are cool, but textures are cooler!&#x20;

The `Player` needs a texture, so the engine must provide a way to load a texture in an easy and efficient way. The `ResourceManager` will be the provider of the external resources used through the game.

{% code title="engine/resource_manager.h" %}
```cpp
#pragma once

#include <SFML/Graphics/Texture.hpp>
#include <filesystem>
#include <string>
#include <string_view>
#include <unordered_map>

namespace ng {

/// @brief Manages the loading and caching of game resources such as textures, sound buffers, and fonts.
///        Ensures that resources are loaded only once and provides access to them.
class ResourceManager {
 public:
  ResourceManager() = default;
  ~ResourceManager() = default;

  ResourceManager(const ResourceManager& other) = delete;
  ResourceManager& operator=(const ResourceManager& other) = delete;
  ResourceManager(ResourceManager&& other) = delete;
  ResourceManager& operator=(ResourceManager&& other) = delete;

  /// @brief Loads a texture from the specified file path. If the texture is already loaded, returns the cached instance.
  /// @param filename The relative path to the texture file.
  /// @return A reference to the loaded SFML Texture. Lifetime is bound to the resource manager instance.
  sf::Texture& LoadTexture(const std::filesystem::path& filename);

 private:
  /// @brief The prefix for all resource file paths.
  static constexpr std::string_view kPrefix_ = "resources/";

  /// @brief Cache for loaded textures, mapping file paths to SFML Textures.
  std::unordered_map<std::filesystem::path, sf::Texture> textures_;
};

}  // namespace ng
```
{% endcode %}

{% code title="engine/resource_manager.cc" %}
```cpp
#include "resource_manager.h"

#include <SFML/Graphics/Texture.hpp>
#include <filesystem>

namespace ng {

sf::Texture& ResourceManager::LoadTexture(
    const std::filesystem::path& filename) {
  // Construct the absolute path to the texture file by combining the prefix and the given filename.
  // The absolute path is used as key to avoid mapping the same resource to different relative paths.
  std::filesystem::path full_path =
      std::filesystem::absolute(kPrefix_ / filename);

  // Attempt to find the texture in the cache.
  auto it = textures_.find(full_path);
  // If the texture is not found in the cache...
  if (it == textures_.end()) {
    // Insert the loaded texture into the cache, using the path as the key.
    it = textures_.insert({full_path, sf::Texture(full_path)}).first;
  }

  // Return a reference to the loaded texture.
  return it->second;
}

}  // namespace ng
```
{% endcode %}

## Copy Resources Directory

First, let's make a new directory inside the `game`, called `resources`.

This directory will contain all the game assets, textures, sound effects, fonts, etc...

As this guide progresses and requires specific assets, it will be your responsibility to add them to your project. The guide itself will not explicitly prompt you to do so; instead, you will encounter errors when attempting to access files that are missing from your resources folder. You have two options for obtaining these assets: either download the complete resources directory from the [main repository](https://github.com/Penca53/jp-engine/tree/main/game/resources) (recommended), or download the assets for each chapter individually from the [guide's repository](https://github.com/Penca53/jp-engine-guide).

```
jp-engine/
└── game/
    └── resources/
        ├── texture.png
        ├── sound_effect.wav
        └── my_font.ttf

```

The paths in the `ResourceManager` are relative to the `resources` directory, but the relative path of the application "begins" at the current working directory. In our case, the current working directory is where the binary resides.

The built binary is located in a completely different directory `jp-engine/build/game/` , so the resources directory needs to be copied from the `game` directory to the `build` directory. The following CMake addition does that operation.

{% code title="game/CMakeLists.txt" %}
```cmake
# -----

# --- Add: Copy the resources/ directory to the built binary directory ---
add_custom_target(copy_resources
    # Define a custom target named 'copy_resources'.
    COMMAND ${CMAKE_COMMAND} -E copy_directory_if_different
        # Use CMake's 'copy_directory_if_different' command.
        # This command copies a directory only if the source and destination are different.
        "${CMAKE_CURRENT_SOURCE_DIR}/resources"
        # Specify the source directory.
        "$<TARGET_FILE_DIR:game>/resources"
        # Specify the destination directory. "$<TARGET_FILE_DIR:game>" works for multi-target builds too.
)

# Make the 'game' target depend on the 'copy_resources' target.
# This ensures that 'copy_resources' is executed before 'game' is built.
add_dependencies(game copy_resources)
# --- End ---
```
{% endcode %}

The `game` target depends on `copy_resources` so that the copy command is executed (if there are changes in the `resources` directory) every time the `game` target is built.

Note: if a newly added file is not copied over after the build process, you may try to reconfigure the CMake project, or, more drastically, delete the `build` directory.
