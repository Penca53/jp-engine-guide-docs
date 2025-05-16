# Camera Manager

## Camera Keeper

The engine requires knowledge of all Cameras present in the scene to perform its rendering process correctly. Therefore, all Cameras are registered with the CameraManager, unlike standard Nodes. The CameraManager uses the draw order of each camera (where lower values are drawn first) to iterate through them.&#x20;

{% code title="engine/camera_manager.h" %}
```cpp
#pragma once

#include <set>

#include "camera.h"

namespace ng {

/// @brief Manages a collection of cameras within a scene, handling resizing and providing access to them in draw order.
class CameraManager {
  // Camera needs to be able to call AddCamera, and RemoveCamera.
  friend class Camera;

 public:
  /// @brief Comparator class used to sort cameras based on their draw order.
  class DrawOrderCompare {
   public:
    /// @brief Compares two Camera pointers based on their draw order.
    /// @param a A pointer to the first Camera.
    /// @param b A pointer to the second Camera.
    /// @return True if camera 'a' should be drawn before camera 'b', false otherwise.
    bool operator()(const Camera* a, const Camera* b) const {
      return a->GetDrawOrder() < b->GetDrawOrder();
    };
  };

  /// @brief Called when the game window is resized. Notifies all managed cameras to update their view size.
  /// @param size The new size of the window.
  void OnWindowResize(sf::Vector2f size);

  /// @brief Returns a constant reference to the set of managed cameras, sorted by their draw order.
  /// @return A constant reference to the set of Camera pointers.
  [[nodiscard]] const std::set<Camera*, DrawOrderCompare>& GetCameras() const;

 private:
  /// @brief Adds a camera to the managed set. Called by Camera during its addition to a scene.
  /// @param camera A pointer to the Camera to add. This pointer must not be null and the Camera's lifetime should be managed externally to this class.
  void AddCamera(Camera* camera);

  /// @brief Removes a camera from the managed set. Called by Camera during its removal from a scene.
  /// @param camera A pointer to the Camera to remove. This pointer must not be null and the Camera's lifetime should be managed externally to this class.
  void RemoveCamera(Camera* camera);

  // A set containing pointers to all cameras managed by this manager, sorted using DrawOrderCompare.
  std::set<Camera*, DrawOrderCompare> cameras_;
};

}  // namespace ng
```
{% endcode %}

{% code title="engine/camera_manager.cc" %}
```cpp
#include "camera_manager.h"

#include <SFML/System/Vector2.hpp>
#include <cassert>
#include <set>

#include "camera.h"

namespace ng {

void CameraManager::OnWindowResize(sf::Vector2f size) {
  for (Camera* camera : cameras_) {
    camera->SetViewSize(size);
  }
}

const std::set<Camera*, CameraManager::DrawOrderCompare>&
CameraManager::GetCameras() const {
  return cameras_;
}

void CameraManager::AddCamera(Camera* camera) {
  assert(camera);
  cameras_.insert(camera);
}

void CameraManager::RemoveCamera(Camera* camera) {
  assert(camera);
  cameras_.erase(camera);
}

}  // namespace ng
```
{% endcode %}

For each camera, the engine sets the rendering target's view to that camera's perspective and then draws the entire scene. During this draw pass, the current camera acts as the reference point for determining Node visibility based on its rendering layers.

{% code title="engine/scene.h" %}
```cpp
// -----
#include "camera_manager.h" // Add: Include.

class Scene {
public:
  // -----
  // --- Add: Getter. ---
  /// @brief Returns a reference to the CameraManager, which handles multiple cameras within the scene.
  /// @return A reference to the CameraManager.
  [[nodiscard]] CameraManager& GetCameraManager();
  // --- End ---
private:
  // -----
  // Manages the cameras within the scene.
  CameraManager camera_manager_; // Add: Create a CameraManager.
  // -----
};
```
{% endcode %}

{% code title="engine/scene.cc" %}
```cpp
// -----
// --- Add: Includes. ---
#include "camera_manager.h" 
#include "layer.h"
// --- End ---

namespace ng {

Scene::Scene(App* app) : root_(std::make_unique<Node>(app)) {
  assert(app);
  root_->SetName("SceneRoot");
  // Render all layers by default on the root node.
  root_->SetLayer(static_cast<Layer>(~0ULL)); // Add: Set layer.
}

// -----
// --- Add: Getter. ---
CameraManager& Scene::GetCameraManager() {
  return camera_manager_;
}
// --- End ---

// -----
void Scene::InternalDraw(sf::RenderTarget& target) {
  // --- Change: Draw the scene through every camera. ---
  for (const Camera* camera : camera_manager_.GetCameras()) {
    target.setView(camera->GetView());
    root_->InternalDraw(*camera, target);
  }
  // --- End ---
}
// -----

}  // namespace ng
```
{% endcode %}
