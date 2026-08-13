# Section 1 — Calculation Flow & Dependencies

## Prompt 1
Analyze the Portfolio notebook as a business calculation.

Ignore notebook structure and Python implementation details unless they affect calculation behavior.

Identify:

1. Purpose of this portfolio calculation.
2. Upstream portfolios or datasets it depends on.
3. Major calculation steps in execution order.
4. Business inputs required by each step.
5. Important intermediate datasets produced.
6. Final portfolio/result produced.
7. How the resulting portfolio is subsequently used.
8. Business rules or assumptions that materially affect the result.

For every significant finding, reference the relevant notebook code/cell/function so it can be verified.

Clearly label anything that is inferred rather than directly demonstrated by the code.

Do not propose target architecture, components, services, APIs, or classes.

## Prompt 2
Analyze the Alpha calculation in this notebook.

Explain:

1. What two or more portfolio states/results are being compared.
2. Inputs required for the alpha calculation.
3. Major transformations/calculations performed.
4. Output produced.
5. Dependency on upstream portfolio calculations.
6. Classification, hierarchy, weighting, return, or other data that materially affects the calculation.
7. Important assumptions embedded in the implementation.

Describe this primarily in investment/performance calculation terminology rather than Python terminology.

Reference code evidence for significant findings and clearly identify inferred behavior.

Do not propose target architecture.

## Prompt 3
Using the verified analysis of TAM, Policy, Extended and Positioning notebooks, construct one end-to-end M***R calculation dependency model.

Show:

* TAM portfolio
* Policy portfolio
* Extended portfolio
* Positioning portfolio
* corresponding alpha calculations
* dependency between portfolio calculations
* major external data required at each stage
* important intermediate outputs passed between stages

For every transition, explain what business transformation occurs.

Separate portfolio construction from alpha calculation.

Produce:

1. A concise textual walkthrough.
2. A simple dependency diagram using Mermaid.
3. A table with columns: Stage | Depends On | Major Inputs | Transformation | Output.

Do not include application components or technology architecture.

# Section 2 — Common Pattern vs Portfolio-Specific Logic

## Prompt 1
Compare the implementation of TAM, Policy, Extended and Positioning portfolio calculations.

Analyze them by business/calculation behavior rather than by Python function or notebook structure.

Identify:

* calculation stages common across multiple portfolio types
* common data transformations
* common validation or preparation logic
* common input/output structures
* repeated calculation patterns
* behavior unique to TAM
* behavior unique to Policy
* behavior unique to Extended
* behavior unique to Positioning

Produce a comparison matrix:

Behavior | TAM | Policy | Extended | Positioning | Common or Variable

Do not propose classes, services, microservices or deployment architecture.

## Prompt 2
Review the portfolio notebooks for conceptually repeated business logic.

Do not focus only on duplicated code. Two implementations may use different Python code while representing the same business capability.

Look specifically for repeated concepts involving:

* loading portfolio inputs
* resolving calculation context
* classification/hierarchy processing
* portfolio construction
* weights
* normalization
* returns
* contribution calculations
* alpha calculations
* aggregation
* result preparation

For each repeated concept explain:

1. What remains consistent across portfolio types.
2. What varies.
3. What drives the variation: data, configuration, business rule, algorithm, or portfolio type.
4. Whether the commonality is strongly demonstrated by the notebooks or only appears possible.

Do not design the solution yet.

## Prompt 3
Based on the verified notebook analysis, evaluate whether TAM, Policy, Extended and Positioning should conceptually be treated as:

A. Independent portfolio calculation implementations, or

B. Variations within a common portfolio calculation framework.

Do not choose an answer based on software-design preference.

Use evidence from the notebooks.

Identify:

* invariant behavior
* portfolio-specific behavior
* configuration-driven variation
* algorithm-driven variation
* places where forcing a common abstraction would make the design more complicated
* places where separate implementations would unnecessarily duplicate behavior

Provide evidence and tradeoffs. Do not propose implementation classes or services.

# Section 3 — Logical Components & Contracts

## Prompt 1
Using the verified M***R calculation chain and common-vs-variable analysis, propose the minimum set of logical application responsibilities required to implement the calculation flow.

Derive responsibilities from stable business/calculation capabilities, not from notebook cells, Python functions or existing prototype structure.

For each proposed logical component provide:

* Name
* Primary responsibility
* Calculation capabilities it supports
* Inputs
* Outputs
* Major dependencies
* Logic that belongs inside the component
* Logic that should remain outside the component

Prefer fewer cohesive components over many specialized components.

Do not create microservices.
Do not define deployment boundaries.
Do not propose AWS infrastructure.
Assume these components can initially exist within one application/process.

## Prompt 2
Critically review the proposed M***R logical component model against the notebook evidence.

For each component ask:

* Does it represent a genuinely distinct responsibility?
* Is the boundary supported by different data or behavior?
* Is it simply wrapping an existing Python function?
* Does it introduce unnecessary abstraction?
* Is business logic being duplicated between components?
* Is one component doing too many unrelated things?
* Would combining any components produce a simpler design without losing important separation?

Recommend simplifications where appropriate.

The objective is the smallest coherent component model that can support TAM, Policy, Extended and Positioning calculations.

## Prompt 3
Using the approved logical component model, identify the major data exchanged between components.

Focus only on contracts important to the architecture. Do not document every temporary DataFrame.

For each contract identify:

* Business name of the dataset/object
* Producer
* Consumer
* Business purpose
* Key identifiers
* Major required data elements
* As-of/effective-date semantics
* Important validation expectations

For each important element classify it as:

OBSERVED — directly demonstrated by notebook implementation

INFERRED — implied by notebook behavior but requiring confirmation

PROPOSED — needed for the target application but not present in the notebook

Highlight any contract where the notebook implementation is too ambiguous to establish a reliable target contract.

# Section 4 — Execution & Traceability

## Prompt 1
Assume the M***R notebooks no longer exist and the same calculations must execute within a production application.

Using the verified calculation dependency model, describe the execution of one complete calculation request.

Address only behavior required to execute the calculation correctly:

* calculation context
* portfolio/fund
* as-of date
* calculation chain
* dependency ordering
* configuration required
* upstream data required
* intermediate portfolio results
* alpha calculations
* final result
* execution completion/failure

Identify which calculations must be sequential because of actual data dependencies and which could potentially execute independently.

Do not introduce queues, microservices, workflow products or cloud technologies unless the calculation requirements specifically require them.

Produce a simple sequence/dependency view.

## Prompt 2
Review the notebook calculations specifically for behavior that works implicitly in a notebook but must become explicit in a production application.

Look for examples such as:

* variables carrying state between cells
* dependence on execution order
* intermediate DataFrames held only in memory
* hard-coded calculation context
* manual selection of inputs
* hidden configuration
* implicit portfolio identity
* assumptions about successful upstream calculations
* manual reruns
* lack of execution status
* inability to distinguish different calculation runs

For each gap explain:

Notebook behavior | Why it becomes a production concern | Information or behavior the target design needs

Do not propose infrastructure solutions yet.

## Prompt 3
Take a final Positioning Alpha result as the example.

Determine what information would be needed to explain that result backwards through the M***R calculation chain:

Positioning Alpha
→ Positioning Portfolio
→ Extended Portfolio
→ Policy Portfolio
→ TAM
→ relevant configuration/classification
→ source calculation data

Identify the minimum information that must be associated with calculation results and intermediate portfolios to support this explanation.

Consider:

* calculation/run identity
* portfolio identity
* as-of date
* parent/upstream portfolio
* configuration used
* source inputs used
* intermediate results
* calculation step performed

Separate:

1. Information required to understand/explain a result.
2. Information required to reproduce a calculation.
3. Operational information useful primarily for troubleshooting.

Keep the model minimal. Do not design a generalized data-lineage platform.

