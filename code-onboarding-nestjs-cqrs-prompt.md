# Code QA Task — Code Onboarding & Comprehension (TypeScript / NestJS CQRS)

## prompt_title

What Is the Hero Model Doing With AggregateRoot and apply()?

## prompt_statement

I just joined this NestJS backend team and the heroes module is the first area I've been
assigned to. Before I start contributing I want to make sure I actually understand how the
existing pieces work — otherwise I'll just be copying patterns I don't really get.

I found this domain model class and I'm confused about a few things. It extends something
called `AggregateRoot` which I haven't seen before. Inside the `killEnemy` and `addItem`
methods it calls `this.apply()` and passes in a new event object, but I don't understand
what `apply()` actually does with that event or where it goes after being passed in. The
methods also have a `// logic` comment where I'd expect the actual state mutation to be,
so I'm not sure if this is a placeholder or if the event application is the logic. I also
don't understand why the event is created inside the model method rather than in the
handler that called `killEnemy` in the first place.

Can you walk me through what this class does, what `AggregateRoot` gives it, what
`apply()` is doing when it's called, and what the relationship is between calling
`killEnemy()` and the `HeroKilledDragonEvent` that gets created inside it?

```typescript
import { AggregateRoot } from '../../../../src';
import { HeroFoundItemEvent } from '../events/impl/hero-found-item.event';
import { HeroKilledDragonEvent } from '../events/impl/hero-killed-dragon.event';

export class Hero extends AggregateRoot {
  constructor(private readonly id: string) {
    super();
  }

  killEnemy(enemyId: string) {
    // logic
    this.apply(new HeroKilledDragonEvent(this.id, enemyId));
  }

  addItem(itemId: string) {
    // logic
    this.apply(new HeroFoundItemEvent(this.id, itemId));
  }
}
```
