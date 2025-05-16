# Camera

## Observers of the World

A `Camera` serves to render the Nodes within a scene from a specific viewpoint. Additionally, it employs Layers to selectively render content based on a Node's assigned layer and the Camera's configured rendering layers. Internally, the Camera class utilizes an `sf::View` to define this alternative perspective of the game world. For proper operation, each Camera instance automatically registers itself with the scene's `CameraManager`.

{% code title="engine/camera.h" %}
```cpp
#pragma once

#include <SFML/Graphics/View.hpp>
#include <SFML/System/Vector2.hpp>
#include <cstdint>

#include "layer.h"
#include "node.h"

namespace ng {

/// @brief Represents a camera in the game world, defining the viewport and visible layers.
class Camera : public Node {
 public:
  /// @brief Constructs a Camera with a default draw order of 0 and rendering the default layer.
  /// @param app A pointer to the App instance this camera belongs to. This pointer must not be null.
  explicit Camera(App* app);

  /// @brief Constructs a Camera with a specified draw order and rendering layers.
  /// @param app A pointer to the App instance this camera belongs to. This pointer must not be null.
  /// @param draw_order The order in which this camera should be processed for drawing (lower values are processed first).
  /// @param layers A bitmask of Layer flags indicating which layers this camera should render.
  Camera(App* app, int32_t draw_order, Layer layers);

  /// @brief Returns the SFML View associated with this camera.
  /// @return A constant reference to the SFML View.
  [[nodiscard]] const sf::View& GetView() const;

  /// @brief Returns the draw order of this camera.
  /// @return The draw order value.
  [[nodiscard]] int32_t GetDrawOrder() const;

  /// @brief Returns the bitmask of layers that this camera renders.
  /// @return The Layer flags for rendering.
  [[nodiscard]] Layer GetRenderLayers() const;

  /// @brief Sets the size of the camera's view.
  /// @param size The new size of the view.
  void SetViewSize(sf::Vector2f size);

 protected:
  void OnAdd() override;
  void Update() override;
  void OnDestroy() override;

 private:
  // The SFML View representing the camera's viewport.
  sf::View view_;
  // The order in which this camera is processed for drawing.
  int32_t draw_order_ = 0;
  // Bitmask of layers this camera should render.
  Layer render_layers_ = Layer::kDefault;
};

}  // namespace ng
```
{% endcode %}

{% code title="engine/camera.cc" %}
```cpp
#include "camera.h"

#include <SFML/Graphics/View.hpp>
#include <SFML/System/Vector2.hpp>
#include <cstdint>

#include "app.h"
#include "layer.h"
#include "node.h"

namespace ng {

Camera::Camera(App* app) : Camera(app, 0, Layer::kDefault) {}

Camera::Camera(App* app, int32_t draw_order, Layer render_layers)
    : Node(app), draw_order_(draw_order), render_layers_(render_layers) {}

const sf::View& Camera::GetView() const {
  return view_;
}

int32_t Camera::GetDrawOrder() const {
  return draw_order_;
}

Layer Camera::GetRenderLayers() const {
  return render_layers_;
}

void Camera::SetViewSize(sf::Vector2f size) {
  view_.setSize(size);
}

void Camera::OnAdd() {
  SetViewSize(sf::Vector2f(GetApp()->GetWindow().getSize()));
  view_.setCenter(GetGlobalTransform().getPosition());
  GetScene()->GetCameraManager().AddCamera(this);
}

void Camera::Update() {
  view_.setCenter(GetGlobalTransform().getPosition());
}

void Camera::OnDestroy() {
  GetScene()->GetCameraManager().RemoveCamera(this);
}

}  // namespace ng
```
{% endcode %}

A Node has a rendering layer that determines whether it is visible by a camera or not. While usually a Node is on one layer, the system allows for Nodes to exist on multiple rendering layers simultaneously.

{% code title="engine/node.h" %}
```cpp
// -----
#include "layer.h" // Add: Include Layer.

namespace ng {

class Camera; // Add: forward declare Camera.

class Node {
 public:
  // -----
 
  // --- Add: Add getter and setter for layer. ---
  /// @brief Returns the rendering layer of this node.
  /// @return The Layer this node belongs to.
  [[nodiscard]] Layer GetLayer() const;

  /// @brief Sets the rendering layer of this node.
  /// @param layer The new Layer for this node.
  void SetLayer(Layer layer);
  // --- End ---
  
 private:
  // -----
  // --- Change: Add new camera parameter. ---
  /// @brief Internal method called during the draw phase. Draws the node and its children if they belong to the camera's render layers.
  /// @param camera The Camera used for rendering.
  /// @param target The SFML RenderTarget to draw to.
  void InternalDraw(const Camera& camera, sf::RenderTarget& target);
  // --- End ---

  // -----
  // --- Add: Rendering layer. ---
  // The rendering layer of this node.
  Layer layer_ = Layer::kDefault;
  // --- End ---
};

}  // namespace ng
```
{% endcode %}

{% code title="engine/node.cc" %}
```cpp
#include "layer.h" // Add: Include Layer.

namespace ng {

// -----
// --- Add: Getter and setter. ---
Layer Node::GetLayer() const {
  return layer_;
}

void Node::SetLayer(Layer layer) {
  layer_ = layer;
}
// --- End ---

// -----

// --- Change: Draw the node only if the layers intersect. ---
void Node::InternalDraw(const Camera& camera, sf::RenderTarget& target) {
  if ((std::to_underlying(layer_) &
       std::to_underlying(camera.GetRenderLayers())) == 0) {
    return;
  }

  Draw(target);
  for (auto& child : children_) {
    child->InternalDraw(camera, target);
  }
}
// --- End ---
// -----

} //  namespace ng
```
{% endcode %}

The `InternalDraw` function now implements a visibility check based on layers. It uses a bitmask operation to compare the current Node's layer against the camera's specified rendering layers. A non-zero result from this operation indicates a match, meaning the Node should be drawn. If the bitmask operation yields zero, **the Node and all its descendants** will be skipped during the rendering process for this camera.

<pre class="language-cpp" data-title="game/score_manager.cc"><code class="lang-cpp"><strong>// -----
</strong><strong>#include "engine/layer.h" // Add: Include Layer.
</strong>
namespace game {

ScoreManager::ScoreManager(ng::App* app)
    : ng::Node(app),
      score_text_(
          GetApp()->GetResourceManager().LoadFont("Roboto-Regular.ttf")) {
  // -----
  SetLayer(ng::Layer::kUI); // Add: Set rendering layer.
}

void ScoreManager::Update() {
  // --- Add: Center the text to the window. Done in Update to adapt to window resizes. ---
  float width = static_cast&#x3C;float>(GetApp()->GetWindow().getSize().y);
  SetLocalPosition({0, -((width / 2.F) - 8)});
  // --- End ---
}

}  // namespace game
</code></pre>

By setting the UI layer on the ScoreManager, the text will be drawn only by the cameras that include the UI layer.
