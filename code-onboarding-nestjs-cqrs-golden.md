# Golden Answer — What Is the Hero Model Doing With AggregateRoot and apply()?

## intent_category
Code Onboarding & Comprehension

## golden_answer

`Hero` extends `AggregateRoot`, which is a base class provided by `@nestjs/cqrs`. Its purpose is to give your domain model the machinery for tracking domain events, which are records of significant things that happened to this aggregate during a request. Internally, `AggregateRoot` maintains a list called `INTERNAL_EVENTS` and exposes three key pieces of behavior: `apply()` to stage a new event onto that list, `commit()` to flush all staged events out through the event bus, and stub implementations of `publish()` and `publishAll()` that do nothing by default.

When `killEnemy(enemyId)` calls `this.apply(new HeroKilledDragonEvent(this.id, enemyId))`, nothing is dispatched anywhere. `apply()` simply pushes the new event object onto `INTERNAL_EVENTS` and returns. The event sits there, staged, waiting. No handler is notified and no saga receives anything yet.

The actual delivery happens when `commit()` is called. `commit()` invokes `publishAll()`, which iterates over every staged event and calls `publish()` for each one. The catch is that `publish()` is a no-op stub by default; it does nothing unless it has been patched. That patching is done by `EventPublisher.mergeObjectContext(hero)`. When a command handler wraps the hero instance with `mergeObjectContext`, it replaces the stub `publish()` and `publishAll()` methods on that specific instance with real implementations that route through the `EventBus`. From that point on, calling `commit()` actually delivers the staged events to every registered event handler and saga in the system.

The `// logic` comment is a placeholder for state mutation — the in-memory changes to the aggregate's own fields that should accompany the event. In a real implementation you might update a `kills` counter, remove the defeated enemy from an `activeEnemies` list, or adjust the hero's experience points. That code goes where the comment sits, before or alongside the `apply()` call. The `apply()` records the fact that the kill happened as a domain event; the surrounding logic updates the object's state accordingly. That placeholder is not unusual; in the NestJS CQRS sample app it signals that the actual domain rules are left for whoever is building on top of the framework.

The event is constructed inside `killEnemy()` rather than in the command handler for a domain-driven design reason: the aggregate owns its own logic. The handler's responsibility is coordination: retrieve the hero, invoke the operation, save the result, call `commit()`. The hero's responsibility is to know what events its own state transitions produce. If you moved event construction into the handler, the handler would need to know the hero's `id` and which event class corresponds to which action. The domain model would become a passive data bag rather than a self-contained object that encapsulates its own behavior. By creating `HeroKilledDragonEvent` inside `killEnemy()`, the `Hero` class stays the single authority on what a kill means in domain terms.

Both `killEnemy()` and `addItem()` follow the exact same pattern — `apply()` with a new event object — which tells you this is the standard mechanism for recording all domain events on this aggregate, not something special to one method.

The full sequence, end to end: `killEnemy()` runs (you'd put your state mutation in place of `// logic`), `apply()` stages the event in `INTERNAL_EVENTS`, the command handler later calls `hero.commit()`, `commit()` calls `publishAll()`, which calls `publish()` for each staged event, and, because `mergeObjectContext()` already patched those methods, the events flow through the `EventBus` to every registered handler and saga.
