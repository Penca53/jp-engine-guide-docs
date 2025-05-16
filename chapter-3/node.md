# Node

## Unit of the Graph

In a game engine's scene graph, a `Node` is the fundamental building block used to organize and represent everything within the game's world.&#x20;

Every `Node` maintains ownership of its child Nodes (if any) and, unless they are root Node, always has a parent Node, realizing a tree-like structure.

### Node Basics

<pre class="language-cpp"><code class="lang-cpp"><strong>#pragma once
</strong>
#include &#x3C;cstddef>
#include &#x3C;memory>
#include &#x3C;string>
#include &#x3C;vector>

#include "derived.h"

namespace ng {

class App;

/// @brief The base class for all entities in the game world, forming a scene graph.
///        Manages local and global transformations, parent-child relationships, and rendering layers.
class Node {
 public:
  /// @brief Constructs a Node associated with a specific App instance.
  /// @param app A pointer to the App instance this node belongs to. This pointer must not be null.
  explicit Node(App* app);
  virtual ~Node() = default;

  Node(const Node&#x26; other) = delete;
  Node&#x26; operator=(const Node&#x26; other) = delete;
  Node(Node&#x26;&#x26; other) = delete;
  Node&#x26; operator=(Node&#x26;&#x26; other) = delete;

  /// @brief Returns the name of the node.
  /// @return A constant reference to the node's name.
  [[nodiscard]] const std::string&#x26; GetName() const;

  /// @brief Sets the name of the node.
  /// @param name The new name for the node.
  void SetName(std::string name);

  /// @brief Returns the App instance associated with this node.
  /// @return A pointer to the App instance. This pointer is never null after construction.
  [[nodiscard]] App* GetApp() const;

  /// @brief Returns the parent node in the scene graph. Can be null if this is the root node or not yet added as a child.
  /// @return A pointer to the parent Node, or null if there is no parent.
  [[nodiscard]] Node* GetParent() const;

  /// @brief Adds a new child node to this node. Ownership of the child is transferred.
  /// @param new_child A unique pointer to the Node to be added. This pointer must not be null.
  void AddChild(std::unique_ptr&#x3C;Node> new_child);

  /// @brief Creates and adds a new child node of the specified type to this node.
  /// @tparam T The type of the Node to create, must derive from Node.
  /// @tparam Args The constructor arguments for the Node type T.
  /// @param args The arguments to forward to the constructor of T.
  /// @return A reference to the newly created and added Node.
  template &#x3C;Derived&#x3C;Node> T, typename... Args>
  T&#x26; MakeChild(Args&#x26;&#x26;... args) {
    auto child = std::make_unique&#x3C;T>(app_, std::forward&#x3C;Args>(args)...);
    T&#x26; ref = *child;
    AddChild(std::move(child));
    return ref;
  }

  /// @brief Schedules a child node for destruction. The actual removal happens at the beginning of the next frame.
  /// @param child_to_destroy A constant reference to the child Node to be destroyed.
  void DestroyChild(const Node&#x26; child_to_destroy);

  /// @brief Schedules this node for destruction. The actual removal happens at the beginning of the next frame by its parent.
  void Destroy();

  /// @brief Returns the first child node of a specific type.
  /// @tparam T The type of the child Node to retrieve, must derive from Node.
  /// @return A pointer to the first child of type T, or nullptr if no such child exists.
  template &#x3C;Derived&#x3C;Node> T>
  [[nodiscard]] T* GetChild() {
    for (const auto&#x26; child : children_) {
      T* c = dynamic_cast&#x3C;T*>(child.get());
      if (c != nullptr) {
        return c;
      }
    }
    return nullptr;
  }

private:
  // The name of the node.
  std::string name_;
  
  // Pointer to the App instance. Never null after construction.
  App* app_ = nullptr;

  // Pointer to the parent node in the scene graph. Can be null for the root.
  Node* parent_ = nullptr;

  // Vector of child nodes. Ownership is managed by this node.
  std::vector&#x3C;std::unique_ptr&#x3C;Node>> children_;
  // Indices of children to be erased in the next update cycle.
  std::vector&#x3C;size_t> children_to_erase_;
  // Vector of children to be added in the next update cycle.
  std::vector&#x3C;std::unique_ptr&#x3C;Node>> children_to_add_;
};

}  // namespace ng
</code></pre>

Node creation involves either manually creating a unique pointer to the desired type and then attaching it as a child to a parent Node, or using the parent Node's `MakeChild` function, which streamlines this process.&#x20;

A crucial aspect of the Node system is its handling of destruction: when a Node is destroyed, it triggers a recursive destruction of all its children, ensuring that the entire hierarchical branch originating from the destroyed Node is properly cleaned up.

#### Derived Concept

The `Derived` C++ Concept is employed to restrict the allowable types for the `MakeChild` and `GetChild` methods. This enforcement of stricter static type checking in these templated methods helps catch type errors at compile time.

{% code title="engine/derived.h" %}
```cpp
#pragma once

#include <type_traits>

namespace ng {

/// @brief Concept that checks if type `T` is derived from type `U`.
/// @tparam T The potentially derived type.
/// @tparam U The base type.
template <class T, class U>
concept Derived = std::is_base_of_v<U, T>;

}  // namespace ng
```
{% endcode %}

{% code title="engine/node.cc" %}
```cpp
#include "node.h"

#include <cassert>
#include <cstddef>
#include <memory>
#include <string>
#include <utility>

namespace ng {

Node::Node(App* app) : app_(app) {
  assert(app);
}

const std::string& Node::GetName() const {
  return name_;
}

void Node::SetName(std::string name) {
  name_ = std::move(name);
}

App* Node::GetApp() const {
  return app_;
}

Node* Node::GetParent() const {
  return parent_;
}

void Node::AddChild(std::unique_ptr<Node> new_child) {
  new_child->parent_ = this;
  children_to_add_.push_back(std::move(new_child));
}

void Node::DestroyChild(const Node& child_to_destroy) {
  for (size_t i = 0; i < children_.size(); ++i) {
    if (children_[i].get() == &child_to_destroy) {
      children_to_erase_.push_back(i);
      break;
    }
  }
}

void Node::Destroy() {
  if (parent_ != nullptr) {
    parent_->DestroyChild(*this);
  }
}

}  // namespace ng
```
{% endcode %}

### Local and Global Transforms

A node's local transform defines its position, rotation, and scale relative to its own origin. In contrast, the global transform describes its position, rotation, and scale within the world coordinate system. For example, if node A is positioned at world coordinates (0, 10) and has a child node B with a local position of (2, 5), node A's local and global transforms are identical because it has no parent influencing its world position. However, node B's local position (2, 5) differs from its global position, which would be (2, 15) (assuming no rotation or scaling). This is because B's global position is calculated by adding its local position to its parent's global position. To optimize performance, the global transform is computed on demand, only when its value is explicitly requested. This lazy computation avoids unnecessary updates after every translation, rotation, or scale operation.

{% code title="engine/node.h" %}
```cpp
// --- Add: Includes. ---
#include <SFML/Graphics/Transformable.hpp>
#include <SFML/System/Angle.hpp>
#include <SFML/System/Vector2.hpp>
// --- End ---

namespace ng {

class Node {
 public:
  // -----
  // --- Add: Getters and setters for the local transform and global transform. ---
  /// @brief Returns the local transformation of this node.
  /// @return A constant reference to the local SFML Transformable.
  [[nodiscard]] const sf::Transformable& GetLocalTransform() const;

  /// @brief Returns the global transformation of this node, taking into account its parent's transformations.
  ///        This is calculated lazily and cached until the local transform changes.
  /// @return A constant reference to the global SFML Transformable.
  [[nodiscard]] const sf::Transformable& GetGlobalTransform() const;

  /// @brief Sets the local position of the node.
  /// @param position The new local position.
  void SetLocalPosition(sf::Vector2f position);

  /// @brief Sets the local rotation of the node.
  /// @param rotation The new local rotation.
  void SetLocalRotation(sf::Angle rotation);

  /// @brief Sets the local scale of the node.
  /// @param scale The new local scale.
  void SetLocalScale(sf::Vector2f scale);

  /// @brief Translates the node by a given delta vector in its local space.
  /// @param delta The translation vector.
  void Translate(sf::Vector2f delta);
  // --- End ---

 private:
  // --- Add: Make the global transform dirty. ---
  /// @brief Marks the global transform as dirty, forcing a recalculation on the next GetGlobalTransform call and propagating the dirty flag to children.
  void DirtyGlobalTransform();
  // --- End ---
  
  // -----

  // The name of the node.
  std::string name_;
  // --- Add: Local transform and cached global transform. ---
  // The local transformation of the node.
  sf::Transformable local_transform_;
  // The cached global transformation of the node. Mutable for lazy evaluation.
  mutable sf::Transformable global_transform_;
  // Flag indicating if the global transform needs to be recalculated. Mutable for lazy evaluation.
  mutable bool is_global_transform_dirty_ = false;
  // --- End ---

  // Pointer to the App instance. Never null after construction.
  App* app_ = nullptr;
  
  // -----
};

}  // namespace ng
```
{% endcode %}

{% code title="engine/node.cc" %}
```cpp
// -----
// --- Add: Includes. ---
#include <SFML/Graphics/Transform.hpp>
#include <SFML/Graphics/Transformable.hpp>
#include <SFML/System/Angle.hpp>
#include <SFML/System/Vector2.hpp>
#include <cmath>
#include <span>
// --- End ---

namespace ng {

void Node::AddChild(std::unique_ptr<Node> new_child) {
  new_child->parent_ = this;
  new_child->DirtyGlobalTransform(); // Add: Mark the child's transform dirty because parent changed.
  children_to_add_.push_back(std::move(new_child));
}

// --- Add: Implement local and global transform operations. ---
const sf::Transformable& Node::GetLocalTransform() const {
  return local_transform_;
}

const sf::Transformable& Node::GetGlobalTransform() const {
  if (is_global_transform_dirty_) {
    if (parent_ != nullptr) {
      sf::Transform new_transform =
          parent_->GetGlobalTransform().getTransform() *
          GetLocalTransform().getTransform();

      auto matrix = std::span(new_transform.getMatrix(), 16);

      float a00 = matrix[0];
      float a01 = matrix[4];
      float a02 = matrix[12];
      float a10 = matrix[1];
      float a11 = matrix[5];
      float a12 = matrix[13];

      sf::Angle rotation(sf::radians(std::atan2(-a01, a00)));
      sf::Vector2 scale(std::sqrt((a00 * a00) + (a10 * a10)),
                        std::sqrt((a01 * a01) + (a11 * a11)));
      sf::Vector2f position(a02, a12);

      global_transform_.setRotation(rotation);
      global_transform_.setScale(scale);
      global_transform_.setPosition(position);
    } else {
      global_transform_ = GetLocalTransform();
    }

    is_global_transform_dirty_ = false;
  }

  return global_transform_;
}

void Node::SetLocalPosition(sf::Vector2f position) {
  local_transform_.setPosition(position);
  DirtyGlobalTransform();
}

void Node::SetLocalRotation(sf::Angle rotation) {
  local_transform_.setRotation(rotation);
  DirtyGlobalTransform();
}

void Node::SetLocalScale(sf::Vector2f scale) {
  local_transform_.setScale(scale);
  DirtyGlobalTransform();
}

void Node::Translate(sf::Vector2f delta) {
  local_transform_.move(delta);
  DirtyGlobalTransform();
}

void Node::DirtyGlobalTransform() {
  if (is_global_transform_dirty_) {
    return;
  }

  is_global_transform_dirty_ = true;
  for (auto& child : children_) {
    child->DirtyGlobalTransform();
  }
}
// --- End ---

}  // namespace ng
```
{% endcode %}

### Node Lifecycle

Nodes within the scene often have specific functions (methods) that the engine calls automatically at different points in their lifecycle. The exact names vary greatly between engines (e.g., Unity uses `Awake`, `Start`, `Update`, `OnDestroy`; Godot uses `_ready`, `_process`, `_draw`, `_exit_tree`), but the concepts are similar:

1. **`OnAdd` (Initialization):**
   * **When:** Called shortly after the object is added to the active scene tree and is ready. It runs once per object lifecycle.
   * **Purpose:** Used for one-time setup tasks like initializing variables, getting references to other objects or components, etc.
2. **`Update` (Logic):**
   * **When:** Called **every single tick** while the object is active in the scene.
   * **Purpose:** This is where most ongoing game logic happens: handling input, moving the object, checking collisions, running AI logic, updating animations, etc.
3. **`Draw` (Rendering):**
   * **When:** Called during the rendering phase of each frame for objects that need to be drawn.
   * **Purpose:** Responsible for issuing commands to draw the object's visual representation (sprite, model, UI element) on the screen.
4. **`OnDestroy` (Cleanup):**
   * **When:** Called just before the object is permanently removed from the scene (either destroyed individually or because the whole scene is being unloaded).
   * **Purpose:** Used for cleanup tasks: releasing resources, saving data, disconnecting from event systems, and notifying other objects if necessary.

{% code title="engine/node.h" %}
```cpp
#include <SFML/Graphics/RenderTarget.hpp> // Add: Include.

namespace ng {

// -----
class Scene; // Add: Forward declare the Scene class.

class Node {
 public:
  // --- Add ---
  // Scene needs to be able to call InternalOnAdd, InternalUpdate,
  // InternalDraw, and InternalOnDestroy.
  friend class Scene;
  // --- End ---
  
 // -----
 
  // --- Add: Scene getter. ---
  /// @brief Returns the Scene this node belongs to. Can be null if the node is not yet added to a scene.
  /// @return A pointer to the Scene, or null if the node is not part of a scene.
  [[nodiscard]] Scene* GetScene() const;
  // --- End ---
  
 // -----

 // --- Add: Lifecycle methods. ---
 protected:
  /// @brief Called when the node is added to a scene graph.
  virtual void OnAdd();
  /// @brief Called during the update phase of the game loop.
  virtual void Update();
  /// @brief Called during the draw phase of the game loop.
  /// @param target The SFML RenderTarget to draw to.
  virtual void Draw(sf::RenderTarget& target);
  /// @brief Called when the node is about to be destroyed or removed from the scene graph.
  virtual void OnDestroy();
 // --- End ---

 private:
  // --- Add: Internal node's lifecycle logic. ---
  /// @brief Removes children that were scheduled for destruction in the previous frame.
  void EraseDestroyedChildren();
  /// @brief Adds children that were queued to be added in the previous frame.
  void AddQueuedChildren();

  /// @brief Internal method called when the node is added to a scene. Notifies the node and its children.
  /// @param scene A pointer to the Scene this node is being added to. This pointer must not be null.
  void InternalOnAdd(Scene* scene);
  /// @brief Internal method called during the update phase. Updates the node and its children.
  void InternalUpdate();
  /// @brief Internal method called during the draw phase. Draws the node and its children.
  /// @param target The SFML RenderTarget to draw to.
  void InternalDraw(sf::RenderTarget& target);
  /// @brief Internal method called when the node is about to be destroyed. Notifies the node and its children.
  void InternalOnDestroy();
  // --- End ---

  // -----

  // Pointer to the parent node in the scene graph. Can be null for the root.
  Node* parent_ = nullptr;
  // --- Add: Pointer to the scene owning this node. ---
  // Pointer to the Scene this node belongs to. Can be null if not yet added to a scene.
  Scene* scene_ = nullptr;
  // --- End ---
  // -----
};

}  // namespace ng
```
{% endcode %}

```cpp
// -----
// --- Add: Includes. ---
#include <SFML/Graphics/RenderTarget.hpp>
#include <algorithm>
#include <cstdint>
#include <functional>

#include "scene.h"
// --- End ---

namespace ng {

// -----
// --- Add: Scene and lifecycle methods. ---
Scene* Node::GetScene() const {
  return scene_;
}

void Node::OnAdd() {}

void Node::Update() {}

void Node::Draw([[maybe_unused]] sf::RenderTarget& target) {}

void Node::OnDestroy() {}

void Node::EraseDestroyedChildren() {
  if (children_to_erase_.empty()) {
    return;
  }

  // This is required because a newly destroyed child may destroy a new child
  // from its parent (this node), invalidating the current children_to_erase_ vector.
  auto prev_frame_children_to_erase = std::move(children_to_erase_);
  std::ranges::sort(prev_frame_children_to_erase, std::greater<>());
  for (size_t to_erase : prev_frame_children_to_erase) {
    children_[to_erase]->InternalOnDestroy();
    children_.erase(children_.begin() + static_cast<int64_t>(to_erase));
  }
}

void Node::AddQueuedChildren() {
  // This is required because a newly added child may add a new child
  // to its parent (this node), invalidating the current children_to_add_ vector.
  auto prev_frame_children_to_add = std::move(children_to_add_);
  for (auto& to_add : prev_frame_children_to_add) {
    Node* tmp = to_add.get();
    children_.push_back(std::move(to_add));
    tmp->InternalOnAdd(scene_);
  }
}

void Node::InternalOnAdd(Scene* scene) {
  scene_ = scene;
  scene_->RegisterNode(this);
  OnAdd();
}

void Node::InternalUpdate() {
  EraseDestroyedChildren();
  AddQueuedChildren();

  Update();
  for (auto& child : children_) {
    child->InternalUpdate();
  }
}

void Node::InternalDraw(sf::RenderTarget& target) {
  Draw(target);
  for (auto& child : children_) {
    child->InternalDraw(target);
  }
}

void Node::InternalOnDestroy() {
  scene_->UnregisterNode(this);
  OnDestroy();
  for (auto& child : children_) {
    child->InternalOnDestroy();
  }
}
// --- End ---

}  // namespace ng
```

### Scheduled Add and Destroy

You may have noticed that in the `AddChild` and `DestroyChild` methods, the node being added/destroyed is not processed right away, but rather scheduled to be processed. If you were to add or remove a Node _directly_ within the `Update` method, it could happen _while_ the engine (or another part of your code) is currently iterating over the list or tree structure containing that parent node.

Adding and removing an item from a collection while iterating over it can invalidate the iterator. The loop might crash, skip the next element, or exhibit other unpredictable (undefined) behavior because the structure it's traversing has changed unexpectedly underneath it.

To avoid these problems, node additions and removals are scheduled for the next tick of execution.\
When you call AddChild and DestroyChild, the engine doesn't modify the scene graph immediately. Instead, it puts the request into a queue.

Only _after_ all the main processing and potentially dangerous iterations for the current frame are complete, the engine processes these queued requests and actually modifies the scene graph structure.
