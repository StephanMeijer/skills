# Smells and Rewrites

Each entry names a debt or smell, the rewrite that fixes it, and the trap to avoid. The rewrite always removes something (a copy, a branch, a layer, a guess the reader must make) or names something implicit; it never just rearranges it.

## Debts and Smells Worth Fixing

### Duplication with small variations

Two or more blocks do the same thing with different constants, field names, or a single differing step.

- **Rewrite**: one owner of the logic, with the variation as a parameter, a data table, or a passed-in function.
- **Trap**: merging code that only looks alike but changes for different reasons. If the copies would evolve independently, the duplication is cheaper than the coupling. Three real copies justify a merge; two coincidental ones do not.

### Magic numbers and strings

Unnamed literals whose meaning the reader has to guess: `86400`, `3`, `0.15`, `"pending"`, `"X-Api-Key"`, `"/var/lib/app"`, array indices into tuples, and HTTP status or error codes written as bare numbers.

- **Rewrite**: a named constant, enum, or configuration value whose name states what the value means, not what it is (`SESSION_TTL_SECONDS`, not `EIGHTY_SIX_THOUSAND_FOUR_HUNDRED`). Use the language's strongest form: an enum or literal union for a closed set of strings, a typed constant or unit-carrying type for quantities, a standard-library constant where one exists (`http.HTTPStatus.NOT_FOUND`, `math.pi`, `time.Hour`).
- **Trap**: naming values that are already self-explanatory, such as `0` and `1` as identity or index arithmetic, `""`, `-1` for "last", a percentage's `100`, or a literal used once right beside a name that explains it. `const TWO = 2` is noise. Also leave test expectations literal; a test that reads its expected value from the same constant as the code proves nothing.

### Scattered values without an owner

The same value, or values that must agree, written in several places: a timeout in three call sites, a route path in a handler and a client, a queue name in a producer and a consumer, a limit in validation and in an error message, a key string in a writer and a reader. Changing one and missing another is a bug waiting to happen.

- **Rewrite**: one owner, in the module that owns the concept, imported everywhere else. Values that vary by environment go into configuration read once at the boundary. Derive dependent values from the owner (`MAX_LEN` in the message comes from the constant, not a second literal).
- **Trap**: a single `constants` file that collects every value in the project. Constants belong with the concept they describe; a global grab bag couples unrelated modules. Two literals that are equal by coincidence and would change independently are not the same value; do not merge them.

### Primitive obsession and data clumps

Domain concepts passed as bare strings, numbers, or tuples (an email, money as a float, an ID that could be confused with another ID), and the same group of parameters travelling together through several signatures.

- **Rewrite**: a small type or value object that names the concept and validates it once at the boundary; a parameter object or record for the clump.
- **Trap**: wrapping every primitive. Introduce a type when it prevents a real mix-up or removes repeated validation, not for ceremony.

### Long functions and long parameter lists

A function that does several jobs in sequence, needs scrolling to read, or takes more parameters than a reader can keep straight, especially several of the same type in a row.

- **Rewrite**: extract steps with their own purpose into well-named functions; group parameters that belong together; split by job when a flag selects behavior.
- **Trap**: chopping a coherent, linear function into fragments the reader must jump between. Length alone is not the smell; mixed responsibilities are.

### Misleading or inconsistent names

Names that say something the code does not do, the same concept called three things across the codebase, or comments that explain what a better name would make obvious.

- **Rewrite**: rename to the domain's term, as recorded in a glossary when there is one, and use it consistently. Delete a comment the new name makes redundant.
- **Trap**: renaming public API or persisted keys without a migration; that is an issue, not a local fix.

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
