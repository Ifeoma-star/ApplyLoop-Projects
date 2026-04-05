# Rubric — How Does the NestJS CQRS Heroes Module Actually Work?

## intent_category
Code Onboarding & Comprehension

## difficulty
Hard

---

## Rubric Evaluation

| # | ID | Section | Description | Weight | Rationale | Dependent On |
|---|---|---|---|---|---|---|
| 1 | reasoning-1 | reasoning | Identifies @CommandHandler as the decorator that maps a command class to its handler using NestJS metadata | critical | The command-to-handler mapping is the entry point of the entire flow. A developer who cannot explain this cannot explain how anything gets called. | |
| 2 | reasoning-2 | reasoning | Explains that mergeObjectContext() patches the publish() method on the hero instance to route events through the EventBus | critical | Many developers assume AggregateRoot handles EventBus routing by default. Identifying the patching step is what makes the event delivery mechanism visible. | reasoning-1 |
| 3 | reasoning-3 | reasoning | Explains that apply() stages the event in AggregateRoot's internal list without dispatching it | critical | A developer who misses this will not understand why events fail to arrive at handlers when apply() is called without a subsequent commit(). | reasoning-2 |
| 4 | reasoning-4 | reasoning | Explains that commit() triggers event delivery by calling publishAll() | critical | Without understanding commit(), a developer cannot explain what actually sends events to the EventBus after they have been staged. | reasoning-3 |
| 5 | reasoning-5 | reasoning | Explains that CqrsModule automatically dispatches every command emitted from the saga's Observable through the CommandBus | critical | Getting the saga contract wrong produces structurally broken code. The developer must know that returning an Observable is the full extent of the saga's job. | reasoning-4 |
| 6 | reasoning-6 | reasoning | Explains that CqrsModule.forRoot() makes its exported providers globally available to all feature modules | critical | Developers new to NestJS commonly assume each feature module must import CqrsModule individually. Knowing about global registration removes one of the most frequent points of confusion. | |
| 7 | reasoning-7 | reasoning | Explains that Scope.REQUEST causes NestJS to create a fresh handler instance for each incoming request | critical | The entire scoped variant rests on this lifecycle difference. Without it, the purpose of injecting AsyncContext into the handler makes no sense. | |
| 8 | reasoning-8 | reasoning | Explains that AsyncContext.merge copies the request context from the incoming event onto the outgoing command | critical | Skipping this call at the saga boundary silently breaks request isolation. Knowing why it exists is what prevents that class of bug entirely. | reasoning-7 |
| 9 | reasoning-9 | reasoning | Notes that the // logic comment in killEnemy() is a placeholder for in-memory state mutation | bonus | Recognising this shows the model read carefully beyond the call chain and understands that apply() records an event without mutating the aggregate's own state. | reasoning-3 |
| 10 | reasoning-10 | reasoning | Incorrectly states that apply() immediately dispatches the event to the EventBus when it is called | penalty | A developer who accepts this will never call commit() and will waste hours debugging events that were silently dropped. | |
| 11 | reasoning-11 | reasoning | Incorrectly states that the saga manually dispatches commands by calling commandBus.execute() inside its body | penalty | The saga's only job is to return an Observable. Stating otherwise leads the developer to write saga code that fundamentally breaks the pattern. | |
| 12 | completeness-1 | completeness | Explains the mechanism by which a dispatched command reaches its designated handler | critical | Without this, the developer's opening question about how the framework routes commands goes completely unanswered. | |
| 13 | completeness-2 | completeness | Identifies what mergeObjectContext does to the hero instance | critical | The patching of publish() must be named explicitly. Leaving it vague means the event delivery mechanism remains unexplained. | |
| 14 | completeness-3 | completeness | Identifies the source of the events$ stream that HeroesGameSagas receives | critical | Without tracing events$ back to CqrsModule, the developer has no way to understand how the saga connects to the rest of the system. | |
| 15 | completeness-4 | completeness | Identifies what happens to DropAncientItemCommand after it is returned from the saga's map operator | critical | The automatic dispatch by CqrsModule is the key detail the developer is asking about. Leaving it unnamed leaves the question only half answered. | |
| 16 | completeness-5 | completeness | Describes what Scope.REQUEST does to the handler's instantiation lifecycle | critical | The developer asked this specifically. A response that names Scope.REQUEST without explaining the per-request instantiation does not give a useful answer. | |
| 17 | completeness-6 | completeness | Describes what AsyncContext.merge does inside the scoped saga | critical | This is the most specific question in the prompt. Glossing over it or skipping it leaves the scoped flow unexplained. | |
| 18 | style-1 | style | Presents code references inline rather than in standalone code blocks | critical | Code Onboarding responses are plain-language walkthroughs. Standalone code blocks signal the model is rewriting or demonstrating rather than explaining. | |
| 19 | style-2 | style | Communicates in plain language suitable for a new team member | critical | The prompt is a comprehension task aimed at someone new to the codebase. A response written in dense technical prose without context misses the audience entirely. | |
| 20 | style-3 | style | Traces the complete end-to-end runtime flow from command dispatch through to the final handler | bonus | Walking through the full sequence in order shows the model synthesised the entire system rather than answering each question in isolation. | |
