# Resource Manager (Fonts)

## Font Loading

The `ScoreManager` needs to access fonts from the `resources` to draw UI text on the screen.

Adding support to `sf::Font` in the `ResourceManager` is an easy task.

{% code title="engine/resource_manager.h" %}
```cpp
// -----
#include <SFML/Graphics/Font.hpp> // Add: Include font.

class ResourceManager {
public:
  // -----
  
  // --- Add: Load sound buffer ---
  /// @brief Loads a font from the specified file path. If the font is already loaded, returns the cached instance.
  /// @param filename The relative path to the font file.
  /// @return A reference to the loaded SFML Font. Lifetime is bound to the resource manager instance.
  sf::Font& LoadFont(const std::filesystem::path& filename);
  // --- End --- 
  
private:
  // -----  
  // --- Add: Cached fonts. ---
  /// @brief Cache for loaded fonts, mapping file paths to SFML Fonts.
  std::unordered_map<std::filesystem::path, sf::Font> fonts_;
  // --- End ---
};
```
{% endcode %}

{% code title="engine/resource_manager.cc" %}
```cpp
#include <SFML/Graphics/Font.hpp> // Add: Include font

namespace ng {

// -----

// --- Add: Implement font loading. ---
sf::Font& ResourceManager::LoadFont(const std::filesystem::path& filename) {
  std::filesystem::path full_path =
      std::filesystem::absolute(kPrefix_ / filename);

  auto it = fonts_.find(full_path);
  if (it == fonts_.end()) {
    it = fonts_.insert({full_path, sf::Font(full_path)}).first;
  }

  return it->second;
}
// --- End ---

}  // namespace ng
```
{% endcode %}
