# RunPigsRun: About the Progress

 A little report on a progress.

I had some issues with collisions, as it turned out, I haven’t taken into account collisions between objects passing each other. Previously, collisions were only checked, when objects were ending their movement on a tile. It’s a game’s clock: one movement iteration (movement from one tile to another, also with a different speed).

When I have introduced Enemy objects (objects killing Heroes) it turned out, that there is no collision detection between objects passing each other… Collisions where checked when all objects finished their movement on a target tile. I’ve fixed that lately and now collisions events are distinguished on:

- those fired, whenever collision between objects occur
- and on those fired, when both objects end their movement on the same tile.

 Maybe next time I’ll explain the reason behind this concept.

I have made code easier by introducing ‘void’ type fields, to check collisions for objects which bounces off outside the road. In this way empty fields are also participating in collision tests.

The next thing is choosing tools by a player. The set of available tools is limited and separate for every level (a different set for each level). When player exceeds the limit, the first chosen tool is erased to make room for a new choice. It allows to change decision with a minimum clicking.

Also tools have different modes: e.g. Signpost tool (which changes Hero’s direction) can be rotated. Player changes tool’s mode by choosing again the same field: tool’s next state is chosen and after choosing all the states, the tool is removed. In this way player by a single operation can either change tool’s mode or remove it. It’s an important concept, as more tools will have their modes too.

Clear conditions about lose/win introduced. Now whenever Hero dies (falls into the water/is killed by the enemy/bounces outside the road) the game is lost: black rectangle displayed and level is restarted. Player wins when all the heroes finish their movement on the Exit fields. 
