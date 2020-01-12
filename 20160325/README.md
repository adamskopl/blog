# RunPigsRun: Handling Collisions

## Collisions in RunPigsRun

One of the things that gives me some confidence about finishing this project is the form of its gameplay. Objects are moved from field to field in a predictable way. Collisions are checked after all the objects finish their movement, which simplifies the work.

Which collisions are already handled?

1. Hero + Bouncer
  * changes object’s speed
  * plays scaling animation for the bouncer
2. Hero + Void field; Hero + Water field
  * removes hero from the game
  * reports game’s loss
  * plays field’s scale animation
3. Hero + Exit
  * removes hero from the game
  * reports that hero was saved

## How it’s implemented

I really like Javascript’s duck typing. Earlier Mr. Czocher reported in comments, that I should checkout Typescript, which (I assume from the introduction videos) will make code more standardized e.g. by interfaces. I’m really curious if it will not lower the speed of coding, as with my current code organization I’m having a really small amount of bugs and generally work goes smooth. That was just a small digression. What I want to say, that with Javascript’s conventions I’ve come up with a nice way to define functions handling collisions between pair of objects.

First: objects which ends their movement outside of the map are removed. Before that, CollisionsHandler sends signal about object which is about to be removed and GameResultResolver checks if it’s not a Hero. 

```js
/**
 * Every remove should go through here to ensure signal dispatch.
 */
CollisionsHandler.prototype.handleRemove = 
function(GAME_OBJECT, SIGNAL) {
  // dispatch signal BEFORE removing object
  this.signals[SIGNAL].dispatch(GAME_OBJECT);
  this.gameObjectsManager.remove(GAME_OBJECT);
};

GameResultResolver.prototype.slotObjectRemoved = 
function(OBJECT) {
  if (OBJECT.type === GOT.HERO)
    this.resultObject.loss = true;
};
```

Next all the objects on the tile are compared between each other with handleCollisionPair().

```js
CollisionsHandler.prototype.handleTileCollisions = 
function(TILE) {
  for (var I = 0; I < TILE.length; I++)
    for (var J = I + 1; J < TILE.length; J++) {
      // unique pair (no two objects compared more than once)
      RESULTS = handleCollisionPair(TILE[I], TILE[J]);
      if (RESULTS === undefined) {
        console.error("RESULTS === undefined");
        continue;
      }
      for (var R in RESULTS)
        this.handleCollisionResult(RESULTS[R]);
    }
};
```

## handleCollisionPair()

Now the heart of this post. I was wondering how to write simple code, which will:

- handle collision result between a pair of objects
- take into account that the order of passed objects is not predictable (is it
  HERO vs BOUNCER or BOUNCER vs HERO?)

The solution is quite elegant in my opinion: objects’ type names are used to locate the name of the function returning collision result. 

```js
/**
 * Get collision function name by concating object type names 
 * sorted alphabetically.
 */
function getCollisionFunctionName(OBJ_A, OBJ_B) {
  return "collision" + ((OBJ_A.type < OBJ_B.type) ?
    OBJ_A.type + OBJ_B.type :
    OBJ_B.type + OBJ_A.type);
};
```

So: collision between ‘hero’ and ‘t_bouncer’ gives the same function name as collision between ‘t_bouncer’ and ‘hero’. In this case: collisionherot_bouncer(). handleCollisionPair() returns the result of such function and passed arguments are also sorted alphabetically, which ensures that collision function is invoked always in the same way: 

```js
function handleCollisionPair(OBJ_A, OBJ_B) {
  var fname = getCollisionFunctionName(OBJ_A, OBJ_B);
  if (typeof window[fname] !== 'function')
    console.error("no colision function: " + fname);
  if (OBJ_A.type < OBJ_B.type)
    return window[fname](OBJ_A, OBJ_B);
  else
    return window[fname](OBJ_B, OBJ_A); // switch objects (alphabetical order)
};
```

## Collision results

Functions checking collision result return an array of CollisionResult objects which keep data about objects modified as a result of the collision, type of operation which should be performed and optional arguments. 

```js
function CollisionResult(OBJECT, OPERATION, ARG) {
  this.object = OBJECT;
  this.operation = OPERATION;
  this.arg = ARG;
};

COLLISION_OPERATION = Object.freeze({
  REMOVE: "remove",
  RESCUE: "rescue", // rescue object (hero)
  SCALE_ANIMATION: "scaleAnim", // temporary firework
  SPEED_CHANGE: "speed"
});
```

Example of function resolving collision between HERO and BOUNCER: 

```js
function collisionherot_bouncer(HERO, TOOL_BOUNCER) {
  return [
    new CollisionResult(
      HERO, COLLISION_OPERATION.SPEED_CHANGE, 2),
    new CollisionResult(
      TOOL_BOUNCER, COLLISION_OPERATION.SCALE_ANIMATION, 1.5)
  ];
};
```

Easy and clear: collision changes Hero’s speed (bounce) and plays scale animation for bouncer. Now every object from the array is passed to handleCollisionResult(). 

```js
CollisionsHandler.prototype.handleCollisionResult = 
function(RESULT) {
  switch (RESULT.operation) {
    case COLLISION_OPERATION.REMOVE:
      this.handleRemove(RESULT.object, "objectRemoved");
    case COLLISION_OPERATION.SPEED_CHANGE:
      RESULT.object.setSpeed(RESULT.arg);
      break;
    case COLLISION_OPERATION.SCALE_ANIMATION:
      RESULT.object.startScaleAnimation(this.game, RESULT.arg);
      break;
    case COLLISION_OPERATION.RESCUE:
      if (RESULT.object.type === GOT.HERO)
        this.handleRemove(RESULT.object, "objectRescued");
      break;
    default:
      console.error("not implemented: " + RESULT.operation);
  }
};
```

## In the end.

If the new object is introduced, e.g ‘superObject’, and collision between this object and ‘hero’ object should be handled, only the… 

```js
function collisionherosuperobject(HERO, SUPEROBJECT) {
  return [];
};
```

 returning the array of collision results, should be added.

New collision = new function. Easy. 
