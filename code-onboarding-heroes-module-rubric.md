# Rubric — How Does the NestJS CQRS Heroes Module Actually Work?

## intent_category
Code Onboarding & Comprehension

## difficulty
Hard

---

## Rubric Evaluation

| # | ID | Section | Description | Weight | Rationale | Dependent On |
|---|---|---|---|---|---|---|
| 1 | reasoning-1 | reasoning | Identifies @CommandHandler as the decorator that maps a command class to its handler using NestJS metadata | critical | Entry point of the entire command flow; missing it means the routing mechanism cannot be explained at all. | |
| 2 | reasoning-2 | reasoning | Explains that mergeObjectContext() patches the publish() method on the hero instance to route events through the EventBus | critical | Patching publish() is the step that wires an aggregate to the EventBus; skipping it leaves event delivery unexplained. | reasoning-1 |
| 3 | reasoning-3 | reasoning | Explains that apply() stages the event in AggregateRoot's internal list without dispatching it | critical | apply() only stages events internally; missing this creates confusion about why events fail without a subsequent commit(). | reasoning-2 |
| 4 | reasoning-4 | reasoning | Explains that commit() triggers event delivery by calling publishAll() | critical | commit() is what actually sends staged events to the EventBus; without it the delivery sequence is incomplete. | reasoning-3 |
| 5 | reasoning-5 | reasoning | Explains that CqrsModule automatically dispatches every command emitted from the saga's Observable through the CommandBus | critical | Returning an Observable is the saga's entire contract; misrepresenting it leads to structurally broken saga code. | reasoning-4 |
| 6 | reasoning-6 | reasoning | Explains that CqrsModule.forRoot() makes its exported providers globally available to all feature modules | critical | Global registration is why feature modules need no explicit CqrsModule import; leaving it unnamed perpetuates one of the most common NestJS misconceptions. | |
| 7 | reasoning-7 | reasoning | Explains that Scope.REQUEST causes NestJS to create a fresh handler instance for each incoming request | critical | Per-request instantiation is the defining behavior of Scope.REQUEST; naming the scope without the lifecycle explanation gives an incomplete picture. | |
| 8 | reasoning-8 | reasoning | Explains that AsyncContext.merge copies the request context from the incoming event onto the outgoing command | critical | Context loss at the saga boundary silently breaks request isolation; naming the merge call explains how that isolation is preserved. | reasoning-7 |
| 9 | reasoning-9 | reasoning | Notes that the // logic comment in killEnemy() is a placeholder for in-memory state mutation | bonus | Recognising the placeholder shows careful reading of killEnemy() and understanding that apply() records events without mutating aggregate state. | reasoning-3 |
| 10 | reasoning-10 | reasoning | Incorrectly states that apply() immediately dispatches the event to the EventBus when it is called | penalty | Fundamentally wrong about event staging; accepting this means commit() is never understood as necessary. | |
| 11 | reasoning-11 | reasoning | Incorrectly states that the saga manually dispatches commands by calling commandBus.execute() inside its body | penalty | The saga's only job is to return an Observable; stating otherwise leads to saga code that fundamentally breaks the pattern. | |
| 12 | completeness-1 | completeness | Explains the mechanism by which a dispatched command reaches its designated handler | critical | Command routing is the opening question; leaving it unanswered means the core mechanism goes completely unexplained. | |
| 13 | completeness-2 | completeness | Identifies what mergeObjectContext does to the hero instance | critical | publish() patching must be named explicitly; vague references to mergeObjectContext leave the event delivery mechanism unexplained. | |
| 14 | completeness-3 | completeness | Identifies the source of the events$ stream that HeroesGameSagas receives | critical | Without tracing events$ back to CqrsModule, the saga's connection to the rest of the system remains unexplained. | |
| 15 | completeness-4 | completeness | Identifies what happens to DropAncientItemCommand after it is returned from the saga's map operator | critical | Automatic dispatch by CqrsModule is the key handoff; leaving it unnamed disconnects the saga's return value from what runs next. | |
| 16 | completeness-5 | completeness | Describes what Scope.REQUEST does to the handler's instantiation lifecycle | critical | Per-request instantiation is the behavior being described; naming Scope.REQUEST without explaining the lifecycle gives an incomplete answer. | |
| 17 | completeness-6 | completeness | Describes what AsyncContext.merge does inside the scoped saga | critical | AsyncContext.merge is the only mechanism that preserves request context across the saga boundary; skipping it leaves the scoped flow unexplained. | |
| 18 | style-1 | style | Presents code references inline rather than in standalone code blocks | critical | Standalone code blocks signal demonstration rather than explanation; onboarding walkthroughs should be in plain language. | |
| 19 | style-2 | style | Communicates in plain language suitable for a new team member | critical | Dense technical prose without plain-language framing misses the target audience of an onboarding walkthrough. | |
| 20 | style-3 | style | Traces the complete end-to-end runtime flow from command dispatch through to the final handler | bonus | Walking through the full sequence in order shows the response synthesised the entire system rather than answering each question in isolation. | |
