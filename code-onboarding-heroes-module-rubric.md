# Rubric — How Does the NestJS CQRS Heroes Module Actually Work?

## intent_category
Code Onboarding & Comprehension

## difficulty
Hard

---

## Reasoning

| ID | Description | Weight | Rationale | Dependent On |
|---|---|---|---|---|
| reasoning-1 | States that `@CommandHandler(KillDragonCommand)` attaches metadata to the handler class that maps the command class to the handler, and that `CommandBus` reads this metadata at registration time to build its internal handler map | critical | This is the entry point of the entire flow. Without understanding how the command-to-handler mapping works, nothing else makes sense. | |
| reasoning-2 | Explains that `mergeObjectContext()` patches the `publish()` and `publishAll()` methods on the specific hero instance to route through the `EventBus`, replacing the no-op stubs that `AggregateRoot` provides by default | critical | Depends on reasoning-1 because the handler must be understood before its internals can be explained. This is the most commonly misunderstood step — models assume `AggregateRoot` already wires to `EventBus` out of the box. | reasoning-1 |
| reasoning-3 | Explains that `apply()` stages the event in `AggregateRoot`'s internal event list without dispatching it, and that `commit()` is what flushes those staged events by calling `publishAll()` | critical | Depends on reasoning-2 because `publishAll()` only works after `mergeObjectContext()` has patched it. Understanding the staging vs. flushing distinction is critical to the whole event lifecycle. | reasoning-2 |
| reasoning-4 | Explains that `CqrsModule` subscribes to the saga's returned `Observable<ICommand>` at bootstrap, and automatically dispatches every emitted command through the `CommandBus` — the saga does not call `commandBus.execute()` manually | critical | Depends on reasoning-3 because the saga only receives events after `commit()` delivers them through the `EventBus`. A model that thinks the saga dispatches commands manually has fundamentally misread the architecture. | reasoning-3 |
| reasoning-5 | Explains that `@Saga()` is discovered by `CqrsModule` at bootstrap, which calls `dragonKilled` with the global `EventBus` observable stream as `events$` and keeps the subscription alive for the lifetime of the application | critical | Independent because it explains saga wiring at the module level, not the event-flow level. A model can get this right even if it gets reasoning-3 wrong. | |
| reasoning-6 | Explains that `CqrsModule.forRoot()` registers the module as globally scoped, making its exported providers — `CommandBus`, `EventBus`, `QueryBus`, `EventPublisher` — available to all feature modules without each feature module needing to import `CqrsModule` | critical | Independent. This is a standalone NestJS DI concept — global module scoping — that is separate from the event/command flow mechanics. | |
| reasoning-7 | Explains that `Scope.REQUEST` causes NestJS to create a new handler instance per incoming request rather than reusing a singleton, which is required when the handler must carry per-request state | critical | Independent. This is the gateway to understanding the entire scoped variant. Getting this wrong makes reasoning-8 and reasoning-9 impossible to answer correctly. | |
| reasoning-8 | Explains that `AsyncContext` carries per-request contextual data and is passed as a second argument to `mergeObjectContext()` so that events committed by the aggregate are tagged with that request's context, enabling downstream scoped handlers to operate within the same request scope | critical | Depends on reasoning-7 because `AsyncContext` only exists and matters in request-scoped handlers. Without understanding `Scope.REQUEST` first, the purpose of `AsyncContext` cannot be correctly explained. | reasoning-7 |
| reasoning-9 | Explains that `AsyncContext.merge(event, command)` copies the async context from the incoming event onto the new command so that when the `CommandBus` dispatches it, the scoped handler receives the same request context that originated the event | critical | Depends on reasoning-8 because `AsyncContext.merge` is only meaningful once it is clear what `AsyncContext` holds and why it needs to travel with the command across the saga boundary. | reasoning-8 |
| reasoning-10 | Notes that the `// logic` comment in `killEnemy()` and `addItem()` is a placeholder for in-memory state mutation on the aggregate, and that the apply/commit pattern is identical whether the command was triggered by a direct user request or by a saga | bonus | Depends on reasoning-3 because recognising the `// logic` placeholder requires already understanding what `apply()` does. Bonus because it shows deeper reading of the code rather than just answering the stated questions. | reasoning-3 |

---

## Completeness

| ID | Description | Weight | Rationale | Dependent On |
|---|---|---|---|---|
| completeness-1 | Answers what `@CommandHandler` does at the framework level and how `CommandBus` knows which handler to invoke for a given command | critical | Directly asked in the prompt. A response that skips this leaves the developer's first question unanswered. | |
| completeness-2 | Answers what `mergeObjectContext` does to the hero instance and why `AggregateRoot.publish()` alone is not sufficient to deliver events to the `EventBus` | critical | Directly asked in the prompt. This is the most technically nuanced question and must be addressed completely. | |
| completeness-3 | Answers how `HeroesGameSagas` receives events and what happens to the `DropAncientItemCommand` returned from the `map` operator | critical | Directly asked in the prompt. Both halves of this question — how events arrive and what happens to the returned command — must be covered. | |
| completeness-4 | Answers where `CqrsModule` is registered and how its providers reach handlers and sagas inside `HeroesGameModule` without `CqrsModule` appearing in that module's `imports` array | critical | Directly asked in the prompt. A response that just says "it uses global scope" without explaining the NestJS global module mechanism is incomplete. | |
| completeness-5 | Answers what `Scope.REQUEST` does to the handler's lifecycle compared to the default singleton scope | critical | Directly asked in the prompt. Must contrast request-scoped with singleton — stating one without the other is insufficient. | |
| completeness-6 | Answers what `AsyncContext` is and why it is injected and passed through `mergeObjectContext` in the scoped handlers | critical | Directly asked in the prompt. A response that only names `AsyncContext` without explaining what it carries and why it is necessary is incomplete. | |
| completeness-7 | Answers what `AsyncContext.merge(event, command)` does in the scoped saga and why there is no equivalent call in the regular `HeroesGameSagas` | critical | Directly asked in the prompt. The contrast between the scoped and regular saga is the key insight — answering only one side does not satisfy the question. | |

---

## Style

| ID | Description | Weight | Rationale | Dependent On |
|---|---|---|---|---|
| style-1 | Response contains no standalone code blocks — only inline code references such as `apply()`, `AsyncContext`, or `mergeObjectContext()` are acceptable | critical | Code Onboarding golden answers must be plain-language walkthroughs. Standalone code blocks indicate the model is writing a tutorial rather than explaining existing code. | |
| style-2 | Response uses plain instructor-to-new-developer language and does not suggest changes or improvements to any of the provided code files | critical | The prompt is a comprehension task, not a review. Suggestions signal the model has misread the intent category. | |
| style-3 | Explanation traces the full end-to-end runtime flow in sequence: command dispatch → handler → aggregate → commit → EventBus → saga → new command → handler, for both the regular and scoped variants | bonus | Bonus because tracing the full dual-path flow shows exceptional comprehension beyond just answering each question in isolation. | |

---

## Penalty

| ID | Description | Weight | Rationale | Dependent On |
|---|---|---|---|---|
| penalty-1 | States that `apply()` immediately publishes or dispatches the event to the `EventBus` at the moment it is called | penalty | Factually wrong and propagates a fundamental misunderstanding of the staging/commit lifecycle to the new developer. | |
| penalty-2 | States that `publish()` or `publishAll()` on `AggregateRoot` route events to the `EventBus` without mentioning that `mergeObjectContext()` must first patch those methods on the instance | penalty | Omitting the patching step gives the developer a broken mental model — they will write code expecting events to flow without ever calling `mergeObjectContext()`. | |
| penalty-3 | States that `CqrsModule` must be explicitly imported in each feature module for its providers to be available to that module's handlers | penalty | Directly contradicts how `CqrsModule.forRoot()` global scoping works and will mislead the developer into adding unnecessary imports. | |
| penalty-4 | States that the saga manually dispatches `DropAncientItemCommand` by calling `commandBus.execute()` or `commandBus.dispatch()` inside the saga body | penalty | Factually wrong. The saga returns an `Observable<ICommand>` and `CqrsModule` handles dispatch. This is a harmful misconception about the saga contract. | |
| penalty-5 | States that `Scope.REQUEST` creates a new handler instance per event rather than per incoming request | penalty | Factually wrong. Confusing per-event with per-request would lead to incorrect assumptions about handler isolation and context propagation. | |
| penalty-6 | Suggests changes or improvements to any of the provided code files | penalty | Net-new harmful addition that violates the Code Onboarding intent — the task is to explain, not critique. | |
