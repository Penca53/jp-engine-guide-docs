# Tile

## Unit of the Map

A Tile, the unit of a Tilemap, is defined by a unique ID and the data necessary for rendering its visual form. This ID is used by the engine's user to choose the appropriate Tile from the Tileset while painting the world map.

{% code title="engine/tile.h" %}
```cpp
#pragma once

#include <SFML/Graphics/Rect.hpp>
#include <cstdint>
#include <optional>

/// @brief The unique identifier for all the tiles. The user of the engine must define the enum
//         to use Tile related operations (i.e. Tilemap).
enum class TileID : uint64_t;

namespace ng {

/// @brief Represents a single tile in a tilemap, containing its ID and optional texture coordinates.
class Tile {
 public:
  /// @brief Constructs a Tile with only an ID. The texture coordinates will be empty.
  /// @param id The unique identifier for this tile.
  explicit Tile(TileID id);

  /// @brief Constructs a Tile with an ID and its corresponding texture coordinates.
  /// @param id The unique identifier for this tile.
  /// @param texture_coords The rectangular coordinates within a texture atlas for this tile.
  Tile(TileID id, sf::IntRect texture_coords);

  /// @brief Returns the unique identifier of the tile.
  /// @return The TileID of this tile.
  [[nodiscard]] TileID GetID() const;

  /// @brief Returns the optional texture coordinates of the tile within a texture atlas.
  /// @return A constant reference to an optional sf::IntRect. It will contain the texture coordinates if set, or be empty otherwise.
  [[nodiscard]] const std::optional<sf::IntRect>& GetTextureCoords() const;

 private:
  // The unique identifier of the tile.
  TileID id_{};
  // Optional texture coordinates within a texture atlas. Empty if the tile doesn't have specific texture coordinates.
  std::optional<sf::IntRect> texture_coords_;
};

}  // namespace ng
```
{% endcode %}

{% code title="engine/tile.cc" %}
```cpp
#include "tile.h"

#include <SFML/Graphics/Rect.hpp>
#include <optional>

namespace ng {

Tile::Tile(TileID id) : id_(id) {}

Tile::Tile(TileID id, sf::IntRect texture_coords)
    : id_(id), texture_coords_(texture_coords) {}

TileID Tile::GetID() const {
  return id_;
}

const std::optional<sf::IntRect>& Tile::GetTextureCoords() const {
  return texture_coords_;
}

}  // namespace ng
```
{% endcode %}

{% code title="game/tile_id.h" %}
```cpp
#pragma once

#include <cstdint>

enum class TileID : uint64_t {  // NOLINT
  kVoid = 0,
  kDirt,
};
```
{% endcode %}
