# Rubric — What Is This Saga File Actually Doing?

## intent_category
Code Onboarding & Comprehension

## difficulty
Medium

## rubric

```json
{
  "reasoning": [
    {
      "id": "R1",
      "weight": "critical",
      "criterion": "Identifies @Saga() as a decorator from @nestjs/cqrs (not standard NestJS) that CqrsModule uses to auto-discover saga properties at bootstrap and subscribe to them"
    },
    {
      "id": "R2",
      "weight": "critical",
      "criterion": "Explains that events$ is the global EventBus observable stream — every event published anywhere in the system flows through it, not just HeroKilledDragonEvent"
    },
    {
      "id": "R3",
      "weight": "critical",
      "depends_on": "R2",
      "criterion": "Explains that ofType(HeroKilledDragonEvent) is a class-based filter operator from @nestjs/cqrs that discards all events whose class does not match HeroKilledDragonEvent"
    },
    {
      "id": "R4",
      "weight": "critical",
      "criterion": "Explains that delay(1000) is an RxJS operator that shifts each event forward by 1,000 milliseconds in the observable timeline — it is not a synchronous sleep, a blocking call, or a setTimeout"
    },
    {
      "id": "R5",
      "weight": "critical",
      "criterion": "Explains that the DropAncientItemCommand emitted from the map operator is automatically dispatched through the CommandBus by CqrsModule — the saga does not call commandBus.dispatch() manually"
    },
    {
      "id": "R6",
      "weight": "critical",
      "depends_on": "R1",
      "criterion": "Explains that CqrsModule calls dragonKilled with the global events$ stream and subscribes to the returned Observable<ICommand>, keeping the saga alive for the lifetime of the application"
    },
    {
      "id": "R7",
      "weight": "bonus",
      "depends_on": "R1",
      "criterion": "Notes that dragonKilled is declared as an arrow function property assigned with =, not a class method, which ensures lexical this binding"
    },
    {
      "id": "R8",
      "weight": "bonus",
      "criterion": "Identifies the saga's role as a process manager that decouples HeroKilledDragonEvent from its downstream DropAncientItemCommand side effect — the event producer does not need to know about the command"
    }
  ],
  "completeness": [
    {
      "id": "C1",
      "weight": "critical",
      "criterion": "Answers what @Saga() means and why the decorator exists"
    },
    {
      "id": "C2",
      "weight": "critical",
      "criterion": "Answers what the dragonKilled property is and who calls it"
    },
    {
      "id": "C3",
      "weight": "critical",
      "criterion": "Answers who feeds events into events$ — specifically, the EventBus via CqrsModule's subscription setup"
    },
    {
      "id": "C4",
      "weight": "critical",
      "criterion": "Answers what ofType does to the stream"
    },
    {
      "id": "C5",
      "weight": "critical",
      "criterion": "Answers what happens to the DropAncientItemCommand returned from the map operator"
    },
    {
      "id": "C6",
      "weight": "bonus",
      "criterion": "Explains why the saga exists as a separate class from command and event handlers — it manages reactive chains between events and commands, not individual event processing"
    }
  ],
  "style": [
    {
      "id": "S1",
      "weight": "critical",
      "criterion": "Response contains no standalone code blocks — only inline code references such as `@Saga()`, `ofType`, or `events$` are acceptable"
    },
    {
      "id": "S2",
      "weight": "critical",
      "criterion": "Response uses plain, instructor-to-new-developer language and does not suggest changes or improvements to the saga code"
    },
    {
      "id": "S3",
      "weight": "bonus",
      "criterion": "Explanation traces the full runtime flow from HeroKilledDragonEvent being published to DropAncientItemCommand being dispatched, in sequence"
    }
  ],
  "penalty": [
    {
      "id": "P1",
      "weight": "penalty",
      "criterion": "States that delay(1000) is equivalent to setTimeout, a blocking sleep, or a synchronous pause rather than an RxJS stream operator"
    },
    {
      "id": "P2",
      "weight": "penalty",
      "criterion": "States that the saga manually dispatches DropAncientItemCommand by calling commandBus.dispatch() or similar inside the saga body"
    },
    {
      "id": "P3",
      "weight": "penalty",
      "criterion": "States that @Saga() is a standard NestJS decorator available outside of the @nestjs/cqrs package"
    },
    {
      "id": "P4",
      "weight": "penalty",
      "criterion": "States that events$ only contains HeroKilledDragonEvent events, implying the stream is pre-filtered before reaching the saga"
    },
    {
      "id": "P5",
      "weight": "penalty",
      "criterion": "Suggests changes or improvements to the HeroesGameSagas code"
    }
  ]
}
```
