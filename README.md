kit-ecs
======
### Kit implementation of Entity/Component/System paradigm
------
Quick Start Guide
------

Creating an `Entity` is easy:
```c
var player: Entity = Entity.new("Player");
```
The static new function returns a stack allocated `Entity`, taking in the name of the `Entity`. If you wish to create a heap allocated (pointer) to an `Entity`:
```c
var player: Ptr[Entity] = allocator.alloc(sizeof Entity);
player = Entity.new("Player");
```
Next we should create an `Engine`, which is the entry point to the entire ECS structure.
```c
var engine: Engine = Engine.new();
```
An application will typically only have one `Engine`, an `Entity` can only have one `Engine`.

Adding the our `Entity` to the `Engine` is easy.
```c
engine.addEntity(player);
```
Next we will create a `Component` to add to our `Entity`. 
```c
struct PositionComponent {
    var x: Float;
    var y: Float;
}

implement Component for PositionComponent{
    function typeIdentifier(): CString{
        return "Position";
    }
}

struct VelocityComponent {
    var x: Float;
    var y: Float;
}

implement Component for VelocityComponent{
    function typeIdentifier(): CString{
        return "Velocity";
    }
}
```

The `Trait Component` requires the method `typeIdentifier()` to be defined. This should return a unique `CString` that denotes the `type` of the component. 

Adding the `Component` to the `Entity` is simple, and can be done before or after adding to the `Engine` as `Entites` will let the `Engine` know it has changed.

```c
var playerPosition: PositionComponent = struct PositionComponent {
    x: 12.0,
    y: 12.0
};

var playerPosition: PositionComponent = struct VelocityComponent {
    x: 1.2,
    y: 1.2
};

player.addComponent(playerPosition);
```
We've now done the `Entity` and `Component` section of ECS, now we should cover the `System`.
```c
struct MovementSystem{
    var entityIsMoving: Bool = false;
}

implement System for MovementSystem{

    public function update(delta: Float, engine: Ptr[Engine]): Void{
        var entitesWithPosition = engine.entitiesForComponent("Position");

        for entity in entityArray{
            this.entityIsMoving = true;
            entity.getComponent("Position").unwrap().x += (entity.getComponent("Velocity").unwrap.x * delta);
            entity.getComponent("Position").unwrap().y += (entity.getComponent("Velocity").unwrap.y * delta);
            this.entityIsMoving = false;
        }
    }
    function typeIdentifier(): CString {
        return "Position";
    }
}
```
`Trait System` requires `struct` to define two methods, `update(delta: Float, engine: Ptr[Engine])` and `typeIdentifier()`, the later returns a unique `CString` identifer for this system. The former method is called each iteration of the engine update method, and is passed the delta time and a pointer to the `Engine` itself for querying.

You would then add the system itself to the engine.
```c
var movementSystem = struct MovementSystem{
    entityIsMoving: false
};

engine.addSystem(movementSystem);
```

Inside your game loop, you would then call:
```c
engine.update(GetDeltaTime());
```
The engine will then call update on every system it has.