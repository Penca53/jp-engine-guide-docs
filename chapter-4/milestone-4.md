# Milestone 4

## Physics Interactions, Banana, and Scene Reload

The player can collect the banana and interact with the mushrooms.&#x20;

When a player hits a mushroom while falling, it removes the mushroom from the scene.&#x20;

When a mushroom hits the player while it is not falling, it removes it.  When a mushroom hits another mushroom, it flips its horizontal direction.

Pressing Enter after the player is dead reloads the whole scene.

Notice the green outline around the player and the mushrooms that visually represent the colliders (only in Debug mode). Compared to the previous Milestone, the player and the mushrooms collide with the map on the edges of the collider, and not on the edges of the sprite itself (which instead has some empty room around itself).

<figure><img src="../.gitbook/assets/Screen Recording 2025-04-11 at 15.32.07.gif" alt=""><figcaption><p>(No SFX in the recording)</p></figcaption></figure>
