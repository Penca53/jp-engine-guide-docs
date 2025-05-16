# Scene

## Graph Keeper

Think of a `Scene` as a **container** or a **level** within your game. It holds all the objects, characters, environments, cameras, and UI elements needed for a specific part of the game.

To enable runtime validation of node existence, all nodes within the scene are registered in a set. This allows the engine users to reliably check if a particular node is still valid during gameplay. This is particularly useful in scenarios where a node might be destroyed at any time, and other nodes depend on its presence to function correctly.

{% code title="engine/scene.h" %}
```cpp
#pragma once

#include <SFML/Graphics/RenderTarget.hpp>
#include <memory>
#include <string>
#include <unordered_set>

#include "derived.h"
#include "node.h"

namespace ng {

class App;

/// @brief Represents a self-contained part of the game world managing nodes.
class Scene {
 public:
  // App needs to be able to call InternalOnAdd, InternalUpdate,
  // InternalDraw, and InternalOnDestroy.
  friend class App;
  // Node needs to be able to call RegisterNode, and UnregisterNode.
  friend class Node;

  /// @brief Constructs a Scene associated with a specific App instance.
  /// @param app A pointer to the App instance this scene belongs to. This pointer must not be null.
  explicit Scene(App* app);

  /// @brief Returns the name of the scene.
  /// @return A constant reference to the scene's name.
  [[nodiscard]] const std::string& GetName() const;

  /// @brief Sets the name of the scene.
  /// @param name The new name for the scene.
  void SetName(std::string name);

  /// @brief Adds a new child node to the root of the scene. Ownership of the node is transferred to the scene.
  /// @param new_child A unique pointer to the Node to be added. This pointer must not be null.
  void AddChild(std::unique_ptr<Node> new_child);

  /// @brief Creates and adds a new child node of the specified type to the root of the scene.
  /// @tparam T The type of the Node to create, must derive from Node.
  /// @tparam Args The constructor arguments for the Node type T.
  /// @param args The arguments to forward to the constructor of T.
  /// @return A reference to the newly created and added Node.
  template <Derived<Node> T, typename... Args>
  T& MakeChild(Args&&... args) {
    return root_->MakeChild<T>(std::forward<Args>(args)...);
  }

  /// @brief Checks if a given Node pointer is currently registered within this scene.
  /// @param node A pointer to the Node to check. Can be null (no-op).
  /// @return True if the Node is valid and belongs to this scene, false otherwise.
  bool IsValid(const Node* node) const;

 private:
  /// @brief Internal method called when the scene is added to the App. Notifies the root node.
  void InternalOnAdd();
  /// @brief Internal method called during the game loop to update the scene's logic. Updates the root node.
  void InternalUpdate();
  /// @brief Internal method called during the game loop to draw the scene. Draws the root node.
  /// @param target The SFML RenderTarget to draw to.
  void InternalDraw(sf::RenderTarget& target);
  /// @brief Internal method called when the scene is about to be destroyed or unloaded. Notifies the root node.
  void InternalOnDestroy();

  /// @brief Registers a Node with the scene, allowing for quick validation. Called by Node during its addition to the hierarchy.
  /// @param node A pointer to the Node being registered. This pointer must not be null.
  void RegisterNode(const Node* node);
  /// @brief Unregisters a Node from the scene. Called by Node during its removal from the hierarchy.
  /// @param node A pointer to the Node being unregistered. This pointer must not be null.
  void UnregisterNode(const Node* node);

  // The name of the scene.
  std::string name_;

  // A set containing all Nodes currently registered in the scene for fast lookup.
  std::unordered_set<const Node*> scene_nodes_;

  // The root node of the scene graph. All entities in the scene are descendants of this node.
  // Ownership is managed by the Scene. This pointer is never null after construction.
  std::unique_ptr<Node> root_;
};

}  // namespace ng
```
{% endcode %}

{% code title="engine/scene.cc" %}
```cpp
#include "scene.h"

#include <SFML/Graphics/RenderTarget.hpp>
#include <cassert>
#include <memory>
#include <string>
#include <utility>

#include "app.h"
#include "node.h"

namespace ng {

Scene::Scene(App* app) : root_(std::make_unique<Node>(app)) {
  assert(app);
  root_->SetName("SceneRoot");
}

const std::string& Scene::GetName() const {
  return name_;
}

void Scene::SetName(std::string name) {
  name_ = std::move(name);
}

void Scene::AddChild(std::unique_ptr<Node> new_child) {
  root_->AddChild(std::move(new_child));
}

bool Scene::IsValid(const Node* node) const {
  return scene_nodes_.contains(node);
}

void Scene::InternalOnAdd() {
  root_->InternalOnAdd(this);
}

void Scene::InternalUpdate() {
  root_->InternalUpdate();
}

void Scene::InternalDraw(sf::RenderTarget& target) {
  root_->InternalDraw(target);
}

void Scene::InternalOnDestroy() {
  root_->InternalOnDestroy();
}

void Scene::RegisterNode(const Node* node) {
  assert(node);
  scene_nodes_.insert(node);
}

void Scene::UnregisterNode(const Node* node) {
  assert(node);
  scene_nodes_.erase(node);
}

}  // namespace ng

```
{% endcode %}

Notice how most of the logic of the scene consists of relaying most of the calls to the internally managed root Node.

## App and Scene Integration

The App manages a single active scene at any given time, orchestrating the calls to its `Update` and `Draw` methods according to the established Game Loop rhythm. Similar to how Nodes are handled, scene loading and unloading operations are scheduled to occur during the next game tick.

{% code title="engine/app.h" %}
```cpp
#include <memory> // Add: Include for unique_ptr.

#include "scene.h" // Add: Include for Scene.

class App {
public:
  // -----
  // --- Add: Load / Unload active scene. ---
  /// @brief Loads a new scene, replacing the currently active one. The old scene (if any) will be unloaded in the next frame.
  /// @param scene A unique pointer to the new Scene to load. Ownership is transferred to the App. This pointer must not be null.
  /// @return A reference to the App instance for method chaining.
  App& LoadScene(std::unique_ptr<Scene> scene);

  /// @brief Unloads the currently active scene. The unloading process will happen at the beginning of the next frame.
  void UnloadScene();
  // --- End ---
  // -----


private:
  // -----
  // --- Add: Own the active game scene. ---
  // The currently active game scene. Can be null if no scene is loaded. Ownership is managed by the App.
  std::unique_ptr<Scene> scene_;
  // A scene scheduled to be loaded in the next frame. Ownership is managed by the App. Can be null.
  std::unique_ptr<Scene> scheduled_scene_to_load_;
  // Flag indicating if the current scene is scheduled for unloading.
  bool is_scene_unloading_scheduled_ = false;
  // --- End ---
};
```
{% endcode %}

```cpp
// -----
// --- Add: Includes. ---
#include <memory>
#include <utility>
// --- End ---

namespace ng {

// -----

void App::Run() {
  // -----
  while (window_.isOpen()) {
    auto current = std::chrono::steady_clock::now();

    std::chrono::duration elapsed = (current - previous);

    previous = current;
    lag += elapsed;

    // --- Add: Unload / Load pending scenes.  ---
    if (is_scene_unloading_scheduled_) {
      scene_->InternalOnDestroy();
      scene_ = nullptr;
      is_scene_unloading_scheduled_ = false;
    }

    if (scheduled_scene_to_load_) {
      scene_ = std::move(scheduled_scene_to_load_);
      scene_->InternalOnAdd();
      scheduled_scene_to_load_ = nullptr;
    }
    // --- End ---

    PollInput();

    // Process game logic updates based on the target TPS.
    while (lag >= NanosecondsPerTick()) {
      // --- Add: Update the whole scene. ---
      if (scene_) {
        scene_->InternalUpdate();
      }
      // --- End ---
      lag -= NanosecondsPerTick();
    }

    window_.clear();

    // --- Add: Draw the whole scene. ---
    if (scene_) {
      scene_->InternalDraw(window_);
    }
    // --- End ---

    window_.display();
  }
}

// -----
// --- Add: Load / Unload active scene. ---
App& App::LoadScene(std::unique_ptr<Scene> scene) {
  scheduled_scene_to_load_ = std::move(scene);
  return *this;
}

void App::UnloadScene() {
  is_scene_unloading_scheduled_ = true;
}
// --- End ---

}  // namespace ng
```
