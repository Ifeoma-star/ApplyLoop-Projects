# Code QA Task — Code Onboarding & Comprehension (TypeScript / NestJS CQRS)

## prompt_title

How Does the NestJS CQRS Heroes Module Actually Work?

## prompt_statement

I just joined this NestJS backend team and the heroes game module is the first thing I've been asked to understand before I make any changes. I've read through all the files below and I can follow each one individually, but I cannot figure out how they connect at runtime. The whole thing feels like it's held together by magic I can't see.

Here's what I don't understand. The `KillDragonCommand` is a plain class with no decorators and no base class — it's just a constructor with two properties. So when a controller or test calls `commandBus.execute(new KillDragonCommand(heroId, dragonId))`, how does the framework know to call `KillDragonHandler`? What registers that mapping, and what does `@CommandHandler(KillDragonCommand)` actually do at the framework level?

Inside `KillDragonHandler.execute()`, the code calls `this.publisher.mergeObjectContext(await this.repository.findOneById(+heroId))`. The repository returns a `Hero` instance. What does `mergeObjectContext` do to that instance, and why is it necessary? The `Hero` class already extends `AggregateRoot`, which has `publish()` on it, so why isn't that enough on its own?

After `mergeObjectContext`, the handler calls `hero.killEnemy(dragonId)` and then `hero.commit()`. Inside `killEnemy()` there is a `// logic` comment and then `this.apply(new HeroKilledDragonEvent(this.id, enemyId))`. I understand that `apply()` stages the event, but what is the relationship between calling `killEnemy` and the event eventually reaching `HeroesGameSagas`? The saga is a completely separate class — how does it receive the event? Who connects the two?

In `HeroesGameSagas`, the `dragonKilled` property is decorated with `@Saga()` and takes `events$` as an argument. I've never seen `@Saga()` before and I don't know who calls `dragonKilled`, when, or where `events$` comes from. The function applies `ofType(HeroKilledDragonEvent)` to the stream, waits one second with `delay(1000)`, and then returns a `new DropAncientItemCommand`. What happens to that command after it leaves the `map` operator — does the developer need to dispatch it manually, or does something pick it up automatically?

I also notice that `DropAncientItemHandler` follows the exact same pattern as `KillDragonHandler` — it calls `mergeObjectContext`, then `hero.addItem(itemId)`, then `hero.commit()`. So the saga triggers a second command which goes through another full handler cycle. Is the pattern truly that every domain operation goes through this same apply/commit/publish cycle, even when triggered by a saga rather than directly by a user request?

Looking at `HeroesGameModule`, I see that `CqrsModule` is not imported there at all — there's no `imports` array. But the module registers `HeroesGameSagas` and all the handlers as providers. Where does `CqrsModule` get registered, and how do the handlers and sagas in this module get wired to the buses if `CqrsModule` isn't in their own module? I can see in `AppModule` that `CqrsModule.forRoot()` is imported at the root level — but I still don't understand how that makes the buses available to providers inside `HeroesGameModule`.

There is also a `ScopedModule` registered in `AppModule` alongside `HeroesGameModule`. Looking at the scoped handlers like `ScopedKillDragonHandler`, I notice that `@CommandHandler` is called with a second argument `{ scope: Scope.REQUEST }` — the regular `KillDragonHandler` has no such argument. What does `Scope.REQUEST` do to the handler's lifecycle, and why would you use it here instead of the default singleton scope?

The scoped handlers also inject `@Inject(REQUEST) public readonly context: AsyncContext` into their constructors and pass `this.context` as a second argument to `mergeObjectContext`. The regular handlers pass nothing beyond the hero instance. What is `AsyncContext`, what does it carry, and why does the scoped version need to thread it through `mergeObjectContext` and into the aggregate?

Finally, in the scoped saga's `dragonKilled`, after creating `ScopedDropAncientItemCommand`, there is a call to `AsyncContext.merge(event, command)` before returning the command. There is no equivalent line in the regular `HeroesGameSagas`. What does `AsyncContext.merge` do, and why is it necessary specifically in the scoped saga when passing a command from one request-scoped context into the next handler?

Can you walk me through the full execution path from the moment `commandBus.execute(new KillDragonCommand(...))` is called, through every layer, all the way to `DropAncientItemCommand` being dispatched and handled — and explain how that flow differs in the scoped variant where `AsyncContext` is involved?

```typescript
import { Module } from '@nestjs/common';
import { CqrsModule } from '../../src';
import { UnhandledExceptionCommandHandler } from './errors/commands/unhandled-exception.handler';
import { UnhandledExceptionEventHandler } from './errors/events/unhandled-exception.handler';
import { ErrorsSagas } from './errors/sagas/errors.saga';
import { HeroesGameModule } from './heroes/heroes.module';
import { NoopModule } from './noop/noop.module';
import { ScopedModule } from './scoped/scoped.module';

@Module({
  imports: [HeroesGameModule, CqrsModule.forRoot(), ScopedModule, NoopModule],
  providers: [
    UnhandledExceptionCommandHandler,
    UnhandledExceptionEventHandler,
    ErrorsSagas,
  ],
})
export class AppModule {}
```

```typescript
import { Module } from '@nestjs/common';
import { CommandHandlers } from './commands/handlers';
import { EventHandlers } from './events/handlers';
import { QueryHandlers } from './queries/handlers';
import { HeroRepository } from './repository/hero.repository';
import { HeroesGameSagas } from './sagas/heroes.sagas';

@Module({
  controllers: [],
  providers: [
    HeroRepository,
    ...CommandHandlers,
    ...EventHandlers,
    ...QueryHandlers,
    HeroesGameSagas,
  ],
})
export class HeroesGameModule {}
```

```typescript
import { KillDragonHandler } from './kill-dragon.handler';
import { DropAncientItemHandler } from './drop-ancient-item.handler';

export const CommandHandlers = [KillDragonHandler, DropAncientItemHandler];
```

```typescript
import { HeroKilledDragonHandler } from './hero-killed-dragon.handler';
import { HeroFoundItemHandler } from './hero-found-item.handler';

export const EventHandlers = [HeroKilledDragonHandler, HeroFoundItemHandler];
```

```typescript
import { GetHeroesHandler } from './get-heroes.handler';

export const QueryHandlers = [GetHeroesHandler];
```

```typescript
export * from './get-heroes.query';
```

```typescript
export class KillDragonCommand {
  constructor(
    public readonly heroId: string,
    public readonly dragonId: string,
  ) {}
}
```

```typescript
export class DropAncientItemCommand {
  constructor(public readonly heroId: string, public readonly itemId: string) {}
}
```

```typescript
export class GetHeroesQuery {}
```

```typescript
export class HeroKilledDragonEvent {
  constructor(
    public readonly heroId: string,
    public readonly dragonId: string,
  ) {}
}
```

```typescript
export class HeroFoundItemEvent {
  constructor(public readonly heroId: string, public readonly itemId: string) {}
}
```

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

```typescript
import { Injectable } from '@nestjs/common';
import { Hero } from '../models/hero.model';
import { userHero } from './fixtures/user';

@Injectable()
export class HeroRepository {
  async findOneById(id: number): Promise<Hero> {
    return userHero;
  }

  async findAll(): Promise<Hero[]> {
    return [userHero];
  }
}
```

```typescript
import { Logger } from '@nestjs/common';
import {
  CommandHandler,
  EventPublisher,
  ICommandHandler,
} from '../../../../../src';
import { HeroRepository } from '../../repository/hero.repository';
import { KillDragonCommand } from '../impl/kill-dragon.command';

@CommandHandler(KillDragonCommand)
export class KillDragonHandler implements ICommandHandler<KillDragonCommand> {
  constructor(
    private readonly repository: HeroRepository,
    private readonly publisher: EventPublisher,
  ) {}

  async execute(command: KillDragonCommand) {
    Logger.debug('KillDragonHandler has been called');

    const { heroId, dragonId } = command;
    const hero = this.publisher.mergeObjectContext(
      await this.repository.findOneById(+heroId),
    );
    hero.killEnemy(dragonId);
    hero.commit();
  }
}
```

```typescript
import { Logger } from '@nestjs/common';
import {
  CommandHandler,
  EventPublisher,
  ICommandHandler,
} from '../../../../../src';
import { HeroRepository } from '../../repository/hero.repository';
import { DropAncientItemCommand } from '../impl/drop-ancient-item.command';

@CommandHandler(DropAncientItemCommand)
export class DropAncientItemHandler
  implements ICommandHandler<DropAncientItemCommand>
{
  constructor(
    private readonly repository: HeroRepository,
    private readonly publisher: EventPublisher,
  ) {}

  async execute(command: DropAncientItemCommand) {
    Logger.debug('DropAncientItemHandler has been called');

    const { heroId, itemId } = command;
    const hero = this.publisher.mergeObjectContext(
      await this.repository.findOneById(+heroId),
    );
    hero.addItem(itemId);
    hero.commit();
  }
}
```

```typescript
import { Logger } from '@nestjs/common';
import { EventsHandler, IEventHandler } from '../../../../../src';
import { HeroKilledDragonEvent } from '../impl/hero-killed-dragon.event';

@EventsHandler(HeroKilledDragonEvent)
export class HeroKilledDragonHandler
  implements IEventHandler<HeroKilledDragonEvent>
{
  handle(event: HeroKilledDragonEvent) {
    Logger.debug('HeroKilledDragonHandler has been called');
  }
}
```

```typescript
import { Logger } from '@nestjs/common';
import { EventsHandler, IEventHandler } from '../../../../../src';
import { HeroFoundItemEvent } from '../impl/hero-found-item.event';

@EventsHandler(HeroFoundItemEvent)
export class HeroFoundItemHandler implements IEventHandler<HeroFoundItemEvent> {
  handle(event: HeroFoundItemEvent) {
    Logger.log('HeroFoundItemEvent...');
  }
}
```

```typescript
import { Logger } from '@nestjs/common';
import { IQueryHandler, QueryHandler } from '../../../../../src';
import { HeroRepository } from '../../repository/hero.repository';
import { GetHeroesQuery } from '../impl';

@QueryHandler(GetHeroesQuery)
export class GetHeroesHandler implements IQueryHandler<GetHeroesQuery> {
  constructor(private readonly repository: HeroRepository) {}

  async execute(query: GetHeroesQuery) {
    Logger.debug('GetHeroesQuery has been called');
    return this.repository.findAll();
  }
}
```

```typescript
import { Injectable, Logger } from '@nestjs/common';
import { Observable } from 'rxjs';
import { delay, map } from 'rxjs/operators';
import { ICommand, ofType, Saga } from '../../../../src';
import { DropAncientItemCommand } from '../commands/impl/drop-ancient-item.command';
import { HeroKilledDragonEvent } from '../events/impl/hero-killed-dragon.event';

export const ANCIENT_ITEM_ID = '12456789';

@Injectable()
export class HeroesGameSagas {
  @Saga()
  dragonKilled = (events$: Observable<any>): Observable<ICommand> => {
    return events$.pipe(
      ofType(HeroKilledDragonEvent),
      delay(1000),
      map((event) => {
        Logger.debug('Inside [HeroesGameSagas] Saga');
        return new DropAncientItemCommand(event.heroId, ANCIENT_ITEM_ID);
      }),
    );
  };
}
```

```typescript
import { Hero } from '../../models/hero.model';

export const HERO_ID = '1234';

export const userHero = new Hero(HERO_ID);
```

```typescript
export class UnhandledExceptionCommand {
  constructor(public readonly failAt: 'command' | 'event' | 'saga') {}
}
```

```typescript
export class UnhandledExceptionEvent {
  constructor(public readonly failAt: 'command' | 'event' | 'saga') {}
}
```

```typescript
import { CommandHandler, EventBus, ICommandHandler } from '../../../../src';
import { UnhandledExceptionEvent } from '../events/unhandled-exception.event';
import { UnhandledExceptionCommand } from './unhandled-exception.command';

@CommandHandler(UnhandledExceptionCommand)
export class UnhandledExceptionCommandHandler
  implements ICommandHandler<UnhandledExceptionCommand>
{
  constructor(private readonly eventBus: EventBus) {}

  async execute(command: UnhandledExceptionCommand) {
    if (command.failAt === 'command') {
      throw new Error(`Unhandled exception in ${command.failAt}`);
    } else {
      this.eventBus.publish(new UnhandledExceptionEvent(command.failAt));
    }
  }
}
```

```typescript
import { EventsHandler, IEventHandler } from '../../../../src';
import { UnhandledExceptionEvent } from './unhandled-exception.event';

@EventsHandler(UnhandledExceptionEvent)
export class UnhandledExceptionEventHandler
  implements IEventHandler<UnhandledExceptionEvent>
{
  async handle(event: UnhandledExceptionEvent) {
    if (event.failAt === 'event') {
      throw new Error(`Unhandled exception in ${event.failAt}`);
    }
  }
}
```

```typescript
import { Injectable } from '@nestjs/common';
import { Observable, of } from 'rxjs';
import { delay, mergeMap } from 'rxjs/operators';
import { ofType, Saga } from '../../../../src';
import { UnhandledExceptionEvent } from '../events/unhandled-exception.event';

@Injectable()
export class ErrorsSagas {
  @Saga()
  onError = (events$: Observable<any>): Observable<any> => {
    return events$.pipe(
      ofType(UnhandledExceptionEvent),
      delay(1000),
      mergeMap((event) => {
        if (event.failAt === 'saga') {
          throw new Error(`Unhandled exception in ${event.failAt}`);
        }
        return of();
      }),
    );
  };
}
```

```typescript
export class NoopEvent {}
```

```typescript
import { EventsHandler, IEventHandler } from '../../../../../src';
import { NoopEvent } from '../impl/noop.event';

@EventsHandler(NoopEvent)
export class NoopHandler implements IEventHandler<NoopEvent> {
  handle(event: NoopEvent) {}
}
```

```typescript
import { Module } from '@nestjs/common';
import { NoopHandler } from './events/handlers/noop.handler';

@Module({
  providers: [NoopHandler],
})
export class NoopModule {}
```

```typescript
import { Module } from '@nestjs/common';
import { CommandHandlers } from './commands/handlers';
import { EventHandlers } from './events/handlers';
import { QueryHandlers } from './queries/handlers';
import { HeroRepository } from './repository/hero.repository';
import { HeroesGameSagas } from './sagas/heroes.sagas';

@Module({
  controllers: [],
  providers: [
    HeroRepository,
    ...CommandHandlers,
    ...EventHandlers,
    ...QueryHandlers,
    HeroesGameSagas,
  ],
})
export class ScopedModule {}
```

```typescript
export class ScopedKillDragonCommand {
  constructor(
    public readonly heroId: string,
    public readonly dragonId: string,
  ) {}
}
```

```typescript
export class ScopedDropAncientItemCommand {
  constructor(
    public readonly heroId: string,
    public readonly itemId: string,
  ) {}
}
```

```typescript
export class ScopedGetHeroesQuery {}
```

```typescript
export class ScopedHeroKilledDragonEvent {
  constructor(
    public readonly heroId: string,
    public readonly dragonId: string,
  ) {}
}
```

```typescript
export class ScopedHeroFoundItemEvent {
  constructor(
    public readonly heroId: string,
    public readonly itemId: string,
  ) {}
}
```

```typescript
import { AggregateRoot } from '../../../../src';
import { ScopedHeroFoundItemEvent } from '../events/impl/hero-found-item.event';
import { ScopedHeroKilledDragonEvent } from '../events/impl/hero-killed-dragon.event';

export class Hero extends AggregateRoot {
  constructor(private readonly id: string) {
    super();
  }

  killEnemy(enemyId: string) {
    // logic
    this.apply(new ScopedHeroKilledDragonEvent(this.id, enemyId));
  }

  addItem(itemId: string) {
    // logic
    this.apply(new ScopedHeroFoundItemEvent(this.id, itemId));
  }
}
```

```typescript
import { Hero } from '../../models/hero.model';

export const HERO_ID = '1234';

export const userHero = new Hero(HERO_ID);
```

```typescript
import { Injectable } from '@nestjs/common';
import { Hero } from '../models/hero.model';
import { userHero } from './fixtures/user';

@Injectable()
export class HeroRepository {
  async findOneById(id: number): Promise<Hero> {
    return userHero;
  }

  async findAll(): Promise<Hero[]> {
    return [userHero];
  }
}
```

```typescript
import { ScopedDropAncientItemHandler } from './scoped-drop-ancient-item.handler';
import { ScopedKillDragonHandler } from './scoped-kill-dragon.handler';

export const CommandHandlers = [
  ScopedKillDragonHandler,
  ScopedDropAncientItemHandler,
];
```

```typescript
import { Inject, Logger, Scope } from '@nestjs/common';
import { REQUEST } from '@nestjs/core';
import {
  AsyncContext,
  CommandHandler,
  EventPublisher,
  ICommandHandler,
} from '../../../../../src';
import { HeroRepository } from '../../repository/hero.repository';
import { ScopedKillDragonCommand } from '../impl/scoped-kill-dragon.command';

@CommandHandler(ScopedKillDragonCommand, {
  scope: Scope.REQUEST,
})
export class ScopedKillDragonHandler
  implements ICommandHandler<ScopedKillDragonCommand>
{
  constructor(
    private readonly repository: HeroRepository,
    private readonly publisher: EventPublisher,
    @Inject(REQUEST) public readonly context: AsyncContext,
  ) {}

  async execute(command: ScopedKillDragonCommand) {
    Logger.debug('KillDragonHandler has been called');

    const { heroId, dragonId } = command;
    const hero = this.publisher.mergeObjectContext(
      await this.repository.findOneById(+heroId),
      this.context,
    );
    hero.killEnemy(dragonId);
    hero.commit();

    return { heroId, dragonId };
  }
}
```

```typescript
import { Inject, Logger, Scope } from '@nestjs/common';
import { REQUEST } from '@nestjs/core';
import {
  AsyncContext,
  CommandHandler,
  EventPublisher,
  ICommandHandler,
} from '../../../../../src';
import { HeroRepository } from '../../repository/hero.repository';
import { ScopedDropAncientItemCommand } from '../impl/scoped-drop-ancient-item.command';

@CommandHandler(ScopedDropAncientItemCommand, {
  scope: Scope.REQUEST,
})
export class ScopedDropAncientItemHandler
  implements ICommandHandler<ScopedDropAncientItemCommand>
{
  constructor(
    private readonly repository: HeroRepository,
    private readonly publisher: EventPublisher,
    @Inject(REQUEST) public readonly context: AsyncContext,
  ) {}

  async execute(command: ScopedDropAncientItemCommand) {
    Logger.debug('DropAncientItemHandler has been called');

    const { heroId, itemId } = command;
    const hero = this.publisher.mergeObjectContext(
      await this.repository.findOneById(+heroId),
      this.context,
    );
    hero.addItem(itemId);
    hero.commit();
  }
}
```

```typescript
import { ScopedHeroFoundItemHandler } from './scoped-hero-found-item.handler';
import { ScopedHeroKilledDragonHandler } from './scoped-hero-killed-dragon.handler';

export const EventHandlers = [
  ScopedHeroKilledDragonHandler,
  ScopedHeroFoundItemHandler,
];
```

```typescript
import { Inject, Logger, Scope } from '@nestjs/common';
import { REQUEST } from '@nestjs/core';
import { AsyncContext, EventsHandler, IEventHandler } from '../../../../../src';
import { ScopedHeroKilledDragonEvent } from '../impl/hero-killed-dragon.event';

@EventsHandler(ScopedHeroKilledDragonEvent, {
  scope: Scope.REQUEST,
})
export class ScopedHeroKilledDragonHandler
  implements IEventHandler<ScopedHeroKilledDragonEvent>
{
  constructor(@Inject(REQUEST) public readonly context: AsyncContext) {}

  handle(event: ScopedHeroKilledDragonEvent) {
    Logger.debug('HeroKilledDragonHandler has been called');
  }
}
```

```typescript
import { Inject, Logger, Scope } from '@nestjs/common';
import { REQUEST } from '@nestjs/core';
import { AsyncContext, EventsHandler, IEventHandler } from '../../../../../src';
import { ScopedHeroFoundItemEvent } from '../impl/hero-found-item.event';

@EventsHandler(ScopedHeroFoundItemEvent, {
  scope: Scope.REQUEST,
})
export class ScopedHeroFoundItemHandler
  implements IEventHandler<ScopedHeroFoundItemEvent>
{
  constructor(@Inject(REQUEST) public readonly context: AsyncContext) {}

  handle(event: ScopedHeroFoundItemEvent) {
    Logger.log('HeroFoundItemEvent...');
  }
}
```

```typescript
import { ScopedGetHeroesHandler } from './scoped-get-heroes.handler';

export const QueryHandlers = [ScopedGetHeroesHandler];
```

```typescript
import { Inject, Logger, Scope } from '@nestjs/common';
import { REQUEST } from '@nestjs/core';
import { AsyncContext, IQueryHandler, QueryHandler } from '../../../../../src';
import { HeroRepository } from '../../repository/hero.repository';
import { ScopedGetHeroesQuery } from '../impl';

@QueryHandler(ScopedGetHeroesQuery, {
  scope: Scope.REQUEST,
})
export class ScopedGetHeroesHandler
  implements IQueryHandler<ScopedGetHeroesQuery>
{
  constructor(
    private readonly repository: HeroRepository,
    @Inject(REQUEST) public readonly context: AsyncContext,
  ) {}

  async execute(query: ScopedGetHeroesQuery) {
    Logger.debug('GetHeroesQuery has been called');
    return this.repository.findAll();
  }
}
```

```typescript
import { Injectable, Logger } from '@nestjs/common';
import { Observable } from 'rxjs';
import { delay, map } from 'rxjs/operators';
import { AsyncContext, ICommand, ofType, Saga } from '../../../../src';
import { ScopedDropAncientItemCommand } from '../commands/impl/scoped-drop-ancient-item.command';
import { ScopedHeroKilledDragonEvent } from '../events/impl/hero-killed-dragon.event';

export const ANCIENT_ITEM_ID = '12456789';

@Injectable()
export class HeroesGameSagas {
  @Saga()
  dragonKilled = (events$: Observable<any>): Observable<ICommand> => {
    return events$.pipe(
      ofType(ScopedHeroKilledDragonEvent),
      delay(1000),
      map((event) => {
        Logger.debug('Inside [HeroesGameSagas] Saga');
        const command = new ScopedDropAncientItemCommand(
          event.heroId,
          ANCIENT_ITEM_ID,
        );
        AsyncContext.merge(event, command);
        return command;
      }),
    );
  };
}
```
