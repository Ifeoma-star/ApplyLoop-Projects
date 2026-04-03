# Rubric — What Is the Hero Model Doing With AggregateRoot and apply()?

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
      "criterion": "States that apply() stages the event in an internal list (INTERNAL_EVENTS) and does NOT immediately dispatch it to the EventBus at the moment it is called"
    },
    {
      "id": "R2",
      "weight": "critical",
      "depends_on": "R1",
      "criterion": "Identifies commit() as the method that flushes all staged events by calling publishAll(), which calls publish() for each one"
    },
    {
      "id": "R3",
      "weight": "critical",
      "depends_on": "R2",
      "criterion": "Explains that publish() and publishAll() are no-op stubs on AggregateRoot by default and only become functional after EventPublisher.mergeObjectContext() patches the specific instance"
    },
    {
      "id": "R4",
      "weight": "critical",
      "criterion": "Explains that AggregateRoot's role is to provide domain event tracking — maintaining the internal event list and exposing the apply/commit lifecycle"
    },
    {
      "id": "R5",
      "weight": "critical",
      "criterion": "Explains that the '// logic' comment is a placeholder for in-memory state mutation on the aggregate (updating its own fields) that the developer is expected to implement"
    },
    {
      "id": "R6",
      "weight": "critical",
      "criterion": "Explains that HeroKilledDragonEvent is created inside killEnemy() rather than in the handler because the aggregate owns its domain logic — the handler should not need to know which event type maps to which operation"
    },
    {
      "id": "R7",
      "weight": "bonus",
      "depends_on": ["R1", "R2"],
      "criterion": "Describes the full sequencing: killEnemy() runs → apply() stages the event → the command handler calls commit() → publishAll() delivers events to the EventBus → registered handlers and sagas receive them"
    },
    {
      "id": "R8",
      "weight": "bonus",
      "depends_on": "R5",
      "criterion": "Clarifies that state mutation in the // logic placeholder would modify the hero's in-memory fields, and that persistence to a database happens separately via a repository"
    }
  ],
  "completeness": [
    {
      "id": "C1",
      "weight": "critical",
      "criterion": "Addresses what AggregateRoot gives the Hero class"
    },
    {
      "id": "C2",
      "weight": "critical",
      "criterion": "Addresses what apply() does when called inside killEnemy() or addItem()"
    },
    {
      "id": "C3",
      "weight": "critical",
      "criterion": "Addresses what the '// logic' comment means — whether it is a placeholder or actual working logic"
    },
    {
      "id": "C4",
      "weight": "critical",
      "criterion": "Addresses why HeroKilledDragonEvent is created inside the model method rather than in the calling handler"
    },
    {
      "id": "C5",
      "weight": "bonus",
      "criterion": "Notes that both killEnemy() and addItem() follow the same apply() pattern, indicating this is the standard way all domain events are recorded on this aggregate"
    }
  ],
  "style": [
    {
      "id": "S1",
      "weight": "critical",
      "criterion": "Response contains no standalone code blocks — only inline code references such as `apply()` or `INTERNAL_EVENTS` are acceptable"
    },
    {
      "id": "S2",
      "weight": "critical",
      "criterion": "Response uses plain, instructor-to-new-developer language and does not suggest changes or improvements to the code"
    },
    {
      "id": "S3",
      "weight": "bonus",
      "criterion": "Explanation proceeds in a logical order that mirrors the call sequence (apply → commit → publish) rather than jumping around between concepts"
    }
  ],
  "penalty": [
    {
      "id": "P1",
      "weight": "penalty",
      "criterion": "States that apply() immediately publishes or dispatches the event to the EventBus at the moment it is called"
    },
    {
      "id": "P2",
      "weight": "penalty",
      "criterion": "States that commit() is invoked automatically by apply() without any external trigger from the handler"
    },
    {
      "id": "P3",
      "weight": "penalty",
      "criterion": "States that the '// logic' comment represents actual working logic already present in the class"
    },
    {
      "id": "P4",
      "weight": "penalty",
      "criterion": "Claims that publish() or publishAll() route events to the EventBus without mentioning that mergeObjectContext() must first patch those methods on the instance"
    },
    {
      "id": "P5",
      "weight": "penalty",
      "criterion": "Suggests changes or improvements to the Hero class code"
    }
  ]
}
```
