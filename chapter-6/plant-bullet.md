# Plant Bullet

## Plant Bullet 🌰

The `PlantBullet` travels continuously in a predetermined direction and is destroyed upon colliding with any Tile in the game map. Contact with a PlantBullet will cause the Player to get hit.

{% code title="game/plant_bullet.h" %}
```cpp
#pragma once

#include <SFML/Graphics/RenderTarget.hpp>
#include <SFML/Graphics/Sprite.hpp>
#include <SFML/System/Vector2.hpp>

#include "engine/circle_collider.h"
#include "engine/node.h"
#include "engine/tilemap.h"

namespace game {

class PlantBullet : public ng::Node {
 public:
  PlantBullet(ng::App* app, const ng::Tilemap* tilemap, sf::Vector2f direction);

  bool GetIsDead() const;

 protected:
  void Update() override;
  void Draw(sf::RenderTarget& target) override;

 private:
  const ng::Tilemap* tilemap_ = nullptr;
  sf::Vector2f direction_{-1, 0};
  const ng::CircleCollider* collider_ = nullptr;
  sf::Sprite sprite_;
  bool is_dead_ = false;
};

}  // namespace game
```
{% endcode %}

{% code title="game/plant_bullet.cc" %}
```cpp
#include "plant_bullet.h"

#include <SFML/Graphics/RenderTarget.hpp>
#include <SFML/System/Vector2.hpp>
#include <vector>

#include "engine/app.h"
#include "engine/circle_collider.h"
#include "engine/collider.h"
#include "engine/node.h"
#include "engine/tilemap.h"
#include "player.h"
#include "tile_id.h"

namespace game {

PlantBullet::PlantBullet(ng::App* app, const ng::Tilemap* tilemap,
                         sf::Vector2f direction)
    : ng::Node(app),
      tilemap_(tilemap),
      direction_(direction),
      sprite_(GetApp()->GetResourceManager().LoadTexture("Plant/Bullet.png")) {
  SetName("PlantBullet");
  sprite_.setScale({2, 2});
  sprite_.setOrigin({8, 8});
  sprite_.setTextureRect(sf::IntRect({0, 0}, {16, 16}));

  auto& collider = MakeChild<ng::CircleCollider>(4.F);
  collider_ = &collider;
}

bool PlantBullet::GetIsDead() const {
  return is_dead_;
}

namespace {

bool DoesCollide(sf::Vector2f position, const ng::Tilemap& tilemap) {
  TileID id = tilemap.GetWorldTile(position).GetID();
  return id == TileID::kInvisibleBarrier ||
         (id >= TileID::kDirtTopLeft && id <= TileID::kDirtBottomRight) ||
         (id >= TileID::kStoneHorizontalLeft &&
          id <= TileID::kStoneVerticalBottom) ||
         id == TileID::kPlasticBlock;
}

}  // namespace

void PlantBullet::Update() {
  if (is_dead_) {
    return;
  }

  sf::Vector2f pos = GetGlobalTransform().getPosition();
  if (!tilemap_->IsWithinWorldBounds(pos)) {
    is_dead_ = true;
    Destroy();
    return;
  }

  if (DoesCollide(pos, *tilemap_)) {
    is_dead_ = true;
    Destroy();
    return;
  }

  static constexpr float kMovementSpeed = 6;
  Translate(direction_ * kMovementSpeed);

  std::vector<const ng::Collider*> others =
      GetScene()->GetPhysics().Overlap(*collider_);
  for (const auto* other : others) {
    if (other->GetParent()->GetName() == "Player") {
      auto* player = dynamic_cast<Player*>(other->GetParent());
      player->TakeDamage();
    }
  }
}

void PlantBullet::Draw(sf::RenderTarget& target) {
  sprite_.setScale(sf::Vector2f{-direction_.x * 2, 2.F});
  target.draw(sprite_, GetGlobalTransform().getTransform());
}

}  // namespace game
```
{% endcode %}
