# **Code QA Project Overview**

## **What We're Building**

This evaluation dataset assesses natural-language reasoning, explanation quality, and technical judgment in coding-related interactions. Tasks are non-verifiable (no executable correctness checks) and focus on how well a model can explain, debug, recommend, and reason about code.

Task Complexity

To ensure task complexity, tasks should involve multi-file contexts, architectural decisions, and real-world messiness rather than simple isolated functions.

Tasks should be designed to achieve meaningful discrimination between model capabilities. If pass rates exceed 80%, the task is likely too easy. Target 40-70% pass rates on critical criteria for well-calibrated difficulty.

## **Required Components per Task**

Input Context → `prompt` — Prompts in line with workflows: this may elicit inspection of initial Issue \+ PR pair, an artifact within the repo, etc.

Verifiers → `rubric` — A fully comprehensive rubric evaluating the plain-text model response.

Verifiers → `golden_answer` — A "perfect" expert-defined answer over their given prompt.

## **The Core Principle**

The goal is to create tasks that meaningfully discriminate between model capabilities. Every task should:

* Be self-contained with all necessary context  
* Target realistic developer scenarios  
* Include rubrics that are specific and measurable  
* Avoid tasks that are too easy (\>80% pass) or impossibly hard

Difficulty Calibration

Verify your difficulty tag actually matches what the task requires. If you marked something Hard but it's just a simple bug fix, we will downgrade it. If you marked something Medium but it requires deep domain expertise and architectural decisions, we will upgrade it.  
We will then give you access to the documentation so you can begin.

## **Task Structure**

You should craft an original prompt based on the assigned language and category. All necessary context (code snippets, error messages, logs, etc.) should be included directly in the prompt to make it fully self-contained.

### **Error Messages and Stack Traces**

* Raw error output from compilers, interpreters, or runtime exceptions. Prompts should ask for diagnosis, root cause analysis, or resolution steps.  
* Specs: 20–200 lines; include framework versions, triggering code, and 2+ related files

### **Buggy or Incomplete Code**

* Code containing errors, logic flaws, or missing sections. Prompts should ask the model to identify issues, explain what's wrong, or suggest fixes.  
* Specs: 100–500 lines; 3–6 distinct bugs or gaps; MUST span multiple files

### **Code Snippets**

* Working or partial code in any supported language. Prompts may ask for explanations, optimizations, refactoring suggestions, or code reviews.  
* Specs: 200–1000 lines of code; multi-file context REQUIRED; show imports, dependencies, inheritance

### **Logs**

* Application, server, or system logs (e.g., debug output, access logs, build logs). Prompts may ask for interpretation, anomaly detection, or troubleshooting guidance.  
* Specs: 50–500 lines; include timestamps, log levels; require correlating multiple log sources

### **Commands**

* Shell commands, CLI invocations, or scripts. Prompts may ask for explanations of what a command does, how to modify it, or how to troubleshoot failures.  
* Specs: 10–100 lines; include environment context, multi-stage build processes

### **Configuration Snippets**

* Config files such as YAML, JSON, TOML, INI, or environment variables. Prompts may ask for validation, best practices, or help resolving misconfigurations.  
* Specs: 50–300 lines; MUST include multiple related configs (e.g., Terraform \+ Kubernetes \+ CI/CD)

### **Text-only Descriptions**

* Natural language descriptions of a problem, requirement, or scenario without accompanying code. Prompts should ask for implementation guidance, architectural advice, or approach recommendations.  
* Specs: 200–800 words; rich technical context; architectural constraints; performance requirements

### **No Input (Conceptual/Advisory)**

* General questions with no specific code or context provided. Prompts cover concepts, best practices, trade-offs, or technology comparisons.  
* Specs: 2–8 sentences; focused question requiring deep domain knowledge AND version-specific details

## **Prompt Intent Categories**

Each prompt should fall into one of the following intent categories:

### **Root Cause Analysis**

* Why observed behavior is broken, unexpected, or failing — traced through the codebase. The primary deliverable is a diagnosis of what went wrong and why.  
* Example prompts: "This function returns None intermittently but I can't reproduce it locally. What's happening?" · "Why does this test pass in isolation but fail in the suite?"

### **Code Onboarding & Comprehension**

* Plain-language walkthrough of what existing code, files, or functions do. The primary deliverable is a narrative explanation of what something does, not why it fails or how to change it.  
* Example prompts: "I just joined this project, what does this module actually do?" · "What's the entry point for this service and how does execution flow from there?"

### **Code Debugging & Repair**

* Identify and fix a known or suspected problem. Unlike Root Cause Analysis, the cause is already known or suspected, but the task is producing the actual fix.  
* Example prompts: "Here's the stack trace — can you fix the function causing this?" · "This SQL query returns duplicates when a user has multiple roles, can you fix it?"

### **Architecture & System Design**

* How existing components are structured and interact, why design decisions were made, and how to plan or design systems going forward.  
* Example prompts: "How does authentication flow through this middleware stack?" · "We need to add a caching layer to this service…how should we design it?"

### **PR Triage & Assessment**

* Explain a code change, assess blast radius, classify severity/complexity, flag security risk.  
* Example prompts: "What does this PR actually change and is anything likely to break?" · "This diff touches the auth module; what are the security implications?"

# **Prompt Guidelines**

## **Writing Style for Prompts**

Prompts should read like realistic peer-to-peer developer requests—direct, context-rich, and conversational. Avoid heavy role-play or meta-instructions.

Do NOT use framing such as:

* "You are a..."  
* "Your tasks are..."  
* "Act as a..."

DO write as if you're a developer asking a colleague:

* "I'm getting this error when deploying..."  
* "Can you help me understand why this algorithm..."  
* "I'm trying to decide between X and Y for..."

## 

## **Self-Contained Context**

All necessary context (code snippets, error messages, logs, requirements, system information) must be included directly in the prompt. The model should not need to ask for additional information to provide a substantive answer.

## **Task Structure**

The expert should craft an original prompt based on the assigned language and category. All necessary context (code snippets, error messages, logs, etc.) should be included directly in the prompt to make it fully self-contained.

## **Prompt Intent Categories**

Each prompt should fall into one of the following intent categories:

## **Guidelines for Quality and Complexity**

### **1\. Multi-File Context**

Realistic project structure:

* 500-1,000+ lines across multiple files (e.g., models, services, controllers, config) reflecting how real codebases are organized  
* Bugs spanning file boundaries (incorrect imports, mismatched interfaces, inconsistent type definitions across modules)

Deeply nested logic:

* Complex control flow, callbacks within callbacks, intricate state management  
* Async/await chains across functions, non-obvious state machine transitions

Multi-step causal chains:

* Root cause is far from symptom—tracing through 5+ function calls across files to identify the actual issue  
* Non-obvious interactions between middleware, services, and data layers

### 

### **2\. Mixed-Language Codebases**

* Python calling C extensions, JS with inline SQL, Rust \+ FFI bindings, C++ with Emscripten/WebAssembly \+ JS wrappers  
* Cross-language bugs (memory management at boundaries, type mismatches, encoding issues between systems)  
* Shell scripts \+ Dockerfiles \+ application code \+ SQL migrations interacting in non-obvious ways  
* Configuration wiring across infrastructure layers (e.g., ARM templates \+ KQL \+ API config)

### 

### **3\. New Difficult Prompt Categories**

Specialized domains:

* FPGA HDL, compiler IR representations, RTOS internals, custom DSLs  
* SIEM connector configuration (e.g., Azure Sentinel), information retrieval pipelines (BM25 \+ vector search)  
* Security exploit analysis (ASLR bypass, heap exploitation)

Advanced data structures and algorithms:

* Segment trees with lazy propagation, rollback DSU for dynamic connectivity  
* Augmented AVL trees with subtree aggregates, lock-free concurrent data structures

Performance analysis:

* Cache behavior, memory access patterns, algorithmic complexity tradeoffs  
* Latency correlation across distributed components, WebAssembly vs native performance profiling

Compound/interacting bugs:

* Multiple distinct bugs that interact or mask each other  
* E.g., a race condition that only manifests because of a separate caching bug, or cross-tenant data leakage caused by the intersection of mutable shared state and async execution

Niche library internals:

* Obscure or deprecated APIs (not easily web-searchable)  
* Version-specific quirks: "Why does this work in React 18.2.0 but break in 18.3.0?"  
* Lesser-known framework adapters (e.g., LangChain's OpenAI-compatible adapter for non-OpenAI providers)

### 

### **4\. Red Herrings**

Suspicious-looking code that isn't the issue:

* Outdated comments, unrelated warnings, complex-but-correct logic adjacent to the actual bug  
* Long, noisy logs with the relevant error buried in 200-500 lines of output

User's stated suspicion is wrong:

* The user points toward one cause (e.g., "I think it's a heap exploit issue") when the real problem is elsewhere (e.g., an unbounded write loop)  
* The model should independently diagnose rather than follow the user's lead

Misleading symptoms:

* All tests pass but production fails  
* The app appears to do nothing (silent failure with no errors)  
* Two different errors in different environments where only one is the real root cause

Wrong metric comparisons:

* User compares apples to oranges (e.g., Recall@10 vs nDCG@10) and expects similar values, leading them to believe something is broken when it may not be

### 

### **5\. Incomplete or Ambiguous Prompt Context**

Partial stack traces or logs:

* Truncated error output requiring the model to state what assumptions it's making  
* Intermittent bugs described as "sometimes crashes" or "works locally but fails in CI" without clear reproduction steps

Ambiguous requirements:

* Issues where it's unclear whether observed behavior is a bug, an acceptable tradeoff, or a design decision  
* Rubric rewards acknowledging ambiguity, penalizes confident wrong assumptions

Broad conceptual questions:

* Questions with wide scope where the specific confusion isn't pinpointed (e.g., "How does X work?" without specifying which aspect)

Conflicting constraints:

* Trade-offs that must be navigated (e.g., "optimize for both memory and speed")  
* The model should surface the tension rather than silently pick one

Domain-specific configs:

* Kubernetes YAML, Terraform HCL, Nginx/Apache with non-obvious interactions where missing context changes the correct answer

# **Rubric Guidelines**

We're moving away from just "major" and "minor" weights. Instead, use three types:

	critical — These are must-haves. If a response misses one of these, the answer is fundamentally wrong or incomplete. Think: identifying the actual root cause, proposing a solution that won't break things, addressing the core requirement.

	bonus — These separate good answers from great ones. Things like: discussing edge cases, explaining tradeoffs between approaches, providing a testing strategy, considering performance implications.

	penalty — These are red flags. Responses that do these things are actively harmful or misleading. Examples: suggesting code that breaks existing functionality, stating incorrect facts with confidence, proposing security vulnerabilities.

## 

## **Rubric Structure**

Your rubric has three sections:

* reasoning \- Does the response correctly understand and explain what's happening?  
* completeness \- Does it address all parts of the question and provide enough detail?  
* style \- Is it clear, well-organized, and professional?

Each criterion looks like this:

{  
  "id": "reasoning-1",  
  "description": "Identifies that command-line arguments are passed as strings, not native types",  
  "weight": "critical",  
  "rationale": "This is the root cause \- missing this means the entire diagnosis is wrong",  
  "dependent\_on": \[\]  
}

## **Dependency Chains**

Some criteria only make sense if earlier ones are satisfied. Use `dependent_on` to capture this:

{  
  "id": "reasoning-3",  
  "description": "Proposes correct fix based on the root cause",  
  "weight": "critical",  
  "dependent\_on": \["reasoning-1"\]  
}

If `reasoning-1` isn't satisfied, `reasoning-3` becomes N/A automatically.

## **Rubric Complexity Guidelines**

### **Target Scope**

* 8-30 total criteria for most tasks  
* More complex tasks should have proportionally more criteria

## 

## **Critical Criteria Best Practices**

### **Make criteria specific and measurable**

Bad

"provides a good explanation"

Good

* "traces the error through the call stack from CLI handler to validator to type check"  
* "identifies that the race condition occurs between lines 42-47 when thread A reads while thread B writes"

### 

### **Require Precise Terminology**

Critical items should demand exact technical language:

* "Must specifically identify this as a 'use-after-free' vulnerability"  
* "Must mention that this is undefined behavior per C11 section 6.5.2"  
* "Must identify the TOCTOU (time-of-check-to-time-of-use) race condition"

## 

## **Penalty Items**

Include 4-6 penalty items for common model mistakes:

* Confidently stating incorrect information  
* Proposing fixes that would break existing functionality  
* Missing obvious security implications  
* Falling for red herrings in the code  
* Assuming one interpretation of ambiguous requirements without acknowledging alternatives

## 

## **Dependency Chains**

Use dependency chains to create evaluation depth:

* Chain 3-5 criteria together for complex reasoning paths  
* Example: reasoning-1 (identify symptom) \-\> reasoning-2 (trace to cause) \-\> reasoning-3 (explain mechanism) \-\> completeness-1 (propose fix)

## 

## **General Guidelines**

All rubric criteria must be atomic and self-contained. Each criterion should evaluate exactly one specific point. Instead of combining multiple requirements into a single criterion, break them into separate, individual criteria within your rubric.

Bad (Non-atomic)

{  
  "id": "reasoning-1",  
  "description": "Identifies the root cause as a race condition between lines 42-47 and explains why thread synchronization is needed",  
  "weight": "critical"  
}

Good (Atomic)

{  
  "id": "reasoning-1",  
  "description": "Identifies the root cause as a race condition occurring between lines 42-47",  
  "weight": "critical"  
}

{  
  "id": "reasoning-2",  
  "description": "Explains why thread synchronization is needed to resolve the race condition",  
  "weight": "critical",  
  "dependent\_on": \["reasoning-1"\]  
}

# **Example: AXI4 Interface Design (Text-only Input)**

This example demonstrates a text-only prompt where no code is provided—just a technical question requiring domain expertise.

## **prompt\_statement**

I'm working on an FPGA design using AXI4 and AXI4-Lite interfaces and something about the channel structure is confusing me.

Both AXI4 and AXI4-Lite have five channels:

* Read Address Channel  
* Write Address Channel  
* Read Data Channel  
* Write Data Channel  
* Write Response Channel

I understand why there's a Write Response Channel—the master needs confirmation that the write completed. But why isn't there a Read Response Channel? The master sends a read address and gets data back, but there's no explicit response channel saying "read completed successfully" or "read failed."

This seems asymmetric. Write operations get address \-\> data \-\> response, but read operations just get address \-\> data with no response. How does the master know if a read failed or if the data is valid? Is the response somehow embedded in the Read Data Channel, or is there a design reason for this asymmetry?

## **golden\_answer**

The response is in the Read Data Channel—it's just not a separate channel.

### **How AXI4 Handles Read Responses**

The Read Data Channel includes a `RRESP` signal (read response) that travels with each data beat. So when the slave sends read data back, it simultaneously sends the response status on the same channel.

For writes, you have:

* Write Address Channel: master \-\> slave (address \+ control)  
* Write Data Channel: master \-\> slave (data)  
* Write Response Channel: slave \-\> master (status)

For reads, you have:

* Read Address Channel: master \-\> slave (address \+ control)  
* Read Data Channel: slave \-\> master (data \+ status)

The `RRESP` signal on the Read Data Channel serves the same purpose as the Write Response Channel—it indicates success, error, decode error, etc.

### **Why the Asymmetry?**

It's about data flow efficiency. With writes, the data goes from master to slave, so you need a separate channel for the response to come back. With reads, the data is already flowing from slave to master, so the response just piggybacks on that same channel.

If there were a separate Read Response Channel, it would be redundant—you'd be sending status information twice (once with the data, once on a separate channel) or you'd need to delay sending the data until after a separate response, which would add latency.

### **Signal Details**

On the Read Data Channel, you'll see:

* `RDATA`: the actual data  
* `RRESP[1:0]`: 2-bit response (OKAY, EXOKAY, SLVERR, DECERR)  
* `RVALID`: indicates data and response are valid  
* `RREADY`: master signals it can accept the data  
* `RLAST`: indicates last beat in a burst

So the master knows a read failed by checking `RRESP` when `RVALID` is asserted. If `RRESP` is anything other than OKAY or EXOKAY, the read had an error.

### **Design Principle**

AXI follows a principle of minimizing channels while maintaining full status reporting. Since read data must flow slave \-\> master anyway, embedding the response in that channel eliminates the need for a sixth channel without losing any functionality.

## **rubric**

{  
  "reasoning": \[  
    {  
      "id": "reasoning-1",  
      "description": "Explains that the read response is embedded in the Read Data Channel via the RRESP signal",  
      "weight": "critical",  
      "rationale": "This is the core answer to the question \- missing this means failing to address the prompt",  
      "dependent\_on": \[\]  
    },  
    {  
      "id": "reasoning-2",  
      "description": "Explains the asymmetry is due to data flow direction (read data already flows slave \-\> master)",  
      "weight": "critical",  
      "rationale": "Must explain why the design differs between read and write to demonstrate understanding of the architectural decision",  
      "dependent\_on": \["reasoning-1"\]  
    },  
    {  
      "id": "reasoning-3",  
      "description": "Notes that a separate Read Response Channel would be redundant or add latency",  
      "weight": "bonus",  
      "rationale": "Shows understanding of the design efficiency principle behind AXI4 architecture",  
      "dependent\_on": \["reasoning-2"\]  
    },  
    {  
      "id": "reasoning-4",  
      "description": "Explains that RRESP travels with each data beat in a burst",  
      "weight": "bonus",  
      "rationale": "Demonstrates detailed understanding of how response signaling works across burst transactions",  
      "dependent\_on": \["reasoning-1"\]  
    },  
    {  
      "id": "reasoning-5",  
      "description": "Claims there is no way to detect read errors in AXI4",  
      "weight": "penalty",  
      "rationale": "Factually incorrect \- RRESP exists specifically for this purpose and this claim would mislead implementers",  
      "dependent\_on": \[\]  
    },  
    {  
      "id": "reasoning-6",  
      "description": "States that read responses are sent on a separate channel from read data",  
      "weight": "penalty",  
      "rationale": "Fundamental misunderstanding of AXI4 architecture \- RRESP is part of the Read Data Channel",  
      "dependent\_on": \[\]  
    }  
  \],  
  "completeness": \[  
    {  
      "id": "completeness-1",  
      "description": "Mentions the RRESP signal specifically as the mechanism for read responses",  
      "weight": "critical",  
      "rationale": "Must reference the actual AXI4 signal to demonstrate technical accuracy",  
      "dependent\_on": \[\]  
    },  
    {  
      "id": "completeness-2",  
      "description": "References the Read Data Channel as the location where read responses are carried",  
      "weight": "critical",  
      "rationale": "Must identify the specific channel to correctly answer where read responses are communicated",  
      "dependent\_on": \[\]  
    },  
    {  
      "id": "completeness-3",  
      "description": "Contrasts the Write Response Channel structure with the Read Data Channel to explain the asymmetry",  
      "weight": "bonus",  
      "rationale": "Directly addresses the comparison aspect of the prompt and helps clarify the architectural difference",  
      "dependent\_on": \[\]  
    },  
    {  
      "id": "completeness-4",  
      "description": "Explains the RRESP encoding values (OKAY, EXOKAY, SLVERR, DECERR)",  
      "weight": "bonus",  
      "rationale": "Provides actionable detail that would be useful for implementation or debugging",  
      "dependent\_on": \["completeness-1"\]  
    },  
    {  
      "id": "completeness-5",  
      "description": "Mentions other Read Data Channel signals (RDATA, RVALID, RREADY, RLAST)",  
      "weight": "bonus",  
      "rationale": "Demonstrates comprehensive understanding of the channel structure",  
      "dependent\_on": \["completeness-2"\]  
    },  
    {  
      "id": "completeness-6",  
      "description": "Suggests that RRESP and RDATA travel on separate channels or at separate times",  
      "weight": "penalty",  
      "rationale": "Incorrect \- RRESP travels with the data beat on the same channel, and this misconception could lead to implementation errors",  
      "dependent\_on": \[\]  
    }  
  \],  
  "style": \[  
    {  
      "id": "style-1",  
      "description": "Organizes response with clear structure (e.g., answering where responses are, then explaining why)",  
      "weight": "bonus",  
      "rationale": "Makes the explanation easy to follow and helps reader extract information efficiently",  
      "dependent\_on": \[\]  
    },  
    {  
      "id": "style-2",  
      "description": "Uses AXI4 terminology correctly (manager/subordinate or master/slave, beat, burst, channels)",  
      "weight": "bonus",  
      "rationale": "Demonstrates domain knowledge and ensures professional communication with other hardware engineers",  
      "dependent\_on": \[\]  
    },  
    {  
      "id": "style-3",  
      "description": "Provides a direct answer to the question before diving into supporting details",  
      "weight": "bonus",  
      "rationale": "Helps reader get immediate understanding and satisfies the prompt efficiently",  
      "dependent\_on": \[\]  
    }  
  \]  
}

# **Example: Optimize Algorithms (Code Snippet)**

This example demonstrates a code-heavy prompt asking for algorithm optimization suggestions.

## **prompt\_statement**

Please analyze the following Pine Script code and provide specific suggestions on how to improve its algorithm, calculation methods, and overall efficiency. Your response should include actionable advice that can be directly implemented to enhance the script's performance.

// This work is licensed under a Attribution-NonCommercial-ShareAlike 4.0 International (CC BY-NC-SA 4.0)  
// https://creativecommons.org/licenses/by-nc-sa/4.0/  
// © LuxAlgo

//@version=5  
indicator("DTFX Algo Zones \[LuxAlgo\]", shorttitle \= "LuxAlgo \- DTFX Algo Zones", overlay \= true, max\_boxes\_count \= 500, max\_lines\_count \= 500\)

//--------------------------------------------------------------------------------------------------------------------}  
//Inputs  
//--------------------------------------------------------------------------------------------------------------------{

structureTog \= input.bool(true, title \= "Show Swing Points", group \= "Structure")  
structureLen \= input.int(10, title \= "Structure Length", group \= "Structure")

zoneDispNum \= input.int(1, title \= "Show Last", group \= "Zones", minval \= 0\)  
dispAll \= input.bool(true, title \= "Display All Zones", tooltip \= "Ignores Zone Display \# and Displays all Possible Zones.", group \= "Zones")  
zoneFilter \= input.string("Both", title \= "Zone Display", options \= \["Bullish Only", "Bearish Only", "Both"\], group \= "Zones")  
noOverlap \= input.bool(true, title \= "Clean-Up Level Overlap", group \= "Zones")

fib\_group \= "              Fib Levels                    Line Style                 Bull | Bear"

f1Tog \= input.bool(false, title \= "", group \= fib\_group, inline \= "1")  
f2Tog \= input.bool(true, title \= "", group \= fib\_group, inline \= "2")  
f3Tog \= input.bool(true, title \= "", group \= fib\_group, inline \= "3")  
f4Tog \= input.bool(true, title \= "", group \= fib\_group, inline \= "4")  
f5Tog \= input.bool(false, title \= "", group \= fib\_group, inline \= "5")

f1Lvl \= f1Tog?input.float(0, title \= "", group \= fib\_group, inline \= "1"):na  
f2Lvl \= f2Tog?input.float(0.3, title \= "", group \= fib\_group, inline \= "2"):na  
f3Lvl \= f3Tog?input.float(0.5, title \= "", group \= fib\_group, inline \= "3"):na  
f4Lvl \= f4Tog?input.float(0.7, title \= "", group \= fib\_group, inline \= "4"):na  
f5Lvl \= f5Tog?input.float(1, title \= "", group \= fib\_group, inline \= "5"):na

f1Style \= input.string(". . .", title \= "", options \= \["\_\_\_","- \- \-",". . ."\], group \= fib\_group, inline \= "1")  
f2Style \= input.string("- \- \-", title \= "", options \= \["\_\_\_","- \- \-",". . ."\], group \= fib\_group, inline \= "2")  
f3Style \= input.string("\_\_\_", title \= "", options \= \["\_\_\_","- \- \-",". . ."\], group \= fib\_group, inline \= "3")  
f4Style \= input.string("- \- \-", title \= "", options \= \["\_\_\_","- \- \-",". . ."\], group \= fib\_group, inline \= "4")  
f5Style \= input.string(". . .", title \= "", options \= \["\_\_\_","- \- \-",". . ."\], group \= fib\_group, inline \= "5")

f1BullColor \= input.color(\#089981, title \= "", group \= fib\_group, inline \= "1")  
f2BullColor \= input.color(\#089981, title \= "", group \= fib\_group, inline \= "2")  
f3BullColor \= input.color(\#089981, title \= "", group \= fib\_group, inline \= "3")  
f4BullColor \= input.color(\#089981, title \= "", group \= fib\_group, inline \= "4")  
f5BullColor \= input.color(\#089981, title \= "", group \= fib\_group, inline \= "5")

f1BearColor \= input.color(\#f23645, title \= "", group \= fib\_group, inline \= "1")  
f2BearColor \= input.color(\#f23645, title \= "", group \= fib\_group, inline \= "2")  
f3BearColor \= input.color(\#f23645, title \= "", group \= fib\_group, inline \= "3")  
f4BearColor \= input.color(\#f23645, title \= "", group \= fib\_group, inline \= "4")  
f5BearColor \= input.color(\#f23645, title \= "", group \= fib\_group, inline \= "5")

structureColor \= input.color(color.gray, title \= "Structure Color", group \= "Style")  
bullZoneColor \= input.color(color.new(\#089981,80), title \= "Bullish Zone Color", group \= "Style")  
bearZoneColor \= input.color(color.new(\#f23645,80), title \= "Bearish Zone Color", group \= "Style")

//--------------------------------------------------------------------------------------------------------------------}  
//UDTs  
//--------------------------------------------------------------------------------------------------------------------{

type pb  
    float price  
    int bar

type fs  
    line f1  
    line f2  
    line f3  
    line f4  
    line f5

//--------------------------------------------------------------------------------------------------------------------}  
//Functions  
//--------------------------------------------------------------------------------------------------------------------{

linestyle(\_input) \=\>  
    \_input \== "\_\_\_"?line.style\_solid:  
    \_input \== "- \- \-"?line.style\_dashed:  
    \_input \== ". . ."?line.style\_dotted:  
    na

get\_fibs(\_top,\_bot,\_dir,\_fl1,\_fl2,\_fl3,\_fl4,\_fl5) \=\>  
    rng \= \_dir \== 1 ? \_top \- \_bot : \_bot \- \_top  
    anchor \= \_dir \== 1 ? \_bot : \_top  
    fib1 \= anchor \+ (rng\*\_fl1)  
    fib2 \= anchor \+ (rng\*\_fl2)  
    fib3 \= anchor \+ (rng\*\_fl3)  
    fib4 \= anchor \+ (rng\*\_fl4)  
    fib5 \= anchor \+ (rng\*\_fl5)  
    \[fib1,fib2,fib3,fib4,fib5\]

//--------------------------------------------------------------------------------------------------------------------}  
//Calcs  
//--------------------------------------------------------------------------------------------------------------------{

var int dir \= 0  
float top \= na  
float btm \= na

var pb t \= pb.new(na,na)  
var pb b \= pb.new(na,na)

var bool bos\_up\_check \= false  
var bool bos\_down\_check \= false  
var string last\_bot \= "NA"  
var string last\_top \= "NA"

var zones \= array.new\_box()  
var lvls \= array.new\<fs\>()

var box live\_zone \= na  
var fs live\_lvls \= fs.new(na,na,na,na,na)

upper \= ta.highest(structureLen)  
lower \= ta.lowest(structureLen)

if dir \>= 0 and high\[structureLen\] \> upper  
    dir := \-1  
    top := high\[structureLen\]

if dir \<= 0 and low\[structureLen\] \< lower  
    dir := 1  
    btm := low\[structureLen\]

top\_conf \= not na(top)  
bot\_conf \= not na(btm)

int structure\_confirmed \= 0

if top\_conf  
    t := pb.new(top,bar\_index-structureLen)  
    bos\_up\_check := true  
    if structureTog  
        structure\_confirmed := 1

if bot\_conf  
    b := pb.new(btm,bar\_index-structureLen)  
    bos\_down\_check := true  
    if structureTog  
        structure\_confirmed := \-1

plotshape(structure\_confirmed\>0?true:false, style \= shape.circle, title \= "Swing High", location \= location.abovebar, offset \= \-structureLen, color \= structureColor)  
plotshape(structure\_confirmed\<0?true:false, style \= shape.circle, title \= "Swing Low", location \= location.belowbar, offset \= \-structureLen, color \= structureColor)

HH \= top\_conf and t.price \> t.price\[1\]  
HL \= bot\_conf and b.price \> b.price\[1\]  
LH \= top\_conf and t.price \< t.price\[1\]  
LL \= bot\_conf and b.price \< b.price\[1\]

last\_top := HH?"HH":LH?"LH":last\_top  
last\_bot := LL?"LL":HL?"HL":last\_bot

var int t\_dir \= 0

choch\_up \= ta.crossover(close,t.price) and (high \== upper) and t\_dir \<= 0  
choch\_down \= ta.crossunder(close,b.price) and (low \== lower) and t\_dir \>= 0

if choch\_up  
    t\_dir := 1

if choch\_down  
    t\_dir := \-1

bos\_up \= ta.crossover(close,t.price) and bos\_up\_check and t\_dir \>= 0  
bos\_down \= ta.crossunder(close,b.price) and bos\_down\_check and t\_dir \<= 0

mss\_up \= bos\_up or choch\_up  
mss\_down \= bos\_down or choch\_down

if mss\_up  
    bos\_up\_check := false  
    if zoneFilter \!= "Bearish Only"  
        \_top \= t.price  
        \_bot \= dir \== \-1 ? lower : dir \== 1 ? b.price : 0  
        \[fib1,fib2,fib3,fib4,fib5\] \= get\_fibs(\_top,\_bot,1,f1Lvl,f2Lvl,f3Lvl,f4Lvl,f5Lvl)  
          
        live\_zone := box.new(t.bar,\_top,bar\_index,\_bot,bgcolor \= bullZoneColor,border\_color \= na)  
        live\_lvls := fs.new(  
            line.new(t.bar,(f1Tog?fib1:na),bar\_index,fib1, color \= f1BullColor, style \= linestyle(f1Style)),  
            line.new(t.bar,(f2Tog?fib2:na),bar\_index,fib2, color \= f2BullColor, style \= linestyle(f2Style)),  
            line.new(t.bar,(f3Tog?fib3:na),bar\_index,fib3, color \= f3BullColor, style \= linestyle(f3Style)),  
            line.new(t.bar,(f4Tog?fib4:na),bar\_index,fib4, color \= f4BullColor, style \= linestyle(f4Style)),  
            line.new(t.bar,(f5Tog?fib5:na),bar\_index,fib5, color \= f5BullColor, style \= linestyle(f5Style))  
        )  
          
        zones.push(live\_zone)  
        lvls.push(live\_lvls)

if mss\_down  
    bos\_down\_check := false  
    if zoneFilter \!= "Bullish Only"  
        \_top \= dir \== 1 ? upper : dir \== \-1 ? t.price : 0  
        \_bot \= b.price  
        \[fib1,fib2,fib3,fib4,fib5\] \= get\_fibs(\_top,\_bot,-1,f1Lvl,f2Lvl,f3Lvl,f4Lvl,f5Lvl)  
          
        live\_zone := box.new(b.bar,\_top,bar\_index,\_bot,bgcolor \= bearZoneColor,border\_color \= na)  
        live\_lvls := fs.new(  
            line.new(b.bar,(f1Tog?fib1:na),bar\_index,fib1, color \= f1BearColor, style \= linestyle(f1Style)),  
            line.new(b.bar,(f2Tog?fib2:na),bar\_index,fib2, color \= f2BearColor, style \= linestyle(f2Style)),  
            line.new(b.bar,(f3Tog?fib3:na),bar\_index,fib3, color \= f3BearColor, style \= linestyle(f3Style)),  
            line.new(b.bar,(f4Tog?fib4:na),bar\_index,fib4, color \= f4BearColor, style \= linestyle(f4Style)),  
            line.new(b.bar,(f5Tog?fib5:na),bar\_index,fib5, color \= f5BearColor, style \= linestyle(f5Style))  
        )  
          
        zones.push(live\_zone)  
        lvls.push(live\_lvls)

if zones.size() \> zoneDispNum and not dispAll  
    zones.shift().delete()  
    ln \= lvls.shift()  
    ln.f1.delete()  
    ln.f2.delete()  
    ln.f3.delete()  
    ln.f4.delete()  
    ln.f5.delete()

if lvls.size() \> 1 and noOverlap  
    last\_zone \= zones.get(zones.size()-2)  
    last\_lvl \= lvls.get(lvls.size()-2)  
      
    if last\_lvl.f1.get\_x2() \> live\_lvls.f1.get\_x1()  
        last\_lvl.f1.set\_x2(math.max(live\_zone.get\_left(),last\_zone.get\_right()))  
        last\_lvl.f2.set\_x2(math.max(live\_zone.get\_left(),last\_zone.get\_right()))  
        last\_lvl.f3.set\_x2(math.max(live\_zone.get\_left(),last\_zone.get\_right()))  
        last\_lvl.f4.set\_x2(math.max(live\_zone.get\_left(),last\_zone.get\_right()))  
        last\_lvl.f5.set\_x2(math.max(live\_zone.get\_left(),last\_zone.get\_right()))  
          
        live\_lvls.f1.set\_x1(math.max(live\_zone.get\_left(),last\_zone.get\_right()))  
        live\_lvls.f2.set\_x1(math.max(live\_zone.get\_left(),last\_zone.get\_right()))  
        live\_lvls.f3.set\_x1(math.max(live\_zone.get\_left(),last\_zone.get\_right()))  
        live\_lvls.f4.set\_x1(math.max(live\_zone.get\_left(),last\_zone.get\_right()))  
        live\_lvls.f5.set\_x1(math.max(live\_zone.get\_left(),last\_zone.get\_right()))  
          
        live\_lvls.f1.set\_x2(bar\_index)  
        live\_lvls.f2.set\_x2(bar\_index)  
        live\_lvls.f3.set\_x2(bar\_index)  
        live\_lvls.f4.set\_x2(bar\_index)  
        live\_lvls.f5.set\_x2(bar\_index)

//--------------------------------------------------------------------------------------------------------------------}

Consider the following aspects in your analysis:

* Algorithm: Are there more efficient ways to identify swing points, zones, and trend direction?  
* Calculation: Can the Fibonacci level calculations be optimized? Are there redundant calculations?  
* Efficiency: How can the script be made more efficient in terms of execution speed and resource usage (e.g., reducing the number of lines, boxes, and calculations)?

Provide specific code-level suggestions where possible.

## **golden\_answer**

This script has several optimization opportunities across algorithm design, redundant calculations, and resource management.

### **Algorithm Improvements**

1\. Swing point detection redundancy

The script recalculates `ta.highest(structureLen)` and `ta.lowest(structureLen)` on every bar, but these values only change when new highs/lows occur. Consider caching these values and only updating when necessary:

var float cached\_upper \= na  
var float cached\_lower \= na  
var int last\_calc\_bar \= 0

// Only recalculate every structureLen bars  
if bar\_index \- last\_calc\_bar \>= structureLen  
    cached\_upper := ta.highest(structureLen)  
    cached\_lower := ta.lowest(structureLen)  
    last\_calc\_bar := bar\_index

upper \= cached\_upper  
lower \= cached\_lower

This reduces expensive lookback calculations from every bar to periodic updates.

2\. Redundant zone filtering

The `zoneFilter` check happens inside both `if mss_up` and `if mss_down` blocks. You're creating zones and then immediately checking if they should be filtered. Better to check the filter condition before creating any drawing objects:

if mss\_up and zoneFilter \!= "Bearish Only"  
    // Create zone  
      
if mss\_down and zoneFilter \!= "Bullish Only"  
    // Create zone

This avoids object creation overhead for filtered zones.

### **Calculation Optimizations**

3\. Repeated Fibonacci calculations

The five line creation calls inside `live_lvls := fs.new(...)` each call `line.new()` with very similar parameters. The only differences are the fib level value and color. The repeated calls to `math.max(live_zone.get_left(),last_zone.get_right())` are particularly wasteful—this value is computed 10 times per zone update but never changes.

Extract it once:

if lvls.size() \> 1 and noOverlap  
    last\_zone \= zones.get(zones.size()-2)  
    last\_lvl \= lvls.get(lvls.size()-2)  
      
    if last\_lvl.f1.get\_x2() \> live\_lvls.f1.get\_x1()  
        // Compute once, reuse  
        overlap\_x \= math.max(live\_zone.get\_left(), last\_zone.get\_right())  
          
        last\_lvl.f1.set\_x2(overlap\_x)  
        last\_lvl.f2.set\_x2(overlap\_x)  
        // ... etc

This reduces 10 method calls and comparisons to 1\.

4\. Conditional input processing

Lines like `f1Lvl = f1Tog?input.float(...):na` are evaluated every bar even though inputs only change when the user modifies settings. Pine Script handles this internally, but the ternary pattern adds unnecessary branching. Consider simplifying input declarations or pre-computing enabled fib levels into an array.

### **Resource Usage Efficiency**

5\. Drawing object management

You're creating up to 5 lines per zone, and with `max_lines_count = 500`, you could hit limits on active charts. The current approach deletes old zones when `zones.size() > zoneDispNum`, but this happens after creation. For large datasets, you're churning through objects unnecessarily.

If `dispAll` is false, consider a fixed-size circular buffer approach instead of push/shift.

6\. Line updates in overlap cleanup

The overlap cleanup block updates each of 5 lines individually with separate `set_x1()` and `set_x2()` calls—that's 50 method calls per overlap resolution (5 lines x 2 endpoints x 2 zones x 2.5 average operations). Since all 5 lines in a zone move together, consider creating a helper function that updates all lines in a `fs` object at once.

7\. Redundant array access

last\_zone \= zones.get(zones.size()-2)  
last\_lvl \= lvls.get(lvls.size()-2)

You call `.size()-2` four times. Store the index once:

if lvls.size() \> 1 and noOverlap  
    prev\_idx \= zones.size() \- 2  
    last\_zone \= zones.get(prev\_idx)  
    last\_lvl \= lvls.get(prev\_idx)

### **Specific Code Suggestions**

High-impact changes:

* Cache swing point calculations (saves \~100 lookback operations per bar)  
* Extract repeated `math.max()` call in overlap section (saves \~10 calls per overlap)  
* Move `zoneFilter` check before object creation (avoids creating objects that get immediately filtered)

Medium-impact changes:

* Pre-compute enabled fib levels into an array to avoid repeated ternary checks  
* Create helper function for batch line updates to reduce method call overhead

Low-impact changes:

* Store array indices instead of recalculating `.size()-2`  
* Consider using arrays instead of individual variables for the 5 fib levels to enable loop-based updates

### **Measurement**

To verify improvements, monitor:

* Script execution time (visible in Pine Script performance profiler)  
* Number of active drawing objects on the chart  
* Memory usage with large datasets (500+ bars of history)

Focus on the high-impact changes first—the swing point caching and redundant calculation elimination will have the most noticeable effect on performance.

## **rubric**

{  
  "reasoning": \[  
    {  
      "id": "reasoning-1",  
      "description": "Identifies that swing point calculations (ta.highest/ta.lowest) are redundantly computed on every bar",  
      "weight": "critical",  
      "rationale": "This is one of the primary performance bottlenecks in the algorithm and must be recognized to address the optimization request",  
      "dependent\_on": \[\]  
    },  
    {  
      "id": "reasoning-2",  
      "description": "Recognizes the repeated math.max() calculation in the overlap cleanup section",  
      "weight": "critical",  
      "rationale": "This is computed 10 times per update with the same inputs \- identifying this redundancy is essential for optimization",  
      "dependent\_on": \[\]  
    },  
    {  
      "id": "reasoning-3",  
      "description": "Explains why caching lookback operations would improve performance",  
      "weight": "critical",  
      "rationale": "Must demonstrate understanding of how to reduce computational overhead, not just identify the problem",  
      "dependent\_on": \["reasoning-1"\]  
    },  
    {  
      "id": "reasoning-4",  
      "description": "Identifies that zone filtering happens after object creation rather than before",  
      "weight": "bonus",  
      "rationale": "Shows deeper analysis \- avoiding object creation before filtering reduces memory allocation overhead",  
      "dependent\_on": \[\]  
    },  
    {  
      "id": "reasoning-5",  
      "description": "Explains why the duplicate Fibonacci line creation pattern is inefficient",  
      "weight": "bonus",  
      "rationale": "Demonstrates understanding that repetitive code patterns suggest loop-based refactoring opportunities",  
      "dependent\_on": \[\]  
    },  
    {  
      "id": "reasoning-6",  
      "description": "Suggests rewriting the entire algorithm in a different language or framework",  
      "weight": "penalty",  
      "rationale": "Prompt asks for Pine Script optimizations within the current codebase, not migration to a different platform",  
      "dependent\_on": \[\]  
    },  
    {  
      "id": "reasoning-7",  
      "description": "Claims the code cannot be optimized without showing it's already optimal",  
      "weight": "penalty",  
      "rationale": "The code has clear redundancies and inefficiencies \- dismissing optimization possibilities is incorrect",  
      "dependent\_on": \[\]  
    }  
  \],  
  "completeness": \[  
    {  
      "id": "completeness-1",  
      "description": "Provides specific Pine Script code examples showing how to implement at least one optimization",  
      "weight": "critical",  
      "rationale": "Prompt explicitly asks for actionable, implementable advice with code-level suggestions",  
      "dependent\_on": \[\]  
    },  
    {  
      "id": "completeness-2",  
      "description": "Addresses at least two of the three requested optimization aspects (algorithm structure, calculation efficiency, resource usage)",  
      "weight": "critical",  
      "rationale": "Must cover multiple dimensions of the optimization request to be complete",  
      "dependent\_on": \[\]  
    },  
    {  
      "id": "completeness-3",  
      "description": "Proposes a concrete approach for caching or storing computed swing point values",  
      "weight": "bonus",  
      "rationale": "Demonstrates how to eliminate the redundant lookback calculations identified in reasoning",  
      "dependent\_on": \["reasoning-1", "reasoning-3"\]  
    },  
    {  
      "id": "completeness-4",  
      "description": "Suggests refactoring the duplicate Fibonacci line creation into a loop or function",  
      "weight": "bonus",  
      "rationale": "Addresses the code duplication pattern with a maintainable solution",  
      "dependent\_on": \[\]  
    },  
    {  
      "id": "completeness-5",  
      "description": "Discusses drawing object limits (max\_boxes\_count, max\_lines\_count) and their impact on optimization strategy",  
      "weight": "bonus",  
      "rationale": "Shows awareness of Pine Script resource constraints that affect performance decisions",  
      "dependent\_on": \[\]  
    },  
    {  
      "id": "completeness-6",  
      "description": "Categorizes suggestions by implementation priority or expected performance impact",  
      "weight": "bonus",  
      "rationale": "Helps reader understand which optimizations to implement first for maximum benefit",  
      "dependent\_on": \[\]  
    },  
    {  
      "id": "completeness-7",  
      "description": "Recommends measurement or profiling approach to verify optimization effectiveness",  
      "weight": "bonus",  
      "rationale": "Professional practice \- validating that changes actually improve performance",  
      "dependent\_on": \[\]  
    },  
    {  
      "id": "completeness-8",  
      "description": "Provides only vague suggestions without specific line numbers, variable names, or code changes",  
      "weight": "penalty",  
      "rationale": "Prompt explicitly requests code-level, implementable suggestions \- generic advice fails to meet this requirement",  
      "dependent\_on": \[\]  
    }  
  \],  
  "style": \[  
    {  
      "id": "style-1",  
      "description": "Organizes optimizations into logical categories matching the prompt structure (algorithm, calculation, efficiency)",  
      "weight": "bonus",  
      "rationale": "Makes the response easy to navigate and aligns with the prompt's organizational framework",  
      "dependent\_on": \[\]  
    },  
    {  
      "id": "style-2",  
      "description": "References specific line numbers, variable names, or code sections from the provided script",  
      "weight": "bonus",  
      "rationale": "Demonstrates careful code review and grounds suggestions in the actual codebase",  
      "dependent\_on": \[\]  
    },  
    {  
      "id": "style-3",  
      "description": "Explains the performance rationale behind each optimization suggestion",  
      "weight": "bonus",  
      "rationale": "Helps reader understand optimization principles for future improvements, not just apply fixes mechanically",  
      "dependent\_on": \[\]  
    },  
    {  
      "id": "style-4",  
      "description": "Uses correct Pine Script syntax and terminology in all code examples",  
      "weight": "bonus",  
      "rationale": "Ensures code examples are directly usable and demonstrates domain expertise",  
      "dependent\_on": \[\]  
    }  
  \]  
}

# **Intents**

# **Architecture & System Design**

Definition

How existing components are structured and interact, why design decisions were made, and how to plan or design systems and components going forward. Covers both understanding existing architecture and making architectural recommendations for future changes. Distinction from Code Onboarding & Comprehension is that this is a deep dive into the architecture of the codebase, while Code Onboarding & Comprehension is a simple walkthrough of existing code.

## **Example Prompt Asks**

* "How does authentication flow through this middleware stack?"  
* "We need to add a caching layer to this service…how should we design it?"

## **Specifications**

### **Prompt**

* The prompt must be written in natural language with a peer to peer tone.  
* The prompt must include a description of the system or component.  
* The prompt must include the code or documentation that is relevant to the question.  
* The prompt must be asking about at least one aspect of the intent but may ask about multiple aspects.

### **Golden**

* The golden can have code blocks if it is relevant to the question.

# **Code Debugging & Repair**

Definition

Identify and fix a known or suspected problem. Unlike Root Cause Analysis, the cause is already known or suspected, but the task is producing the actual fix. The primary deliverable is working corrected code with reasoning.

## **Example Prompt Asks**

* "Here's the stack trace `<insert code snippet>` can you fix the function causing this?"  
* "This SQL query returns duplicates when a user has multiple roles, can you fix it?"

## **Specifications**

### **Prompt**

* The prompt must be written in natural language with a peer to peer tone.  
* The prompt must include the source of the error and all necessary context to reproduce the error.  
* The bug/bugs can be more than one bug interacting with each other, one can be known and the other hidden by the first.  
* The bug should be known and the prmopt must ask for a fix.

### **Golden**

* The golden must include the corrected code.  
* The golden must include the reasoning behind the fix.  
* If a bug was hidden in the prompt the golden must call it out and provide fix as well.

# **Code Onboarding & Comprehension**

Definition

Plain-language walkthrough of what existing code, files, or functions do. The primary deliverable is a narrative explanation of what something does, not why it fails or how to change it. Distinction from Architecture Analysis is that this is a simple walkthrough of existing code, while Architecture Analysis is a deep dive into the architecture of the codebase.

## **Example Prompt Asks**

* "I just joined this project, what does this module actually do?"  
* "What's the entry point for this service and how does execution flow from there?"

## **Specifications**

### **Prompt**

* The prompt must be written in natural language with a peer to peer tone.  
* The prompt must include the all necessary code that needs to be comprehended.  
* The prompt must not have any bugs or errors in the code.

### **Golden**

* The golden must be wrote in plain language.  
* The golden can not have code blocks but can have inline code snippets to help explain the code.  
* The golden should not suggest changes to the code, but rather explain what the code does.  
* The golden should use instructor like language to help the user understand the code.

# **PR Triage & Assessment**

Definition

Explain a code change, assess blast radius, classify severity/complexity, flag security risk. You will be providing a diff and the relevent files that are being modified or use a modifed file.

## **Example Prompt Asks**

* "What does this PR actually change and is anything likely to break?"  
* "This diff touches the auth module; what are the security implications?"

## **Specifications**

Definition

You will be uploading the files using the Input files field. You will need to provide all files that are modfied in the diff or are files that use the modified files in the prompt.

### **Prompt**

* The prompt must be written in natural language with a peer to peer tone.  
* The prompt must include a PR diff and code so that the LLM can perform the assessment.  
* Wether a prompt has a security risk will be determined in seeding, so when claiming a prompt check to determine if you need to include a security risk in the prompt and a callout in the golden response.  
* Include all files that are modified in the PR diff or are files that use the modified files in the prompt as input files.

### **Golden**

* The golden should not have code blocks as this is an explanation task, but it can have inline code and comments to aid in the explanation.  
* The golden should the explanation of the code change, blast radius, severity/complexity, and security risk.  
* The golden should be formatted into 4 sections: Code change explaination, then blast radius, then severity/complexity, then security risk.  
* The golden should include explaination about what the changes mean for the other files in the repo. Ie, files that might be indirectly impacted by the change. This should be done across the sections as required.  
* Not every prompt will have a security risk, but the golden should still show the risk was analyzed by stating there is no risk. This will be determined by the category

### **Rubric**

* Rubrics should have at least one check for each section above  
* Rubric criterion dependencies should span sections but never be dependent on something that is wrote later in the golden response, ie. Security risk can depend on code explanation but code explanation can not depend on something in the security risk section

# **Root Cause Analysis**

Definition

Why observed behavior is broken, unexpected, or failing — traced through the codebase. The primary deliverable is a diagnosis of what went wrong and why.

## **Example Prompt Asks**

* "This function returns None intermittently but I can't reproduce it locally. What's happening?"  
* "Why does this test pass in isolation but fail in the suite?"

## **Specifications**

### **Prompt**

* The prompt must be written in natural language with a peer to peer tone.  
* The prompt must include the source of the error and all necessary context to reproduce the error.  
* The bug in the prompt must span multiple files and can be more than one bug interacting with each other.

### **Golden**

* The golden must explain the root cause of the error.  
* The golden can have inline code snippets to help explain the root cause of the error.  
* The golden should not include code blocks for fixing the error as this is a diagnosis task, not a solution task.  
* The golden should walk through why the error is happening, what the expected behavior is, and what the actual behavior is.

### **Prompt Quality**

*  Is the prompt clear and self-contained?  
*  Does it match the given category?  
*  Does it read like a realistic developer request (not role-play or meta-instructions)?

### **Golden Answer Quality**

*  Is the golden answer high quality?  
*  Does it demonstrate expert-level understanding?  
*  Is it appropriately detailed without being verbose?  
*  Does it match the given intent?  
*  Does the golden answer match every part of the rubric? Ensure that every single part of the rubric is answered for in the golden answer, or at least alluded to. You can go in-depth on the correctness of the rubric later—simply check for completeness here.  
*  Does the golden answer also answer bonuses adequately?  
*  Does the golden answer avoid adding extra functionality not specified in the rubric?

### **Rubric Quality**

Count the number of rubric items that are critical:

*  Are there too many rubric items in this category such that we have overprescribed the solution to the model and inflate difficulty scores?  
*  Are there too few critical rubric items? Does the task need to be more difficult?

Do the rubric items make sense?

*  Does every rubric item contribute meaningfully to a productive answer?  
*  Are rubric rationales meaningful and justify the weight choice (why critical vs. bonus, why this penalty severity)?  
*  Is the description of each rubric item specific enough? Rubric criteria must avoid subjective language and instead define one precise, measurable behavior—clearly specifying what must be identified, where it occurs, how the failure manifests, or what constitutes a correct fix.

Are rubric items atomic?

*  Ensure that rubric items do not stack multiple criteria on top of each other. Specifically check for "and" clauses.

Are all rubric items necessary?

*  If there are rubric items that can be rolled into other rubric items without violating atomicity, remove these rubric items.  
*  Does the rubric allow alternative correct diagnoses or fixes? If you can still reach the functionality specified in the golden answer through the rubric, without exactly copying the answer, then the rubric is loose enough. If not, consider loosening the standards of the rubric.

Do rubric weights make sense?

*  For Penalties, the rubric must only be when the model actively ADDS harmful behavior. Anything that says "Does not incorrectly attribute…", "Does not suggest…", "Does not recommend…", "Does not ignore…", "Does not propose…" should be reviewed. Do the penalties just take the reverse of the critical values? If so, request changes. Penalties should be net-new criteria that identify harmful/misleading responses, not just "didn't do X".  
*  For Critical, does the model answer fail if this rubric item is not included in the answer? If not, reevaluate the "critical" label (change to bonus) or make the rubric item harder.  
*  For Bonus, will the model answer work without this rubric criteria being met? If not, make this rubric item critical.

Check rubric dependencies:

*  Do rubric dependencies make logical sense (e.g., a fix depends on identifying the root cause)?  
*  Are dependencies connected by the correct categories?  
* Critical → Critical (allowed)  
* Bonus → Critical (allowed)  
* Critical → Bonus (NOT allowed)  
* Bonus → Bonus (allowed)  
* Rule of thumb: if a Bonus rubric item is triggered by something that is not a bonus, then you should reconsider this dependency.

### **Penalty Criteria**

Critical Check

Do the penalties just take the reverse of the critical values? If so, request changes.

Penalties should be net-new criteria that identify harmful/misleading responses, not just "didn't do X".

*  Are penalties truly harmful behaviors (suggesting broken code, incorrect facts, security issues) rather than just absence of good things?

### **Difficulty Calibration**

Important

* If you marked something Hard but it's just a simple bug fix, we will downgrade it.  
* If you marked something Medium but it requires deep domain expertise and architectural decisions,we will upgrade it.  
*  Does the difficulty level match the actual complexity?

## **Common Issues to Watch For**

### **Vague Criteria**

Criteria that use words like:

* "correctly"  
* "properly"  
* "appropriately"  
* "adequately"  
* "as expected"

These should be replaced with specific, observable behaviors.

### **Stacked Criteria**

Criteria that combine multiple requirements with "and", "or", or any other way should be split into separate atomic criteria.

### **Misaligned Difficulty**

Tasks marked as "Hard" should require:

* Deep domain expertise  
* Multi-step reasoning  
* Architectural decisions  
* Non-obvious tradeoffs

Tasks marked as "Easy" should be straightforward with clear solutions.  
