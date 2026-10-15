# Smells and Rewrites

Each entry names debt that adds code without adding capability, the rewrite that removes it, and the trap to avoid. The rewrite always removes something; it never just rearranges it.

## Debt Worth Removing

### Duplication with small variations

Two or more blocks do the same thing with different constants, field names, or a single differing step.

- **Rewrite**: one owner of the logic, with the variation as a parameter, a data table, or a passed-in function.
- **Trap**: merging code that only looks alike but changes for different reasons. If the copies would evolve independently, the duplication is cheaper than the coupling. Three real copies justify a merge; two coincidental ones do not.

### Reinvented standard or installed library features

Hand-written loops, parsers, retries, date arithmetic, deep merges, path handling, argument parsing, or collection helpers that the language, its standard library, or a dependency already in the project provides.

- **Rewrite**: call the existing facility.
- **Trap**: adding a new dependency to delete twenty lines. Prefer what is already installed; a new dependency needs its own justification.

### Pass-through layers

Wrappers, managers, services, or interfaces that forward calls without adding policy, validation, or a real seam. A single-implementation interface with no test double is usually one.

- **Rewrite**: inline the layer and call the target directly.
- **Trap**: removing a layer that is a real seam (a boundary with a second adapter, a test double, or a stated architectural rule).

### Defensive code for impossible states

Null checks on values the types guarantee, catch blocks that rethrow unchanged, validation repeated at every layer after the boundary already parsed the input, fallbacks for cases that cannot occur.

- **Rewrite**: validate once at the boundary and trust the types inside it.
- **Trap**: deleting a check that guards an untyped boundary such as user input, network data, files, or environment variables.

### Stored state that could be derived

Fields, caches, or flags kept in sync by hand with the data they summarize.

- **Rewrite**: compute the value from its source when needed.
- **Trap**: removing a cache that exists for measured performance reasons.

### Flag parameters

A boolean or mode argument that makes one function do two different jobs.

- **Rewrite**: two functions with honest names, sharing a private helper if needed.

### Conditional ladders

Long `if`/`else` or `switch` chains mapping inputs to values or behavior, often repeated in several places.

- **Rewrite**: a lookup table, a discriminated union with exhaustive matching, or polymorphism when behavior varies by type.
- **Trap**: replacing a readable five-branch `switch` with indirection that is harder to follow.

### Dead code

Unreachable branches, unused exports, functions, parameters, feature flags that are permanently on or off, and configuration nothing reads.

- **Rewrite**: delete it. Version control remembers.
- **Trap**: code reached only through reflection, dynamic imports, public API consumers, or configuration. Search for string references before deleting.

### Accidental complexity in control flow

Deep nesting, flags that track loop state, temporary variables that hold a value used once, manual index bookkeeping.

- **Rewrite**: early returns and guard clauses, iteration constructs that state intent, and extracting a well-named function when a block has its own purpose.

## Not an Improvement

These shrink code while making it worse. Never propose them, and reject them when found in your own drafts:

- nested or chained ternaries;
- long method chains or comprehensions that pack several transformations into one expression;
- several statements crammed onto one line, or logic hidden in boolean short-circuits (`a && doThing()`);
- abbreviated or single-letter names outside tiny lambdas and conventional loop indices;
- clever metaprogramming, reflection, or operator tricks to avoid writing plain code;
- removing named intermediate variables that explain a step;
- removing types, names, or small abstractions that carry domain meaning;
- generic utilities built to merge two call sites that would read better apart;
- reformatting or restyling that a formatter or the repository's conventions already decide.
