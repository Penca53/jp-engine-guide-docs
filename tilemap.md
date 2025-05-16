# Tilemap

## Grid of Tiles

The Tilemap's task is to visually present a grid of Tiles, referencing a specified Tileset.&#x20;

Rendering the Tilemap by using several `sf::Sprites` (one per Tile) would lead to performance issues due to the excessive amount of draw calls (one per sprite).&#x20;

Instead, the Tilemap is rendered as a single mesh constructed with a `sf::VertexArray`. Each Tile within the grid is represented by a quad in the mesh, allowing the entire Tilemap to be drawn with a single rendering call, no matter the size of the Tilemap.

Although a "center origin" is generally followed for objects in this guide, the Tilemap is an exception. Its origin is at the top-left, and therefore the mesh renders from the top-left corner down-right.

{% code title="engine/tilemap.h" %}
```cpp
#pragma once

#include <SFML/Graphics/RenderTarget.hpp>
#include <SFML/Graphics/VertexArray.hpp>
#include <SFML/System/Vector2.hpp>
#include <vector>

#include "tile.h"
#include "tileset.h"

namespace ng {

/// @brief Represents a grid-based map composed of tiles from a Tileset.
class Tilemap {
 public:
  /// @brief Constructs a Tilemap with the specified size and tileset.
  /// @param size The dimensions of the tilemap in tiles (width and height).
  /// @param tileset The Tileset to use for rendering the tiles. Ownership is transferred to the Tilemap.
  Tilemap(sf::Vector2u size, Tileset tileset);

  /// @brief Returns the size of the tilemap in tiles.
  /// @return The dimensions of the tilemap as an sf::Vector2u.
  [[nodiscard]] sf::Vector2u GetSize() const;

  /// @brief Returns the size of individual tiles used by this tilemap.
  /// @return The tile dimensions as an sf::Vector2u.
  [[nodiscard]] sf::Vector2u GetTileSize() const;

  /// @brief Checks if a given tile position (in tile coordinates) is within the bounds of the tilemap.
  /// @param position The tile coordinates to check.
  /// @return True if the position is within the bounds, false otherwise.
  [[nodiscard]] bool IsWithinBounds(sf::Vector2u position) const;

  /// @brief Returns the Tile at the specified tile coordinates.
  /// @param position The tile coordinates to retrieve the tile from.
  /// @return A constant reference to the Tile at the given position. Throws std::out_of_range if the position is out of bounds.
  [[nodiscard]] const Tile& GetTile(sf::Vector2u position) const;

  /// @brief Sets the Tile at the specified tile coordinates using its TileID.
  /// @param position The tile coordinates to set the tile at.
  /// @param tile_id The ID of the tile to set.
  void SetTile(sf::Vector2u position, TileID tile_id);

  /// @brief Checks if a given world position is within the bounds of the tilemap.
  /// @param world_position The world coordinates to check.
  /// @return True if the world position corresponds to a tile within the bounds, false otherwise.
  [[nodiscard]] bool IsWithinWorldBounds(sf::Vector2f world_position) const;

  /// @brief Returns the Tile at the specified world coordinates.
  /// @param world_position The world coordinates to retrieve the tile from.
  /// @return A constant reference to the Tile at the given world position.
  [[nodiscard]] const Tile& GetWorldTile(sf::Vector2f world_position) const;

  /// @brief Sets the Tile at the specified world coordinates using its TileID.
  /// @param world_position The world coordinates to set the tile at.
  /// @param tile_id The ID of the tile to set.
  void SetWorldTile(sf::Vector2f world_position, TileID tile_id);

  /// @brief Converts world coordinates to tile coordinates.
  /// @param world_position The world coordinates to convert.
  /// @return The corresponding tile coordinates.
  [[nodiscard]] sf::Vector2u WorldToTileSpace(
      sf::Vector2f world_position) const;

  /// @brief Renders the tilemap.
  /// @param target The SFML RenderTarget to draw to.
  void Draw(sf::RenderTarget& target);

 private:
  // The dimensions of the tilemap in tiles.
  sf::Vector2u size_;
  // The tileset used by this tilemap. Ownership is held by the Tilemap.
  Tileset tileset_;
  // A vector storing the TileID for each tile in the map.
  std::vector<TileID> tiles_;
  // The vertex array used for rendering the tilemap efficiently.
  sf::VertexArray vertices_;
};

}  // namespace ng
```
{% endcode %}

{% code title="engine/tilemap.cc" %}
```cpp
#include "tilemap.h"

#include <SFML/Graphics/PrimitiveType.hpp>
#include <SFML/Graphics/RenderStates.hpp>
#include <SFML/Graphics/RenderTarget.hpp>
#include <SFML/Graphics/Vertex.hpp>
#include <SFML/System/Vector2.hpp>
#include <cstddef>
#include <cstdint>
#include <span>
#include <utility>

#include "tile.h"
#include "tileset.h"

namespace ng {

// Number of vertices per triangle.
static constexpr size_t kTriangleVertexCount = 3;
// Number of vertices per quad (two triangles).
static constexpr size_t kTrisInQuad = 2 * kTriangleVertexCount;

Tilemap::Tilemap(sf::Vector2u size, Tileset tileset)
    : size_(size),
      tileset_(std::move(tileset)),
      vertices_(sf::PrimitiveType::Triangles, static_cast<size_t>(size_.x) *
                                                  static_cast<size_t>(size_.y) *
                                                  kTrisInQuad) {
  tiles_.resize(static_cast<size_t>(size_.x) * static_cast<size_t>(size_.y));

  for (uint32_t y = 0; y < size_.y; ++y) {
    for (uint32_t x = 0; x < size_.x; ++x) {
      // Get vertex span for current tile.
      std::span<sf::Vertex> triangles =
          std::span(&vertices_[(y * size_.x + x) * kTrisInQuad], kTrisInQuad);

      sf::Vector2f tile_size = sf::Vector2f(tileset_.GetTileSize());
      auto fx = static_cast<float>(x);
      auto fy = static_cast<float>(y);

      // Set vertex positions to form a quad (two triangles).
      triangles[0].position = sf::Vector2f(fx * tile_size.x, fy * tile_size.y);
      triangles[1].position =
          sf::Vector2f((fx + 1) * tile_size.x, fy * tile_size.y);
      triangles[2].position =
          sf::Vector2f(fx * tile_size.x, (fy + 1) * tile_size.y);
      triangles[3].position =
          sf::Vector2f(fx * tile_size.x, (fy + 1) * tile_size.y);
      triangles[4].position =
          sf::Vector2f((fx + 1) * tile_size.x, fy * tile_size.y);
      triangles[5].position =
          sf::Vector2f((fx + 1) * tile_size.x, (fy + 1) * tile_size.y);

      // Set initial vertex colors to transparent.
      triangles[0].color = sf::Color::Transparent;
      triangles[1].color = sf::Color::Transparent;
      triangles[2].color = sf::Color::Transparent;
      triangles[3].color = sf::Color::Transparent;
      triangles[4].color = sf::Color::Transparent;
      triangles[5].color = sf::Color::Transparent;
    }
  }
}

sf::Vector2u Tilemap::GetSize() const {
  return size_;
}

sf::Vector2u Tilemap::GetTileSize() const {
  return tileset_.GetTileSize();
}

bool Tilemap::IsWithinBounds(sf::Vector2u position) const {
  return position.x < size_.x && position.y < size_.y;
}

const Tile& Tilemap::GetTile(sf::Vector2u position) const {
  return tileset_.GetTile(tiles_[(position.y * size_.x) + position.x]);
}

void Tilemap::SetTile(sf::Vector2u position, TileID tile_id) {
  tiles_[(position.y * size_.x) + position.x] = tile_id;

  // Get vertex span for current tile.
  std::span<sf::Vertex> triangles =
      std::span(&vertices_[(position.y * size_.x + position.x) * kTrisInQuad],
                kTrisInQuad);

  auto texture_coords = GetTile(position).GetTextureCoords();
  if (texture_coords.has_value()) {
    auto pos = sf::Vector2f(texture_coords->position);
    auto size = sf::Vector2f(texture_coords->size);

    // Set texture coordinates for each vertex.
    triangles[0].texCoords = sf::Vector2f(pos.x, pos.y);
    triangles[1].texCoords = sf::Vector2f(pos.x + size.x, pos.y);
    triangles[2].texCoords = sf::Vector2f(pos.x, pos.y + size.y);
    triangles[3].texCoords = sf::Vector2f(pos.x, pos.y + size.y);
    triangles[4].texCoords = sf::Vector2f(pos.x + size.x, pos.y);
    triangles[5].texCoords = sf::Vector2f(pos.x + size.x, pos.y + size.y);

    // Set vertex colors to white to render the texture.
    triangles[0].color = sf::Color::White;
    triangles[1].color = sf::Color::White;
    triangles[2].color = sf::Color::White;
    triangles[3].color = sf::Color::White;
    triangles[4].color = sf::Color::White;
    triangles[5].color = sf::Color::White;
  } else {
    // If no texture coordinates, set vertex colors to transparent.
    triangles[0].color = sf::Color::Transparent;
    triangles[1].color = sf::Color::Transparent;
    triangles[2].color = sf::Color::Transparent;
    triangles[3].color = sf::Color::Transparent;
    triangles[4].color = sf::Color::Transparent;
    triangles[5].color = sf::Color::Transparent;
  }
}

bool Tilemap::IsWithinWorldBounds(sf::Vector2f world_position) const {
  if (world_position.x < 0 || world_position.y < 0) {
    return false;
  }

  return IsWithinBounds(WorldToTileSpace(world_position));
}

bool Tilemap::IsWithinWorldBounds(sf::Vector2f world_position) const {
  sf::Vector2f tilemap_relative_position = world_position;

  if (tilemap_relative_position.x < 0 || tilemap_relative_position.y < 0) {
    return false;
  }

  sf::Vector2u tile_position(tilemap_relative_position.componentWiseDiv(
      sf::Vector2f(tileset_.GetTileSize())));

  return IsWithinBounds(tile_position);
}

const Tile& Tilemap::GetWorldTile(sf::Vector2f world_position) const {
  return GetTile(WorldToTileSpace(world_position));
}

void Tilemap::SetWorldTile(sf::Vector2f world_position, TileID tile_id) {
  SetTile(WorldToTileSpace(world_position), tile_id);
}

sf::Vector2u Tilemap::WorldToTileSpace(sf::Vector2f world_position) const {
  return sf::Vector2u(
      world_position.componentWiseDiv(sf::Vector2f(tileset_.GetTileSize())));
}

void Tilemap::Draw(sf::RenderTarget& target) {
  sf::RenderStates state;
  // Set the texture for rendering.
  state.texture = tileset_.GetTexture();
  target.draw(vertices_, state);
}

}  // namespace ng
```
{% endcode %}
