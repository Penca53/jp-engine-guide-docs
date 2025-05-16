# Resource Manager (Sounds)

## Sound Loading

The `Player` needs to access sound effects from the `resources` when jumping.

Adding support to `sf::SoundBuffers` in the `ResourceManager` should take a few minutes.

{% code title="engine/resource_manager.h" %}
```cpp
// -----
#include <SFML/Audio/SoundBuffer.hpp> // Add: Include sound buffers.

class ResourceManager {
public:
  // -----
  
  // --- Add: Load sound buffer. ---
  /// @brief Loads a sound buffer from the specified file path. If the sound buffer is already loaded, returns the cached instance.
  /// @param filename The relative path to the sound buffer file.
  /// @return A reference to the loaded SFML SoundBuffer. Lifetime is bound to the resource manager instance.
  sf::SoundBuffer& LoadSoundBuffer(const std::filesystem::path& filename);
  // --- End --- 
  
private:
  // -----  
  // --- Add: Cached sound buffers. ---
  /// @brief Cache for loaded sound buffers, mapping file paths to SFML SoundBuffers.
  std::unordered_map<std::filesystem::path, sf::SoundBuffer> sound_buffers_;
  // --- End ---
};
```
{% endcode %}

{% code title="engine/resource_manager.cc" %}
```cpp
#include <SFML/Audio/SoundBuffer.hpp> // Add: Include sound buffers.

namespace ng {

// -----

// --- Add: Implement sound buffer loading. ---
sf::SoundBuffer& ResourceManager::LoadSoundBuffer(
    const std::filesystem::path& filename) {
  std::filesystem::path full_path =
      std::filesystem::absolute(kPrefix_ / filename);

  auto it = sound_buffers_.find(full_path);
  if (it == sound_buffers_.end()) {
    it = sound_buffers_.insert({full_path, sf::SoundBuffer(full_path)}).first;
  }

  return it->second;
}
// --- End ---

}  // namespace ng
```
{% endcode %}
