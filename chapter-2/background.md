# Background

## Infinite Vertical Scroll

The blank background is boring. A texture that infinitely scrolls may add some color and dynamism to the overall feel of the game, after all, we don't want to fall asleep.

We already know how to render a texture on the screen, we use a `sf::Sprite`. And we also know how to move it, we can just change the y position of the sprite. But what about the _infinite_ part?

Well, we could make a really big sprite, and hope the player never leaves the game open for more than a few hours. Otherwise, we can be smart about it and we can keep the object still, but move the texture UV mapping, and we already know how to create a quad mesh from the Tilemap!

The background's size needs to cover the whole level, so it will be quite large. An option is to use a texture with the same size as the map, but that is not very flexible for future changes. Instead, we'll enable the `repeated` property of a texture that, instead of stretching the image, infinitely repeats the same texture and allows UVs to go beyond the usual \[0, 1] range.

{% code title="game/background.h" %}
```cpp
#pragma once

#include <SFML/Graphics/Texture.hpp>
#include <SFML/Graphics/VertexArray.hpp>
#include <SFML/System/Vector2.hpp>
#include <cstdint>

#include "engine/app.h"

namespace game {

class Background {
 public:
  Background(ng::App* app, sf::Vector2u size);

  void Update();
  void Draw(sf::RenderTarget& target);

 private:
  ng::App* app_ = nullptr;
  sf::Vector2u size_;
  sf::Texture* texture_ = nullptr;
  sf::VertexArray image_vertices_;
  int32_t t_ = 0;
};

}  // namespace game
```
{% endcode %}

{% code title="game/background.cc" %}
```cpp
#include "background.h"

#include <SFML/Graphics/PrimitiveType.hpp>
#include <SFML/Graphics/RenderStates.hpp>
#include <SFML/Graphics/RenderTarget.hpp>
#include <SFML/System/Vector2.hpp>
#include <cassert>
#include <cstddef>
#include <cstdint>

#include "engine/app.h"

namespace game {

// Number of vertices in a triangle.
static constexpr size_t kTriangleVertexCount = 3;
// Number of vertices in a quad (two triangles).
static constexpr size_t kTrisInQuad = 2 * kTriangleVertexCount;

Background::Background(ng::App* app, sf::Vector2u size)
    : app_(app),
      size_(size),
      texture_(&app_->GetResourceManager().LoadTexture("Gray.png")),
      image_vertices_(sf::PrimitiveType::Triangles, kTrisInQuad) {
  assert(app);

  // Enable texture repeating. This is required to make the texture fill the whole mesh.
  texture_->setRepeated(true);

  sf::Vector2f fsize(size_);
  // Set vertex positions to form a quad covering the background.
  image_vertices_[0].position = sf::Vector2f(0, 0);
  image_vertices_[1].position = sf::Vector2f(fsize.x, 0);
  image_vertices_[2].position = sf::Vector2f(0, fsize.y);
  image_vertices_[3].position = sf::Vector2f(0, fsize.y);
  image_vertices_[4].position = sf::Vector2f(fsize.x, 0);
  image_vertices_[5].position = sf::Vector2f(fsize.x, fsize.y);
}

void Background::Update() {
  static constexpr int32_t kScrollTicksPerPixel = 4;
  float offset = -static_cast<float>(t_) / kScrollTicksPerPixel;

  sf::Vector2f fsize(size_);
  // Set texture coordinates to create a scrolling effect.
  image_vertices_[0].texCoords = sf::Vector2f(0, 0 + offset);
  image_vertices_[1].texCoords = sf::Vector2f(fsize.x, 0 + offset);
  image_vertices_[2].texCoords = sf::Vector2f(0, fsize.y + offset);
  image_vertices_[3].texCoords = sf::Vector2f(0, fsize.y + offset);
  image_vertices_[4].texCoords = sf::Vector2f(fsize.x, 0 + offset);
  image_vertices_[5].texCoords = sf::Vector2f(fsize.x, fsize.y + offset);

  // Increment time counter and wrap around.
  // Technically, wrapping is not required since the texture is repeated, but
  // not wrapping it would lead to potential overflows.
  t_ = (t_ + 1) %
       (static_cast<int32_t>(texture_->getSize().y) * kScrollTicksPerPixel);
}

void Background::Draw(sf::RenderTarget& target) {
  sf::RenderStates state;
  // Set the texture for rendering.
  state.texture = texture_;
  target.draw(image_vertices_, state);
}

}  // namespace game
```
{% endcode %}
